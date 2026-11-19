---
description: Reference for POST /v1/messages, which generates a response in the Anthropic Messages format.
---

# Create a message

This page is the reference for the Anthropic-compatible messages endpoint. It is intended for developers who use Anthropic clients or send requests in the Anthropic Messages format.

## Endpoint

```http
POST https://api.engy.ai/v1/messages
```

## Authentication

Send your API key in the `x-api-key` header, or in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `model` | String | Yes | The model ID. A leading `vendor/` segment is ignored. |
| `messages` | Array | Yes | The conversation, as an array of messages with `role` (`user` or `assistant`) and `content`. |
| `max_tokens` | Integer | No | The maximum number of output tokens. If the prompt plus `max_tokens` exceeds the context window, Engy reduces `max_tokens` to fit. |
| `system` | String or array | No | The system prompt, as a string or an array of text blocks. |
| `stream` | Boolean | No | If `true`, the response is streamed as server-sent events. |
| `temperature` | Number | No | The sampling temperature. |
| `top_p` | Number | No | The nucleus sampling threshold. |
| `top_k` | Integer | No | The number of highest-probability tokens to sample from. |
| `stop_sequences` | Array | No | Sequences at which generation stops. |
| `tools` | Array | No | The tools that the model can call, each with `name`, `description`, and `input_schema`. |
| `tool_choice` | Object | No | `auto`, `any`, `none`, or `tool` with a `name`. |
| `thinking` | Object | No | `{"type": "enabled"}` or `{"type": "disabled"}`. The `budget_tokens` value is ignored. |
| `provider` | Object | No | Worker requirements. For details, see [The provider object](overview.md#the-provider-object). |

### Content blocks

| Block type | Supported in | Description |
| --- | --- | --- |
| `text` | User and assistant messages | Text content. |
| `image` | User messages | An image with a `base64` source. Images with a `url` source fail with status `400`. |
| `tool_use` | Assistant messages | A tool call that the model made. |
| `tool_result` | User messages | The result of a tool call, with `tool_use_id`. |

Other block types, such as documents, are ignored. The `cache_control` and `metadata` fields are also ignored.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `id` | String | The message ID. |
| `type` | String | Always `message`. |
| `role` | String | Always `assistant`. |
| `model` | String | The model ID, as sent in the request. |
| `content` | Array | The `text` and `tool_use` blocks of the response. |
| `stop_reason` | String | `end_turn`, `max_tokens`, `tool_use`, or `refusal`. |
| `stop_sequence` | Null | Always `null`. |
| `usage.input_tokens` | Integer | The number of input tokens that were not read from the cache. |
| `usage.cache_read_input_tokens` | Integer | The number of input tokens read from the cache. |
| `usage.cache_creation_input_tokens` | Integer | Always `0`. Engy does not charge for cache writes. |
| `usage.output_tokens` | Integer | The number of output tokens. |

Non-streaming responses do not contain `thinking` blocks. Streaming responses contain `thinking` blocks unless thinking is disabled. For the event sequence, see [Streaming](https://docs.engy.ai/using-the-api/streaming#other-stream-formats).

## Errors

Errors use the Anthropic format. For details, see [Inference API overview](overview.md#errors).

| Status | Error type | Cause |
| --- | --- | --- |
| `400` | `invalid_request_error` | The request is invalid, or the prompt exceeds the context window. The message reads `prompt is too long: N tokens > M maximum`. |
| `401` | `authentication_error` | The API key is missing or invalid. |
| `402` | `api_error` | The available balance does not cover the request. |
| `404` | `not_found_error` | The model does not exist or is unavailable. |
| `413` | `request_too_large` | The images exceed the size or count limit. |
| `429` | `rate_limit_error` | Capacity is exhausted, or a concurrency limit is reached. |
| `503` | `overloaded_error` | No worker meets the `provider` requirements. |
| `504` | `api_error` | The model server timed out. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/messages \
  -H "x-api-key: $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kimi-k3",
    "max_tokens": 1024,
    "system": "You are a concise assistant.",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os

from anthropic import Anthropic

client = Anthropic(
    base_url="https://api.engy.ai",
    api_key=os.environ["ENGY_API_KEY"],
)

message = client.messages.create(
    model="kimi-k3",
    max_tokens=1024,
    system="You are a concise assistant.",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
)

print(message.content[0].text)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "id": "msg_5c7e2b9d1f3a4e6b8c0d2f4a",
  "type": "message",
  "role": "assistant",
  "model": "kimi-k3",
  "content": [{"type": "text", "text": "The capital of France is Paris."}],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 24,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0,
    "output_tokens": 9
  }
}
```
