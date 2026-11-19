---
description: Base URL, authentication, common fields, and error format of the Engy Inference API.
---

# Inference API overview

This page describes the conventions that apply to every endpoint of the Engy Inference API. It is intended for developers who call the API directly or through an SDK.

## Base URL

```text
https://api.engy.ai
```

Text endpoints use the `/v1` path prefix. Video task endpoints use the `/v2` path prefix.

## Authentication

Send your API key in the `Authorization` header:

```http
Authorization: Bearer sk-engy-...
```

The `POST /v1/messages` endpoint also accepts the key in the `x-api-key` header. The `GET /v1/models` and `POST /v1/messages/count_tokens` endpoints do not require a key.

To create an API key, see [API keys](https://docs.engy.ai/account-and-billing/api-keys).

## Compatibility

| Endpoint | Compatible with |
| --- | --- |
| `POST /v1/chat/completions` | OpenAI Chat Completions |
| `POST /v1/completions` | OpenAI Completions |
| `POST /v1/messages` | Anthropic Messages |
| `POST /v1/messages/count_tokens` | Anthropic token counting |
| `POST /v1/responses` | OpenAI Responses, without stored state |
| `GET /v1/models` | OpenAI Models |
| `/v2/video_generation` endpoints | MiniMax video generation, version 2 |

## The provider object

The `provider` object sets requirements for the worker that serves a request. It is accepted on `/v1/chat/completions`, `/v1/completions`, `/v1/messages`, and `/v1/responses`.

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `zdr` | Boolean | `false` | If `true`, only ZDR-certified workers serve the request. |
| `region_only` | Array of strings | None | Only workers in these regions serve the request. Values: `eu`, `us`, `cn`, `apac`, `other`. |
| `region_exclude` | Array of strings | None | Workers in these regions do not serve the request. Values: `eu`, `us`, `cn`, `apac`, `other`. |
| `require_proof` | Boolean | `false` | If `true`, only workers that can produce proofs serve the request, and the response includes a `proof` field. |

A worker must meet all requirements to serve the request. If no available worker meets them, the request fails with status `503`. An invalid region value fails with status `400`.

Engy removes the `provider` object before the request reaches the model. For details, see [Data retention](https://docs.engy.ai/using-the-api/data-retention), [Region routing](https://docs.engy.ai/using-the-api/region-routing), and [Verify responses](https://docs.engy.ai/get-started/verify-responses).

## Response headers

| Header | Description |
| --- | --- |
| `X-Engy-Request-Id` | The unique identifier of the request. Returned on every inference response, including errors. |
| `X-Engy-Charged-Micro` | The amount charged, in micro-USD. Returned on non-streaming responses from `/v1/chat/completions`, `/v1/completions`, and `/v1/messages`. |
| `Retry-After` | The number of seconds to wait before you retry. Returned on non-streaming `429` responses from `/v1/chat/completions`, `/v1/completions`, and `/v1/responses`. |

## Errors

The OpenAI-compatible endpoints return errors in the following format:

```json
{
  "error": {
    "message": "model 'example' not found",
    "type": "not_found_error",
    "param": null,
    "code": 404
  }
}
```

The `/v1/messages` endpoint returns errors in the Anthropic format:

```json
{
  "type": "error",
  "error": {"type": "not_found_error", "message": "model 'example' not found"}
}
```

For the status codes, error types, and retry guidance, see [Errors and limits](https://docs.engy.ai/using-the-api/errors-and-limits).
