---
description: Reference for POST /v1/completions, which continues a raw prompt without a chat template.
---

# Create a completion

This page is the reference for the completions endpoint. It is intended for developers who send raw prompts or score text with log probabilities.

## Endpoint

```http
POST https://api.engy.ai/v1/completions
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `model` | String | Yes | The model ID. A leading `vendor/` segment is ignored. |
| `prompt` | String or array | Yes | The prompt, as a string or an array of token IDs. For a batch, send an array of strings or an array of token ID arrays. |
| `max_tokens` | Integer | No | The maximum number of output tokens. Must be at least `1`. Defaults to the model's maximum output length. |
| `stream` | Boolean | No | Must be `false` or omitted. Streaming is not supported on this endpoint. |
| `temperature` | Number | No | The sampling temperature. |
| `top_p` | Number | No | The nucleus sampling threshold. |
| `stop` | String or array | No | One or more sequences at which generation stops. |
| `seed` | Integer | No | A seed for sampling. |
| `logprobs` | Integer | No | The number of most likely tokens to return for each position, from 1 to 30. |
| `echo` | Boolean | No | If `true`, the response includes the prompt and its log probabilities. The prompt can contain at most 1,024 tokens. |
| `logprob_start_len` | Integer | No | The token offset in the prompt from which to return log probabilities. At most 1,024 tokens can be scored. |
| `provider` | Object | No | Worker requirements. For details, see [The provider object](overview.md#the-provider-object). |

Engy does not apply a chat template to the prompt. For usage guidance, see [Raw prompts and log probabilities](https://docs.engy.ai/using-the-api/raw-prompts-and-logprobs).

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `id` | String | The completion ID. |
| `object` | String | Always `text_completion`. |
| `created` | Integer | The creation time, as a Unix timestamp in seconds. |
| `model` | String | The model ID, as sent in the request. |
| `choices[].index` | Integer | The index of the choice. |
| `choices[].text` | String | The generated text. |
| `choices[].finish_reason` | String | `stop` or `length`. |
| `choices[].logprobs` | Object or null | The log probabilities, if requested. With `logprob_start_len`, contains `token_logprobs` and `token_ids`. |
| `usage` | Object | The token counts, in the same format as for [chat completions](create-chat-completion.md#response-fields). |
| `x_engy.request_id` | String | The unique identifier of the request. |
| `x_engy.charged_micro` | Integer | The amount charged, in micro-USD. |

## Errors

| Status | Cause |
| --- | --- |
| `400` | The request is invalid. For example, `stream` is `true`, `max_tokens` or `logprobs` is `0`, the scored span exceeds 1,024 tokens, or the prompt plus `max_tokens` exceeds the context window. |
| `401` | The API key is missing or invalid. |
| `402` | The available balance does not cover the request. |
| `404` | The model does not exist or is unavailable. |
| `429` | Capacity is exhausted, or a concurrency limit is reached. |
| `503` | No worker meets the `provider` requirements. |
| `504` | The model server timed out. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/completions \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5.2",
    "prompt": "The capital of France is",
    "max_tokens": 8,
    "temperature": 0
  }'
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

response = client.completions.create(
    model="glm-5.2",
    prompt="The capital of France is",
    max_tokens=8,
    temperature=0,
)

print(response.choices[0].text)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "id": "cmpl-8b1d4e7f2a9c4c03b6e5f1a2",
  "object": "text_completion",
  "created": 1792281600,
  "model": "glm-5.2",
  "choices": [
    {"index": 0, "text": " Paris.", "finish_reason": "stop", "logprobs": null}
  ],
  "usage": {
    "prompt_tokens": 5,
    "completion_tokens": 3,
    "total_tokens": 8,
    "prompt_tokens_details": {"cached_tokens": 0},
    "completion_tokens_details": {"reasoning_tokens": 0}
  },
  "x_engy": {
    "request_id": "8b1d4e7f2a9c4c03b6e5f1a2d3c4b5a6",
    "charged_micro": 8
  }
}
```
