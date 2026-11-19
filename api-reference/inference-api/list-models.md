---
description: Reference for GET /v1/models, which lists the available models with their capabilities and prices.
---

# List models

This page is the reference for the models endpoint of the Inference API. It is intended for developers who select models or display prices in their applications.

## Endpoint

```http
GET https://api.engy.ai/v1/models
```

## Authentication

Authentication is optional. If you send a valid API key in the `Authorization` header, the prices reflect any pricing that applies to your account. Without a key, the endpoint returns list prices.

## Parameters

This endpoint takes no parameters.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `object` | String | Always `list`. |
| `data` | Array | The model objects. |
| `data[].id` | String | The model ID to use in requests. |
| `data[].object` | String | Always `model`. |
| `data[].created` | Integer | The time of the response, as a Unix timestamp in seconds. |
| `data[].owned_by` | String | Always `engy`. |
| `data[].input_modalities` | Array | The input types that the model accepts, such as `text` and `image`. |
| `data[].output_modalities` | Array | The output types. Always `["text"]`. |
| `data[].context_length` | Integer | The maximum number of tokens in the context window. |
| `data[].max_model_len` | Integer | The same value as `context_length`. |
| `data[].pricing.prompt` | String | The price per input token, in USD. |
| `data[].pricing.completion` | String | The price per output token, in USD. |
| `data[].pricing.input_cache_read` | String | The price per cached input token, in USD. Absent if the model has no cached rate. |
| `data[].pricing.request` | String | The price per request, in USD. Always `"0"`. |
| `data[].pricing.image` | String | The price per image, in USD. Always `"0"`. |
| `data[].x_engy.price_source` | String | Present only if the prices are specific to your account. |

Model aliases, such as `deepseek-v4.1-flash-1017`, appear as separate entries with the same values as the model they refer to.

## Errors

This endpoint does not return endpoint-specific errors. An invalid API key does not cause an error. The endpoint returns list prices instead.

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/models
```
{% endtab %}

{% tab title="Python" %}
```python
import os

from openai import OpenAI

client = OpenAI(
    base_url="https://api.engy.ai/v1",
    api_key=os.environ["ENGY_API_KEY"],
)

for model in client.models.list():
    print(model.id)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "object": "list",
  "data": [
    {
      "id": "glm-5.2",
      "object": "model",
      "created": 1792281600,
      "owned_by": "engy",
      "input_modalities": ["text"],
      "output_modalities": ["text"],
      "context_length": 262144,
      "max_model_len": 262144,
      "pricing": {
        "prompt": "0.00000068",
        "completion": "0.0000015",
        "request": "0",
        "image": "0",
        "input_cache_read": "0.00000018"
      }
    }
  ]
}
```
