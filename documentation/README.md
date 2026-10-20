---
description: Learn what Engy provides and how to start sending inference requests.
---

# Introduction

Engy provides verified inference for open-source large language models through an API that is compatible with OpenAI and Anthropic clients. This page introduces the service for developers who want to integrate Engy into their applications and tools.

## What Engy provides

| Capability | Description |
| --- | --- |
| Compatible API | Use existing OpenAI and Anthropic clients by changing the base URL and API key. |
| Verified inference | Request a cryptographic proof with any response, and verify that the advertised model produced the output. |
| Zero data retention | For supported models, request content is held in memory only and is not stored. |
| Open models | Access frontier open-source models, such as `glm-5.2` and `kimi-k3`, through one API. |
| Prepaid billing | Pay as you go with prepaid credits. There are no subscriptions. |

## How it works

1. You send a request to the Engy API with your API key.
2. Engy routes the request to a GPU worker that serves the requested model.
3. The worker returns the response, and Engy charges the cost to your credit balance.
4. If you requested a proof, the response includes it. You can recompute the proof against the model weights to verify the output.

For details, see [Verify responses](get-started/verify-responses.md).

## Get started

* [Quickstart](get-started/quickstart.md): Create an account and send your first request in a few minutes.
* [Dashboard](https://engy.ai/dashboard): Manage your API keys, credits, and usage.
