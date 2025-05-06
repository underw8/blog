# 🐛 Using Cloudflare Access Service Tokens with a Single JSON Header

### TL;DR

* Cloudflare Access supports single-header service token authentication using JSON
* The JSON keys must be lowercase: `cf-access-client-id`, `cf-access-client-secret`
* Set `read_service_tokens_from_header` to your custom header name

### Background

Cloudflare Access supports authenticating requests to protected resources using service tokens, typically via two headers:

```
CF-Access-Client-Id: <client_id>
CF-Access-Client-Secret: <client_secret>
```

However, for some use cases—especially API calls or automation, my case is MCP workers with Zero Trust protected—you may want to consolidate authentication into a single header. While [Cloudflare’s documentation](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/#authenticate-with-a-single-header) mentions this, one critical detail is not clearly documented: **the required lowercase key names in the JSON payload**.

Funny enough, even their sample cURL command is in the incorrect format — wrong casing and use of underscores. 🤦🏻‍♂️

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### Problem

Attempting to use a single header like this:

```
curl -H 'Authorization: {"CF-Access-Client-Id": "abc", "CF-Access-Client-Secret": "def"}' \
     https://example.workers.dev
```

…results in a 403 Forbidden, even if:

* The token values are correct
* The header is well-formed
* Your Cloudflare Access app is configured with `read_service_tokens_from_header`

### Solution

Use lowercase keys in the JSON string:

```
curl -H 'Authorization: {"cf-access-client-id": "abc", "cf-access-client-secret": "def"}' \
     https://example.workers.dev
```

This works only if:

* The `read_service_tokens_from_header` field is set (via Terraform, API)
* The JSON is valid and properly escaped
* The header name matches exactly
