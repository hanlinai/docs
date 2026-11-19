---
description: Reference for POST /v1/responses, which generates a response in the OpenAI Responses format without stored state.
---

# Create a response

This page is the reference for the Responses-compatible endpoint. It is intended for developers who use clients that send requests in the OpenAI Responses format, such as Codex.

## Endpoint

```http
POST https://api.engy.ai/v1/responses
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `model` | String | Yes | The model ID. A leading `vendor/` segment is ignored. |
| `input` | String or array | Yes | The conversation. A string is treated as one user message. For an array, see [Input items](#input-items). |
| `instructions` | String | No | The system prompt. |
| `max_output_tokens` | Integer | No | The maximum number of output tokens. |
| `stream` | Boolean | No | If `true`, the response is streamed as server-sent events. |
| `temperature` | Number | No | The sampling temperature. |
| `top_p` | Number | No | The nucleus sampling threshold. |
| `tools` | Array | No | The tools that the model can call. Only tools with `type` set to `function` are supported. |
| `tool_choice` | String or object | No | `auto`, `required`, `none`, or a named function. |
| `parallel_tool_calls` | Boolean | No | Whether the model can return several tool calls in one response. |
| `provider` | Object | No | Worker requirements. For details, see [The provider object](overview.md#the-provider-object). |

The endpoint is stateless. The `store` and `previous_response_id` parameters have no effect, so send the full conversation in `input` with each request. The `reasoning`, `text`, and `prompt_cache_key` parameters are also ignored.

### Input items

| Item type | Description |
| --- | --- |
| `message` | A message with `role` (`user`, `assistant`, `system`, or `developer`) and text `content`. The `developer` role is treated as `system`. |
| `function_call` | A tool call that the model made, with `call_id`, `name`, and `arguments`. |
| `function_call_output` | The result of a tool call, with `call_id` and `output`. |

Images, reasoning items, and item references are ignored.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `id` | String | The response ID. |
| `object` | String | Always `response`. |
| `created_at` | Integer | The creation time, as a Unix timestamp in seconds. |
| `status` | String | `completed`, or `incomplete` if the output reached `max_output_tokens`. |
| `model` | String | The model ID, as sent in the request. |
| `error` | Null | Always `null` for a successful response. |
| `incomplete_details` | Object or null | `{"reason": "max_output_tokens"}` if `status` is `incomplete`. |
| `output` | Array | A `message` item with `output_text` content, followed by one `function_call` item for each tool call. |
| `usage.input_tokens` | Integer | The number of input tokens. |
| `usage.input_tokens_details.cached_tokens` | Integer | The number of input tokens billed at the cached rate. |
| `usage.output_tokens` | Integer | The number of output tokens. |
| `usage.output_tokens_details.reasoning_tokens` | Integer | The number of reasoning tokens. |
| `usage.total_tokens` | Integer | The sum of input and output tokens. |

The response does not contain reasoning items. Streaming responses end with a `response.completed` event, or with `response.failed` if an error occurs.

## Errors

| Status | Cause |
| --- | --- |
| `400` | The request is invalid, or the prompt plus `max_output_tokens` exceeds the context window. |
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
curl https://api.engy.ai/v1/responses \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5.2",
    "instructions": "You are a concise assistant.",
    "input": "What is the capital of France?"
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

response = client.responses.create(
    model="glm-5.2",
    instructions="You are a concise assistant.",
    input="What is the capital of France?",
)

print(response.output_text)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "id": "resp_7a3c5e9b1d2f4a6c8e0b2d4f",
  "object": "response",
  "created_at": 1792281600,
  "status": "completed",
  "model": "glm-5.2",
  "error": null,
  "incomplete_details": null,
  "output": [
    {
      "type": "message",
      "id": "msg_2e4a6c8b0d1f3a5c7e9b1d3f",
      "status": "completed",
      "role": "assistant",
      "content": [
        {"type": "output_text", "annotations": [], "text": "The capital of France is Paris."}
      ]
    }
  ],
  "usage": {
    "input_tokens": 22,
    "input_tokens_details": {"cached_tokens": 0},
    "output_tokens": 19,
    "output_tokens_details": {"reasoning_tokens": 11},
    "total_tokens": 41
  }
}
```
