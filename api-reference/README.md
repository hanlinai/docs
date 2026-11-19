---
description: Reference documentation for the Engy Inference API and the Engy Management API.
---

# API reference

This space contains the reference for each endpoint of the Engy APIs. It is intended for developers who need exact parameters, response fields, and error codes. For guides and concepts, see the [Engy documentation](https://docs.engy.ai).

## APIs

| API | Base URL | Credential | Purpose |
| --- | --- | --- | --- |
| [Inference API](inference-api/overview.md) | `https://api.engy.ai` | API key (`sk-engy-`) | Generate model output. Requests are charged to your credit balance. |
| [Management API](management-api/overview.md) | `https://engy.ai/api/v1` | Account token (`rk-engy-`) | Read your account data, such as balance and usage. |

## Inference API endpoints

| Endpoint | Description |
| --- | --- |
| [`POST /v1/chat/completions`](inference-api/create-chat-completion.md) | Creates a chat completion in the OpenAI format. |
| [`POST /v1/completions`](inference-api/create-completion.md) | Creates a completion for a raw prompt. |
| [`POST /v1/messages`](inference-api/create-message.md) | Creates a message in the Anthropic format. |
| [`POST /v1/messages/count_tokens`](inference-api/count-message-tokens.md) | Estimates the input tokens of a message request. |
| [`POST /v1/responses`](inference-api/create-response.md) | Creates a response in the OpenAI Responses format. |
| [`GET /v1/models`](inference-api/list-models.md) | Lists the available models. |
| [`POST /v2/video_generation`](inference-api/create-video-task.md) | Submits a video generation task. |
| [`GET /v2/query/video_generation/{task_id}`](inference-api/get-video-task.md) | Returns the status of a video task. |
| [`GET /v2/query/video_generation`](inference-api/list-video-tasks.md) | Lists your video tasks. |
| [`DELETE /v2/video_generation/{task_id}`](inference-api/delete-video-task.md) | Cancels or deletes a video task. |
| [`GET /v2/video_generation/{task_id}/content`](inference-api/download-video.md) | Downloads a video. |

## Management API endpoints

| Endpoint | Description |
| --- | --- |
| [`GET /api/v1/balance`](management-api/get-balance.md) | Returns your balance. |
| [`GET /api/v1/usage`](management-api/get-usage.md) | Returns your requests and spending. |
| [`GET /api/v1/models`](management-api/list-models.md) | Returns the model catalog with prices and token limits. |
