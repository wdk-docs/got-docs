---
title: "AWS"
linkTitle: ""
weight: 12
type: "docs"
---

请求 AWS 服务需要有自己的头签名。 这可以通过使用[`aws4`](https://www.npmjs.com/package/aws4)包来完成。 这对于签名的请求查询的["API Gateway"](https://docs.aws.amazon.com/apigateway/api-reference/signing-requests/)的例子。

```js
const got = require("got");
const AWS = require("aws-sdk");
const aws4 = require("aws4");

const chain = new AWS.CredentialProviderChain();

// Create a Got instance to use relative paths and signed requests
const awsClient = got.extend({
	prefixUrl: "https://<api-id>.execute-api.<api-region>.amazonaws.com/<stage>/",
	hooks: {
		beforeRequest: [
			async options => {
				const credentials = await chain.resolvePromise();
				aws4.sign(options, credentials);
			}
		]
	}
});

const response = await awsClient("endpoint/path", {
	// Request-specific options
});
```
