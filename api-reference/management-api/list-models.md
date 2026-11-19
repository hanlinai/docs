---
description: Reference for GET /api/v1/models, which returns the model catalog with prices and token limits.
---

# List models

This page is the reference for the models endpoint of the Management API. It is intended for developers who need model prices and token limits in micro-USD.

To list models without an account token, use the Inference API endpoint [`GET /v1/models`](../inference-api/list-models.md) instead.

## Endpoint

```http
GET https://engy.ai/api/v1/models
```

## Authentication

Send your account token in the `Authorization` header as a bearer token. For details, see [Management API overview](overview.md#authentication).

## Parameters

This endpoint takes no parameters.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `data[].model` | String | The model ID. |
| `data[].input_per_1k_micro` | Integer | The price per 1,000 input tokens, in micro-USD. |
| `data[].output_per_1k_micro` | Integer | The price per 1,000 output tokens, in micro-USD. |
| `data[].cached_per_1k_micro` | Integer or null | The price per 1,000 cached input tokens, in micro-USD. `null` if the model has no cached rate. |
| `data[].input_per_1k_usd` | String | The price per 1,000 input tokens, in USD. |
| `data[].output_per_1k_usd` | String | The price per 1,000 output tokens, in USD. |
| `data[].max_input_tokens` | Integer | The maximum number of input tokens. |
| `data[].max_output_tokens` | Integer | The maximum number of output tokens. |
| `meta.count` | Integer | The number of models. |

The context window that the Inference API reports in `context_length` is the sum of `max_input_tokens` and `max_output_tokens`.

## Errors

This endpoint returns the common errors only. For details, see [Management API overview](overview.md#errors).

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://engy.ai/api/v1/models \
  -H "Authorization: Bearer $ENGY_ACCOUNT_TOKEN"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.get(
    "https://engy.ai/api/v1/models",
    headers={"Authorization": f"Bearer {os.environ['ENGY_ACCOUNT_TOKEN']}"},
)
response.raise_for_status()

for model in response.json()["data"]:
    print(model["model"], model["input_per_1k_usd"], model["output_per_1k_usd"])
```
{% endtab %}
{% endtabs %}

Response:

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
      "max_input_tokens": 196608,
      "max_output_tokens": 65536
    }
  ],
  "meta": {
    "count": 1
  }
}
```
