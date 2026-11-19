---
description: Reference for POST /v2/video_generation, which submits a video generation task.
---

# Create a video task

This page is the reference for submitting a video generation task. It is intended for developers who generate video clips. For a guide, see [Video generation](https://docs.engy.ai/using-the-api/video-generation).

## Endpoint

```http
POST https://api.engy.ai/v2/video_generation
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `model` | String | Yes | The video model ID, such as `MiniMax-H3`. A leading `vendor/` segment is ignored. |
| `content` | Array | Yes | The [content items](#content-items). Must include at least one non-empty text item. |
| `duration` | Integer | No | The length of the clip, in whole seconds, from 4 to 15. Defaults to `5`. |
| `ratio` | String | No | `adaptive`, `21:9`, `16:9`, `4:3`, `1:1`, `3:4`, or `9:16`. Defaults to `adaptive`. |
| `resolution` | String | No | `768P`, which is the only supported value. Defaults to `768P`. |
| `seed` | Integer | No | A seed for generation. |

The `callback_url` parameter is not supported. A request that sets it fails with status `400`.

### Content items

| Item | Fields | Description |
| --- | --- | --- |
| Text | `type`: `text`, `text` | Part of the prompt. Engy joins all text items into one prompt of at most 7,000 characters. |
| Image | `type`: `image_url`, `image_url.url`, `role` | A frame, as a Base64 data URI. Set `role` to `first_frame` or `last_frame`. An image without a role is the first frame. |

Frames must be JPEG, PNG, or WebP, from 256 to 5,760 pixels per side, with an aspect ratio from 0.4 to 2.5, and at most 30 MB each. You can send at most one first frame and one last frame.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `task_id` | String | The ID of the new task. |
| `status` | String | Always `queued`. |
| `base_resp.status_code` | Integer | `0` on success. |
| `base_resp.status_msg` | String | `success` on success. |

When Engy accepts the task, it reserves credit for the full requested duration.

## Errors

| Status | Cause |
| --- | --- |
| `400` | The request is invalid. For example, the prompt is missing or too long, a value is out of range, or an image is a URL instead of a data URI. |
| `401` | The API key is missing or invalid. |
| `402` | The available balance does not cover the requested duration. |
| `404` | The model is not offered. |
| `413` | A frame exceeds 30 MB, or the request body is too large. |
| `429` | The account has 20 queued or running tasks. The response includes a `Retry-After` header. |
| `503` | Too many uploads are in progress. Retry after the time in the `Retry-After` header. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v2/video_generation \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiniMax-H3",
    "content": [{"type": "text", "text": "A red fox trots across fresh snow at dawn"}],
    "duration": 6,
    "ratio": "16:9"
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.post(
    "https://api.engy.ai/v2/video_generation",
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
    json={
        "model": "MiniMax-H3",
        "content": [{"type": "text", "text": "A red fox trots across fresh snow at dawn"}],
        "duration": 6,
        "ratio": "16:9",
    },
)
response.raise_for_status()

print(response.json()["task_id"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "task_id": "9f1c2b7e4d6a4f0b8e3c5a7d9b1f2e4c",
  "status": "queued",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```
