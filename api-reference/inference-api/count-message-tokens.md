---
description: Reference for POST /v1/messages/count_tokens, which estimates the input tokens of a message request.
---

# Count message tokens

This page is the reference for the token counting endpoint. It is intended for developers who use Anthropic clients that check the prompt size before they send a request.

## Endpoint

```http
POST https://api.engy.ai/v1/messages/count_tokens
```

## Authentication

This endpoint does not require authentication.

## Parameters

### Body parameters

The request body has the same format as [Create a message](create-message.md#body-parameters). The endpoint uses `system`, `messages`, and `tools`, and ignores the other parameters.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `input_tokens` | Integer | The estimated number of input tokens. |

The value is an estimate based on the size of the text, at about 4 bytes per token. It is not a count by the model's tokenizer. The actual count, which Engy uses for billing, is returned in the `usage` field of each response.

## Errors

This endpoint does not return endpoint-specific errors.

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/messages/count_tokens \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kimi-k3",
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

result = client.messages.count_tokens(
    model="kimi-k3",
    system="You are a concise assistant.",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
)

print(result.input_tokens)
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "input_tokens": 14
}
```
