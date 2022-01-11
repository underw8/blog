---
description: How to create deployment previews with S3 and CloudFront
---

# Static website deployment preview with S3 and CloudFront 🪞

```
// deployment-preview-basic-auth-and-router-function

'use strict';
exports.handler = (event, context, callback) => {

    // Get request and request headers, uri
    const request = event.Records[0].cf.request;
    const headers = request.headers;
    const uri = request.uri;
    const host = headers.host[0].value;
    
    console.log(`headers: ${headers}`);
    console.log(`uri: ${uri}`);
    console.log(`host: ${host}`);
    
    let sprintUri = "";
    
    if (host.match('preview')) {
        let hostArray = host.split(".");
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

    console.log(`request.uri: ${request.uri}`);

    // Configure authentication
    const authUser = 'admin';
    const authPass = 'minda';

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
