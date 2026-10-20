---
description: Review the models that Engy serves, their checkpoints and capabilities, and their per-token prices.
---

# Models and pricing

This page lists the models that you can use through the Engy API, along with their capabilities and prices. It is intended for developers who are choosing a model or estimating costs.

## Models

Specify the model ID in the `model` parameter of your request.

| Model ID | Checkpoint | Context window (tokens) | Input | Zero data retention |
| --- | --- | --- | --- | --- |
| `deepseek-v4-flash-0731` | `deepseek-ai/DeepSeek-V4-Flash-0731` | 1,048,576 | Text | Yes |
| `deepseek-v4.1-flash` | `deepseek-ai/DeepSeek-V4.1-Flash` | 1,048,576 | Text, image | Yes |
| `deepseek-v4.1-flash-1017` | TBD | 1,048,576 | Text, image | Yes |
| `glm-5.2` | `zai-org/GLM-5.2` | 262,144 | Text | Yes |
| `glm-5.3` | TBD | 327,680 | Text | Yes |
| `glm-5.3-flash` | TBD | 327,680 | Text, image | Yes |
| `kimi-k3` | `moonshotai/Kimi-K3` | 1,113,088 | Text, image | Yes |
| `qwen3.6-35b-a3b` | `Qwen/Qwen3.6-35B-A3B-FP8` | 208,192 | Text | No |
| `qwen3.8-27b` | `Qwen/Qwen3.8-27B` | 1,001,536 | Text, image | Yes |

All models produce text output.

The checkpoint is the Hugging Face repository from which Engy serves the model. Each model is served from exactly one checkpoint. To verify a response, load this checkpoint. For details, see [Verify responses](verify-responses.md).

Models with zero data retention are served only on hardware that Engy operates. Prompts and outputs for these models are never stored or used for training.

## Pricing

Engy charges per token, with no subscriptions or minimum spend. Prices are in USD per 1 million tokens.

| Model ID | Input | Cached input | Output |
| --- | --- | --- | --- |
| `deepseek-v4-flash-0731` | $0.045 | $0.009 | $0.09 |
| `deepseek-v4.1-flash` | $0.04 | $0.008 | $0.08 |
| `deepseek-v4.1-flash-1017` | $0.04 | $0.008 | $0.08 |
| `glm-5.2` | $0.68 | $0.18 | $1.50 |
| `glm-5.3` | $0.98 | $0.18 | $3.08 |
| `glm-5.3-flash` | $0.135 | $0.027 | $0.45 |
| `kimi-k3` | $1.95 | $0.195 | $9.75 |
| `qwen3.6-35b-a3b` | $0.045 | $0.015 | $0.30 |
| `qwen3.8-27b` | $0.045 | $0.015 | $0.32 |

{% hint style="info" %}
Prices can change. For current prices, see the [pricing page](https://engy.ai/pricing) or query the models endpoint.
{% endhint %}

### Cached input

When a request repeats a prompt prefix that the server has already processed, Engy bills those tokens at the cached input rate. Caching is automatic and requires no configuration.

The number of cached tokens for each request is reported in `usage.prompt_tokens_details.cached_tokens`.

## List models with the API

To retrieve the current model list, including capabilities and prices, call the models endpoint:

```bash
curl https://api.engy.ai/v1/models
```

Each model object contains the following fields:

| Field | Description |
| --- | --- |
| `id` | The model ID to use in requests. |
| `input_modalities` | The input types that the model accepts, such as `text` and `image`. |
| `output_modalities` | The output types that the model produces. |
| `context_length` | The maximum number of tokens in the context window. |
| `pricing.prompt` | The price per input token, in USD. |
| `pricing.input_cache_read` | The price per cached input token, in USD. |
| `pricing.completion` | The price per output token, in USD. |
