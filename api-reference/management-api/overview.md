---
description: Base URL, authentication, response format, errors, and rate limits of the Engy Management API.
---

# Management API overview

This page describes the conventions that apply to every endpoint of the Engy Management API. It is intended for developers who build monitoring, budgeting, or automation tools for their own Engy account.

The current endpoints are read-only. For an introduction and instructions to create an account token, see [Management API](https://docs.engy.ai/account-and-billing/management-api).

## Base URL

```text
https://engy.ai/api/v1
```

## Authentication

Send your account token in the `Authorization` header:

```http
Authorization: Bearer rk-engy-...
```

Account tokens start with `rk-engy-`. API keys (`sk-engy-`) are not accepted.

## Response format

Successful responses return HTTP status `200` with the following structure:

| Field | Type | Description |
| --- | --- | --- |
| `data` | Object or array | The requested data. |
| `meta` | Object | Information about the response, such as units and applied filters. |

Amounts are returned twice: as integers in micro-USD (fields that end in `_micro`), and as decimal strings in USD (fields that end in `_usd`). One USD equals 1,000,000 micro-USD.

## Errors

Failed requests return an `error` object:

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Missing or invalid read token."
  }
}
```

| Code | Status | Description |
| --- | --- | --- |
| `invalid_request` | `400` | A parameter is invalid. |
| `unauthorized` | `401` | The account token is missing or invalid. |
| `forbidden` | `403` | The account is not active. |
| `not_found` | `404` | The resource does not exist. |
| `rate_limited` | `429` | The rate limit is exceeded. |
| `internal` | `500` | An internal error occurred. Retry the request later. |

## Rate limits

Each account token can make 120 requests per minute. When you exceed the limit, the API returns status `429` with the following headers:

| Header | Description |
| --- | --- |
| `Retry-After` | The number of seconds to wait before you retry. |
| `X-RateLimit-Limit` | The maximum number of requests in the window. |
| `X-RateLimit-Window` | The length of the window, in seconds. |

## Machine-readable specification

The following resources describe the Management API and do not require authentication:

| Resource | Description |
| --- | --- |
| [`https://engy.ai/api/v1/openapi.json`](https://engy.ai/api/v1/openapi.json) | The OpenAPI 3.1 specification. |
| [`https://engy.ai/api/v1/llms.txt`](https://engy.ai/api/v1/llms.txt) | A plain-text summary for AI agents. |
