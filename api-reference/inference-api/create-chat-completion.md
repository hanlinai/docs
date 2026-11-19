---
description: Reference for POST /v1/chat/completions, which generates a response to a list of messages.
---

# Create a chat completion

This page is the reference for the chat completions endpoint. It is intended for developers who send requests in the OpenAI Chat Completions format.

## Endpoint

```http
POST https://api.engy.ai/v1/chat/completions
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `model` | String | Yes | The model ID. A leading `vendor/` segment is ignored. |
| `messages` | Array | Yes | The conversation, as an array of [message objects](#message-object). |
| `max_tokens` | Integer | No | The maximum number of output tokens. Defaults to the model's maximum output length. Values above that maximum are reduced to it. |
| `max_completion_tokens` | Integer | No | The same as `max_tokens`. Takes precedence if you send both. |
| `stream` | Boolean | No | If `true`, the response is streamed as server-sent events. Defaults to `false`. |
| `stream_options` | Object | No | Set `include_usage` to `true` to receive an additional usage chunk at the end of the stream. |
| `temperature` | Number | No | The sampling temperature. |
| `top_p` | Number | No | The nucleus sampling threshold. |
| `top_k` | Integer | No | The number of highest-probability tokens to sample from. |
| `min_p` | Number | No | The minimum probability of a token, relative to the most likely token. |
| `stop` | String or array | No | One or more sequences at which generation stops. |
| `seed` | Integer | No | A seed for sampling. |
| `presence_penalty` | Number | No | Penalizes tokens that already appear in the output. |
| `frequency_penalty` | Number | No | Penalizes tokens in proportion to how often they appear in the output. |
| `tools` | Array | No | The tools that the model can call. For details, see [Tool calling](https://docs.engy.ai/using-the-api/tool-calling). |
| `tool_choice` | String or object | No | `auto`, `required`, `none`, or a named function. |
| `parallel_tool_calls` | Boolean | No | Whether the model can return several tool calls in one response. |
| `response_format` | Object | No | Passed to the model. The output is not validated against the format. |
| `reasoning_effort` | String | No | `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, or `max`. For details, see [Reasoning](https://docs.engy.ai/using-the-api/reasoning). |
| `reasoning` | Object | No | An alternative to `reasoning_effort`. Accepts `effort` and `enabled`. |
| `chat_template_kwargs` | Object | No | Arguments for the model's chat template, such as `enable_thinking`. |
| `logprobs` | Boolean | No | If `true`, the response includes log probabilities of the output tokens. |
| `top_logprobs` | Integer | No | The number of most likely tokens to return for each position, from 0 to 30. A value above 0 turns on `logprobs`. |
| `prompt_cache_key` | String | No | Requests with the same key are sent to the same server, to improve cache hits. For details, see [Prompt caching](https://docs.engy.ai/using-the-api/prompt-caching). |
| `user` | String | No | An identifier for the end user. Used as the cache key when `prompt_cache_key` is not set. |
| `provider` | Object | No | Worker requirements. For details, see [The provider object](overview.md#the-provider-object). |

Other OpenAI parameters are passed to the model unchanged. For `kimi-k3`, `temperature`, `top_p`, `n`, `presence_penalty`, and `frequency_penalty` are fixed. For details, see [API overview](https://docs.engy.ai/using-the-api/overview#model-specific-behavior).

### Message object

| Field | Type | Description |
| --- | --- | --- |
| `role` | String | `system`, `user`, `assistant`, or `tool`. |
| `content` | String or array | The message text, or an array of content parts. A part is either `{"type": "text", "text": "..."}` or `{"type": "image_url", "image_url": {"url": "data:image/..."}}`. |
| `tool_calls` | Array | For `assistant` messages: the tool calls that the model made. |
| `tool_call_id` | String | For `tool` messages: the ID of the tool call that this message answers. |

If the last message has the `assistant` role, the model continues that message, and thinking is turned off.

Images must be Base64 data URIs. For limits, see [Image input](https://docs.engy.ai/using-the-api/image-input).

## Response

A non-streaming request returns a chat completion object. A streaming request returns a sequence of chunk objects. For the stream format, see [Streaming](https://docs.engy.ai/using-the-api/streaming).

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `id` | String | The completion ID. |
| `object` | String | Always `chat.completion`. |
| `created` | Integer | The creation time, as a Unix timestamp in seconds. |
| `model` | String | The model ID, as sent in the request. |
| `choices[].index` | Integer | The index of the choice. |
| `choices[].message.role` | String | Always `assistant`. |
| `choices[].message.content` | String or null | The generated text. |
| `choices[].message.reasoning_content` | String or null | The model's reasoning, for reasoning models. |
| `choices[].message.tool_calls` | Array | The tool calls, if the model called tools. |
| `choices[].finish_reason` | String | `stop`, `length`, or `tool_calls`. |
| `choices[].logprobs` | Object or null | The log probabilities, if requested. |
| `usage.prompt_tokens` | Integer | The number of input tokens. |
| `usage.completion_tokens` | Integer | The number of output tokens, including reasoning tokens. |
| `usage.total_tokens` | Integer | The sum of input and output tokens. |
| `usage.prompt_tokens_details.cached_tokens` | Integer | The number of input tokens billed at the cached rate. |
| `usage.completion_tokens_details.reasoning_tokens` | Integer | The number of reasoning tokens. |
| `x_engy.request_id` | String | The unique identifier of the request. |
| `x_engy.charged_micro` | Integer | The amount charged, in micro-USD. |
| `proof` | Object | The proof, if you set `provider.require_proof`. For the format, see [Verify responses](https://docs.engy.ai/get-started/verify-responses). |

## Errors

| Status | Cause |
| --- | --- |
| `400` | The request is invalid, the prompt plus `max_tokens` exceeds the context window, or the model does not accept the input type. |
| `401` | The API key is missing or invalid. |
| `402` | The available balance does not cover the request. |
| `404` | The model does not exist or is unavailable. |
| `413` | The media exceed the size or count limit. |
| `429` | Capacity is exhausted, or a concurrency limit is reached. |
| `503` | No worker meets the `provider` requirements. |
| `504` | The model server timed out. |

For streaming requests, errors that occur after the stream starts are returned in the stream. For details, see [Errors and limits](https://docs.engy.ai/using-the-api/errors-and-limits).

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/chat/completions \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5.2",
    "max_tokens": 1024,
    "messages": [
      {"role": "system", "content": "You are a concise assistant."},
      {"role": "user", "content": "What is the capital of France?"}
    ]
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

response = client.chat.completions.create(
    model="glm-5.2",
    max_tokens=1024,
    messages=[
        {"role": "system", "content": "You are a concise assistant."},
        {"role": "user", "content": "What is the capital of France?"},
    ],
)

print(response.choices[0].message.content)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "id": "chatcmpl-3f2a9c41d8e04b7a9c1e5d20",
  "object": "chat.completion",
  "created": 1792281600,
  "model": "glm-5.2",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The capital of France is Paris.",
        "reasoning_content": "The user asks for the capital of France."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 22,
    "completion_tokens": 19,
    "total_tokens": 41,
    "prompt_tokens_details": {"cached_tokens": 0},
    "completion_tokens_details": {"reasoning_tokens": 11}
  },
  "x_engy": {
    "request_id": "3f2a9c41d8e04b7a9c1e5d20a7b6c3e1",
    "charged_micro": 43
  }
}
```
