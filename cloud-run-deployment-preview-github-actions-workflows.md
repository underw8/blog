# 🔍 Cloud Run Deployment Preview Github Actions workflows

Base on official tutorial by GCP which using Cloud Build ([Configuring deployment previews | Cloud Run Documentation](https://cloud.google.com/run/docs/tutorials/configure-deployment-previews)), I've "converted" these flows to Github Actions workflows.

Below is sample deployment previews for `develop` environment.

{% code title="preview_deploy.yml" %}
```yaml
on:
  pull_request:
    branches:
      - develop

name: Build and Deploy Preview WebApp
env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT }}
  SERVICE: ${{ secrets.GCP_PROJECT }}-webapp
  REGION: us-central1

jobs:
  deploy_preview:
    environment: develop
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Cloud SDK
        uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          project_id: ${{ env.PROJECT_ID }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true

      - name: Authorize Docker push
        run: gcloud auth configure-docker

      - name: Build and Push Container
        run: |-
          docker build -t gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.event.number }}-${{  github.sha }} .
          docker push gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.event.number }}-${{  github.sha }}

      - name: Deploy to Cloud Run
        id: deploy
        run: |-
          result=$(gcloud run deploy ${{ env.SERVICE }} --image gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{ github.event.number }}-${{  github.sha }} --region ${{ env.REGION }} --tag "pr-${{ github.event.number }}" --no-traffic --ingress internal --allow-unauthenticated --format json)
          preview_url=$(echo $result | jq -r --arg pr "pr-${{ github.event.number }}" '.status.traffic[] | select(.tag == $pr) | .url')
          echo "::set-output name=preview_url::$preview_url"

      - name: Update Pull Request
        uses: actions/github-script@0.9.0
        if: github.event_name == 'pull_request'
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const output = `#### Preview URL: \<${{ steps.deploy.outputs.preview_url }}>

            *Pusher: @${{ github.actor }}, Action: \`${{ github.event_name }}\`, Workflow: \`${{ github.workflow }}\`*`;
              
            github.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })

```
{% endcode %}

{% code title="preview_cleanup.yml" %}
```yaml
on:
  pull_request:
    branches:
      - develop
    types: [closed]

name: Cleanup Preview WebApp Deployment
env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT }}
  SERVICE: ${{ secrets.GCP_PROJECT }}-webapp
  REGION: us-central1

jobs:
  cleanup:
    environment: develop
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Cloud SDK
        uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          project_id: ${{ env.PROJECT_ID }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true

      - name: Remove Cloud Run revision tags
        run: |
          gcloud run services update-traffic ${{ env.SERVICE }} --remove-tags "pr-${{ github.event.number }}" --region ${{ env.REGION }}

      - name: Clean-up old GCR tags
        run: |
          tags=$(gcloud container images list-tags gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}  --filter="tags:${{ github.event.number }}-*" --format json | jq -r ".[].tags[]")
          for tag in $tags; do gcloud container images delete gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:$tag --quiet; done

```
{% endcode %}

{% code title="deploy.yaml" %}
```yaml
on:
  pull_request:
    branches:
      - develop
    types: [closed]

name: Build and Deploy WebApp
env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT }}
  SERVICE: ${{ secrets.GCP_PROJECT }}-webapp
  REGION: us-central1

jobs:
  deploy:
    if: github.event.pull_request.merged == true
    environment: develop
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Setup Cloud SDK
        uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          project_id: ${{ env.PROJECT_ID }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true

      - name: Authorize Docker push
        run: gcloud auth configure-docker

      - name: Build and Push Container
        run: |-
          docker build -t gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{  github.sha }} .
          docker push gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{  github.sha }}

      - name: Deploy to Cloud Run
        id: deploy
        run: |-
          gcloud run deploy ${{ env.SERVICE }} --image gcr.io/${{ env.PROJECT_ID }}/${{ env.SERVICE }}:${{  github.sha }} --region ${{ env.REGION }} --ingress internal --allow-unauthenticated
          gcloud run services update-traffic ${{ env.SERVICE }} --to-latest --region ${{ env.REGION }}

```
{% endcode %}

{% hint style="info" %}
The deploy workflow is only intended to run on merged PR, not by direct pushing to the branch. You may replace the trigger with `push` to suite your need.
{% endhint %}
