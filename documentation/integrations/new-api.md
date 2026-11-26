---
description: Add Engy as an upstream channel in New API or One API, keep routing preferences, and measure latency correctly.
---

# New API

This page explains how to add Engy as an upstream channel in [New API](https://github.com/QuantumNous/new-api) or a compatible relay such as One API. It is intended for operators who resell or aggregate model access through a relay.

New API forwards requests to Engy's OpenAI-compatible endpoints. Most requests work without extra configuration. The settings below matter for routing preferences, error handling, and latency measurement.

## Add the channel

{% stepper %}
{% step %}
### Create an API key

In the Engy dashboard, create an API key for the relay. Use a separate key for each relay, so that you can see its usage and revoke it on its own. For details, see [API keys](../account-and-billing/api-keys.md).
{% endstep %}

{% step %}
### Add an OpenAI channel

In New API, add a channel with the following settings:

| Setting | Value |
| --- | --- |
| Type | **OpenAI** |
| Base URL | `https://api.engy.ai` |
| Key | Your Engy API key (`sk-engy-...`) |
| Models | The Engy model IDs that you want to offer, for example `kimi-k3.1-pro` |

Enter the base URL without `/v1`. New API appends the endpoint path itself.

To fill the model list automatically, use the channel's model fetch option. It calls `GET https://api.engy.ai/v1/models`.
{% endstep %}

{% step %}
### Test the channel

Use the channel test in New API, or send a request through the relay:

```bash
curl https://<your-relay>/v1/chat/completions \
  -H "Authorization: Bearer <relay-token>" \
  -H "Content-Type: application/json" \
  -d '{"model": "kimi-k3.1-pro", "messages": [{"role": "user", "content": "Hello"}], "stream": true}'
```
{% endstep %}
{% endstepper %}

## Keep the `provider` object

By default, New API rebuilds each request body from its own list of known fields. The Engy `provider` object is not in that list, so New API drops it. When that happens, the request is served without your requirements, and no error is returned.

The `provider` object carries these fields:

* `zdr`: serve only on ZDR-certified workers
* `region_only` and `region_exclude`: restrict the worker region
* `require_proof`: request a proof with the response

For details, see **The provider object** in the Inference API overview of the API reference.

To keep the `provider` object, choose one of the following:

* Turn on request body pass-through for the Engy channel (`pass_through_body_enabled`). New API then forwards the client's body unchanged.
* Use the channel's parameter override to add a fixed `provider` object to every request, for example `{"provider": {"zdr": true}}`.

{% hint style="warning" %}
If your buyers rely on ZDR or region routing, verify the setting before you go live. Send a request with an unsupported region value, such as `"region_only": ["invalid"]`. Engy rejects it with status `400` only if the `provider` object reaches Engy.
{% endhint %}

New API keeps these request fields without extra configuration: `reasoning_effort`, `thinking`, `chat_template_kwargs`, `top_k`, `logprobs`, and `stream_options`.

## Reasoning output

Engy streams reasoning in the `reasoning_content` field of each delta. New API forwards it unchanged.

Do not turn on the channel's option that converts reasoning into content (`thinking_to_content`), unless your clients expect reasoning inside `<think>` tags in `content`.

## Usage and billing

Engy includes a `usage` object in the final chunk of every stream, even if the request does not set `stream_options.include_usage`. The object includes `prompt_tokens_details.cached_tokens`. New API reads these values, so the relay bills its users the same token counts that Engy reports.

Engy does not charge for a failed request.

## Errors and retries

New API decides whether to retry a failed request on another channel from the HTTP status code.

* When a streamed request fails before any output, Engy answers with the error's HTTP status and a JSON error body, provided the error occurs within 3 seconds. New API can then retry on another channel.
* An error that occurs after Engy has started the stream arrives as a `data:` line with an `error` object, inside a response with status `200`. New API does not inspect this object. It does not retry the request, and it estimates usage for its own billing.
* New API never retries status `504`, which Engy returns when the model server times out.
* For non-streaming requests that run longer than 15 seconds, Engy sends the status `200` early to keep the connection open. A late error then appears in the response body. New API detects the error but does not retry it. Use streaming for long requests.

For the error format, see **Errors** in the Inference API overview of the API reference.

## Measure latency

New API logs a first response time for each request. It is measured from when the relay sends the request until the first `data:` line arrives from Engy.

* Engy sends the first `data:` line together with the first generated token. Reasoning tokens count as generated tokens, so for a reasoning model the first response time is the time to the first reasoning token, not to the first answer token.
* The time that your clients measure also includes the network path to the relay and from the relay to Engy. Large prompts take longer to upload, so this overhead grows with the prompt size.

When you benchmark time to first token through a relay:

* Group results by the `usage.prompt_tokens` that Engy returns, not by an estimate of the prompt size. The number of characters per token depends on the model's tokenizer and on the text.
* Set `max_tokens` high enough that the answer can start after the reasoning. If the output limit is reached during reasoning, the response contains no answer text.
* Start each prompt with unique text, so that prompt caching does not shorten the measured time.
* Keep the concurrency fixed, and collect at least 20 samples for each prompt size before you report percentiles.
