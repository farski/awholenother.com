---
layout: post
title: TypeScript types for Lambda functions
date: 2026-08-15 14:43 -0400
category: ref
tags:
  - AWS
  - Lambda
  - TypeScript
---

## Reference

- [@types/aws-lambda](https://www.npmjs.com/package/@types/aws-lambda)
- [DefinitelyTyped/aws-lambda](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/aws-lambda)
- [API Gateway Lambda proxy format reference (v1 and v2)](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html#http-api-develop-integrations-lambda.proxy-format)
- [Example Lambda@Edge viewer request event structure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#example-viewer-request)
- [Example Lambda@Edge origin request event structure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#example-origin-request)
- [Example Lambda@Edge origin response event structure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#lambda-event-structure-response-origin)
- [Example Lambda@Edge viewer response event structure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#lambda-event-structure-response-viewer)
- [Lambda@Edge viewer/origin request direct response object reference](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-generating-http-responses.html#lambda-generating-http-responses-object)
- [CloudFront Functions Event Structure](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/functions-event-structure.html)

## API Gateway v1 Payload (trigger event)

**Type:** `APIGatewayProxyEvent`

```json
{
  "version": "1.0",
  "resource": "/my/path",
  "path": "/my/path",
  "httpMethod": "GET",
  …
}
```

- Always used by REST APIs in API Gateway. You can opt into this format for HTTP APIs.
- Always includes `multiValueHeaders` and `multiValueQueryStringParameters`, which have duplicates when the request has multiple values for a given key
- `httpMethod` and `path` are top-level fields

```javascript
/** @import { APIGatewayProxyEvent } from "aws-lambda" */

/**
 * @param {APIGatewayProxyEvent} event
 */
export const handler = async (event) => {}
```

## API Gateway v2 Payload (trigger event)

**Type:** `APIGatewayProxyEventV2`

```json
{
  "version": "2.0",
  "routeKey": "$default",
  "rawPath": "/my/path",
  "rawQueryString": "parameter1=value1&parameter1=value2&parameter2=value",
  …
}
```

- Default payload format for HTTP APIs. Never used by REST APIs.
- No `multiValueHeaders` or `multiValueQueryStringParameters`; multiple values are combined in `headers`/`queryStringParameters` as comma-separated values.
- `cookies` are top-level field.
- Includes `rawPath` and `rawQueryString`

```javascript
/** @import { APIGatewayProxyEventV2 } from "aws-lambda" */

/**
 * @param {APIGatewayProxyEventV2} event
 */
export const handler = async (event) => {}
```

## Lambda function URL Payload

**Type:** `APIGatewayProxyEventV2`

```json
{
  "version": "2.0",
  "routeKey": "$default",
  "rawPath": "/my/path",
  "rawQueryString": "parameter1=value1&parameter1=value2&parameter2=value",
  …
}
```

Generally the same as v2 payload

```javascript
/** @import { APIGatewayProxyEventV2 } from "aws-lambda" */

/**
 * @param {APIGatewayProxyEventV2} event
 */
export const handler = async (event) => {}
```

## API Gateway v1.0 Response

**Type:** `APIGatewayProxyResult`

```json
{
    "isBase64Encoded": true|false,
    "statusCode": httpStatusCode,
    "headers": { "headername": "headervalue", ... },
    "multiValueHeaders": { "headername": ["headervalue", "headervalue2", ...], ... },
    "body": "..."
}
```

- Return value must be a structured object.
- `statusCode` and `body` are required. `isBase64Encoded`, `headers`, and `multiValueHeaders` are optional.

```javascript
/** @import { APIGatewayProxyResult } from "aws-lambda" */

/**
 * @returns {Promise<APIGatewayProxyResult>}
 */
export const handler = async (event) => {}
```

## API Gateway v2.0 Response

**Type:** `APIGatewayProxyResultV2`

```json
{
    "cookies" : ["cookie1", "cookie2"],
    "isBase64Encoded": true|false,
    "statusCode": httpStatusCode,
    "headers": { "headername": "headervalue", ... },
    "body": "Hello from Lambda!"
}
```

- All fields are optional.
- No `multiValueHeaders`.
- Can include `cookies`.

The return value from the Lambda function does **not** need to return a structured value. If a string is returned, it's used as the string value of the body. If a object is returned, it is JSON stringified and that is used as the string value of the body. Status code will be `200` for these implicit responses.

When the function might return a structured result or a string:

```javascript
/** @import { APIGatewayProxyResultV2 } from "aws-lambda" */

/**
 * @returns {Promise<APIGatewayProxyResultV2>}
 */
export const handler = async (event) => {}
```

If you know the function is returning a structured result, use `APIGatewayProxyStructuredResultV2` explicitly:

```javascript
/** @import { APIGatewayProxyStructuredResultV2 } from "aws-lambda" */

/**
 * @returns {Promise<APIGatewayProxyStructuredResultV2>}
 */
export const handler = async (event) => {}
```

## Lambda@Edge Viewer Request Payload

**Type:** `CloudFrontRequestEvent`

The input to the function will be a `CloudFrontRequestEvent`, which includes an array of `Records`:

`{ "Records": […] }`

Each record follows the `CloudFrontRequestEventRecord` type and looks like:

`{ "cf": { "config": {…}, "request": {…} } }`.

The `request` field follows `CloudFrontRequest` and includes information about the request that was made to CloudFront (HTTP info, client IP address, etc)

```javascript
/** @import { CloudFrontRequestEvent } from "aws-lambda" */

/**
 * @param {CloudFrontRequestEvent} event
 */
export const handler = async (event) => {}
```

## Lambda@Edge Origin Request Payload

**Type:** `CloudFrontRequestEvent`

Same as Lambda@Edge Viewer Request Payload. The distinction between the two will be the data in the `request`, but both use the same type. An origin request will include an `origin` field under `request` with information about the configured origin that will be used unless the function changes it, in addition to information about the request.

```javascript
/** @import { CloudFrontRequestEvent } from "aws-lambda" */

/**
 * @param {CloudFrontRequestEvent} event
 */
export const handler = async (event) => {}
```

## Lambda@Edge Viewer/Origin Request Return Value

**Type:** `CloudFrontRequest`

A viewer or origin request Lambda@Edge function can return a `CloudFrontRequest` to determine CloudFront behavior for the veiwer or origin request resolution. `CloudFrontRequest` is the same object type that exists within the `CloudFrontRequestEvent` each of these Lambda@Edge types would receive as a payload, so in its simplest form, the function could return the value from the input. Or, more commonly, it would change some aspect of the `request` or generate a new one.

```javascript
/** @import { CloudFrontRequest } from "aws-lambda" */

/**
 * @returns {Promise<CloudFrontRequest>}
 */
export const handler = async (event) => {}
```

## Lambda@Edge Viewer/Origin Request Direct HTTP Response

**Type:** `CloudFrontResultResponse`

```json
{
    "body": "content",
    "bodyEncoding": "text" | "base64",
    "headers": {
        "header name in lowercase": [{
            "key": "header name in standard case",
            "value": "header value"
         }],
         ...
    },
    "status": "HTTP status code (string)",
    "statusDescription": "status description"
}
```

A viewer or origin request Lambda@Edge can return a `CloudFrontResultResponse` to immediately return a specific HTTP response, shortcircuting any normal origin pull behavior. (Note that the origin request function is only invoked in cases where the edge location does not already have a cache, but when that does happen, if you return a `CloudFrontResultResponse`, that response is what would get cached.)

```javascript
/** @import { CloudFrontResultResponse } from "aws-lambda" */

/**
 * @returns {Promise<CloudFrontResultResponse>}
 */
export const handler = async (event) => {}
```

## Variable Lambda@Edge Viewer/Origin Request

If a viewer or origin request may return either a request object or a direct HTTP response:

```javascript
/** @import { CloudFrontResultResponse, CloudFrontRequest } from "aws-lambda" */

/**
 * @returns {Promise<CloudFrontResultResponse | CloudFrontRequest>}
 */
export const handler = async (event) => {}
```

## CloudFront Functions Event Payload

**Type:** `AWSCloudFrontFunction.Event`

```json
{
    "version": "1.0",
    "context": {
        <context object>
    },
    "viewer": {
        <viewer object>
    },
    "request": {
        <request object>
    },
    "response": {
        <response object>
    }
}
```

The input to a CloudFront function is always an `event` object

```javascript
/**
 * @param {AWSCloudFrontFunction.Event} event
 */
export const handler = async (event) => {}
```

Note that there's no `import`. `Event` is in a global namespace, so instead you should include `aws-cloudfront-function` under `types` in `tsconfig.json`.

## CloudFront Functions Event Return values

The CloudFront Function will return either a `Request` or a `Response` (not an entire `Event` object)

```javascript
/**
 * @returns {AWSCloudFrontFunction.Request}
 */
export const handler = async (event) => {}
```

```javascript
/**
 * @returns {AWSCloudFrontFunction.Response}
 */
export const handler = async (event) => {}
```

Note that there's no `import`. `Request` and `Response` are in a global namespace, so instead you should include `aws-cloudfront-function` under `types` in `tsconfig.json`.
