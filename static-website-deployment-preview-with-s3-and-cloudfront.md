---
description: How to create deployment previews with S3 and CloudFront
---

# 🪞 Static website deployment preview with S3 and CloudFront

```
// deployment-preview-proxy-function

'use strict';
exports.handler = (event, context, callback) => {

    // Get request and request headers, uri
    const request = event.Records[0].cf.request;
    const headers = request.headers;
    const uri = request.uri;
    const host = headers.host[0].value;
    
    // Configure authentication
    const authUser = 'admin';
    const authPass = 'minda';

    // Configure deployment preview subdomain
    const subdomain = 'preview';

    let sprintUri = '';
    let hostArray = host.split('.');
    
    // Check whether the subdomain is match
    if (host.match(subdomain) && (hostArray[0] != subdomain)) {
        sprintUri = hostArray[0] + '/';
    }
    
    // Check whether the URI is missing a file name.
    if (uri.endsWith('/')) {
        request.uri += `${sprintUri}index.html`;
    }
    // Check whether the URI is missing a file extension.
    else if (!uri.includes('.')) {
        request.uri += `/${sprintUri}index.html`;
    }

    // Construct the Basic Auth string
    const authString = 'Basic ' + new Buffer(authUser + ':' + authPass).toString('base64');

    // Require Basic authentication
    if (typeof headers.authorization == 'undefined' || headers.authorization[0].value != authString) {
        const body = 'Unauthorized';
        const response = {
            status: '401',
            statusDescription: 'Unauthorized',
            body: body,
            headers: {
                'www-authenticate': [{key: 'WWW-Authenticate', value:'Basic'}]
            },
        };
        callback(null, response);
    }
    
    // Continue request processing if authentication passed
    callback(null, request);
};
```



### Architecture

(chart here . . .)

A user access "sprint-0x.preview.domain.app" will be redirect to "sprint-0x" folder which contains deployment preview for the first sprint.

The "sprint-0x" subdomain can be named/entered with anything as long as there is a folder on the bucket that has the same name.

### Perquisites

* A separated S3 bucket for deployment preview websites (I'll explain why below)
* A subdomain use specifically for deployment preview, i.e: preview.domain.app
* Wildcard ACM certificates for bot \`us-east-1\` region and the region your app are hosted.
* A CloudFront distribution with a wildcard alternative domain name for serving static website and also act as CDN.
* A Lambda@Edge function hosted on CloudFront distribution that works as a proxy (optionally add a basic auth mechanism since preview app tend not to be public)

The total cost at the time of creation is $0.5 for Route53 hosted zone, other charges may be incurred depends on the number of traffics.

### Manually deploy

1. Create a S3 bucket for deployment preview static websites. Each deployment preview will be putted in a separated folder. For example:
2. Create a domain and Route53 hosted zone for our deployment preview sites. Example, preview.domain.app.
3. Request for wildcard ACM certificates on both \`us-east-1\` (for CloudFront) and the region your websites are hosted.
4. Create a CloudFront distribution with the deployment preview bucket as origin and the wildcard domain and certificate.
5. Create a Route53 wildcard record pointing to our newly created CloudFront distribution.
6. Create a Lambda@Edge function with basic lambda execution role in \`us-east-1\` region and deploy to CloudFront.

### Deploy via CloudFormation template

to be added . . .

### Deploy via Terraform modules

to be added

