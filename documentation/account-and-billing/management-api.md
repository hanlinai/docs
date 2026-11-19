---
description: Manage your Engy account programmatically with the Management API.
---

# Management API

The Management API lets you manage your Engy account programmatically, on your own behalf. This page explains what the API provides and how to authenticate and call it. It is intended for developers who build monitoring, budgeting, or automation tools.

## Capabilities

| Endpoint | Description |
| --- | --- |
| `GET /api/v1/balance` | Returns your balance and the amount available after holds. |
| `GET /api/v1/usage` | Returns requests and spending, aggregated by day and by model. |
| `GET /api/v1/models` | Returns the model catalog with prices and token limits. |

The base URL is `https://engy.ai`.

{% hint style="info" %}
The Management API was previously named the Agent API. The endpoints and account tokens are the same.
{% endhint %}

The current endpoints are read-only. To create or delete API keys and to add credits, use the [dashboard](https://engy.ai/dashboard). To send inference requests, use the inference API with an API key.

## Authentication

The Management API uses an account token. Account tokens are different from API keys:

| Credential | Prefix | Purpose | Can spend credits |
| --- | --- | --- | --- |
| Account token | `rk-engy-` | Read account data through the Management API | No |
| API key | `sk-engy-` | Send inference requests | Yes |

Because an account token cannot spend credits, you can give it to monitoring tools and agents with less risk than an API key.

### Create an account token

{% stepper %}
{% step %}
### Open the account tokens page

In the dashboard, go to **Account tokens**.
{% endstep %}

{% step %}
### Create the token

Click **Create account token**.
{% endstep %}

{% step %}
### Copy the token

Copy the token and store it securely.

{% hint style="warning" %}
The token is displayed only once. If you lose it, regenerate it.
{% endhint %}
{% endstep %}
{% endstepper %}

Each account has one active account token. To replace it, click **Regenerate token**. The previous token stops working immediately, so update every tool that uses it.

## Make a request

Send the account token in the `Authorization` header:

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://engy.ai/api/v1/balance \
  -H "Authorization: Bearer $ENGY_ACCOUNT_TOKEN"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.get(
    "https://engy.ai/api/v1/balance",
    headers={"Authorization": f"Bearer {os.environ['ENGY_ACCOUNT_TOKEN']}"},
)
response.raise_for_status()

print(response.json()["data"]["available_usd"])
```
{% endtab %}
{% endtabs %}

## Endpoints

### Get balance

`GET /api/v1/balance` returns the current balance:

```json
{
  "data": {
    "available_micro": 24500000,
    "available_usd": "24.50",
    "balance_micro": 25000000,
    "balance_usd": "25.00"
  },
  "meta": {
    "currency": "USD",
    "unit": "micro"
  }
}
```

| Field | Description |
| --- | --- |
| `balance_micro` | The total balance, in micro-USD |
| `balance_usd` | The total balance, in USD |
| `available_micro` | The balance minus holds for requests in progress, in micro-USD |
| `available_usd` | The balance minus holds for requests in progress, in USD |

### Get usage

`GET /api/v1/usage` returns requests and spending for a time window.

| Query parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `days` | Integer | No | The window in days. Accepted values are `7` and `30`. The default is `30`. |
| `model` | String | No | Returns usage for this model ID only. |

```bash
curl "https://engy.ai/api/v1/usage?days=7&model=glm-5.2" \
  -H "Authorization: Bearer $ENGY_ACCOUNT_TOKEN"
```

```json
{
  "data": {
    "total": {"requests": 1200, "cost_micro": 3400000, "cost_usd": "3.40"},
    "by_day": [
      {"day_start": 1792281600, "requests": 180, "cost_micro": 510000, "cost_usd": "0.51"}
    ],
    "by_model": [
      {"model": "glm-5.2", "requests": 1200, "cost_micro": 3400000, "cost_usd": "3.40"}
    ]
  },
  "meta": {
    "days": 7,
    "model": "glm-5.2"
  }
}
```

| Field | Description |
| --- | --- |
| `total` | The totals for the whole window |
| `by_day` | The totals per UTC day. `day_start` is the start of the day, as a Unix timestamp in seconds |
| `by_model` | The totals per model, sorted by cost in descending order |

Usage covers complete UTC days only, up to and including the previous day.

### List models

`GET /api/v1/models` returns the model catalog:

```json
{
  "data": [
    {
      "model": "glm-5.2",
      "input_per_1k_micro": 680,
      "output_per_1k_micro": 1500,
      "cached_per_1k_micro": 180,
      "input_per_1k_usd": "0.00068",
      "output_per_1k_usd": "0.0015",
      "max_input_tokens": 262144,
      "max_output_tokens": 262144
    }
  ],
  "meta": {
    "count": 1
  }
}
```

| Field | Description |
| --- | --- |
| `model` | The model ID |
| `input_per_1k_micro` | The price per 1,000 input tokens, in micro-USD |
| `output_per_1k_micro` | The price per 1,000 output tokens, in micro-USD |
| `cached_per_1k_micro` | The price per 1,000 cached input tokens, in micro-USD. The value is `null` if the model has no cached rate |
| `input_per_1k_usd` | The price per 1,000 input tokens, in USD |
| `output_per_1k_usd` | The price per 1,000 output tokens, in USD |
| `max_input_tokens` | The maximum number of input tokens |
| `max_output_tokens` | The maximum number of output tokens |

## Response format

Successful responses return HTTP status `200` with the following structure:

| Field | Description |
| --- | --- |
| `data` | The requested data |
| `meta` | Information about the response, such as units and applied filters |

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

| Code | HTTP status | Description |
| --- | --- | --- |
| `invalid_request` | 400 | A parameter is invalid, such as a `days` value other than `7` or `30` |
| `unauthorized` | 401 | The account token is missing or invalid |
| `forbidden` | 403 | The account is not active |
| `not_found` | 404 | The resource does not exist |
| `rate_limited` | 429 | The rate limit is exceeded |
| `internal` | 500 | An internal error occurred. Retry the request later |

## Rate limits

Each account token can make 120 requests per minute. When you exceed the limit, the API returns HTTP status `429` with the following headers:

| Header | Description |
| --- | --- |
| `Retry-After` | The number of seconds to wait before you retry |
| `X-RateLimit-Limit` | The maximum number of requests in the window |
| `X-RateLimit-Window` | The length of the window, in seconds |

## Machine-readable specification

The following resources describe the Management API and do not require authentication:

| Resource | Description |
| --- | --- |
| [`https://engy.ai/api/v1/openapi.json`](https://engy.ai/api/v1/openapi.json) | The OpenAPI 3.1 specification |
| [`https://engy.ai/api/v1/llms.txt`](https://engy.ai/api/v1/llms.txt) | A plain-text summary for AI agents |
