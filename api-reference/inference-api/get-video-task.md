---
description: Reference for GET /v2/query/video_generation/{task_id}, which returns the status of a video task.
---

# Get a video task

This page is the reference for retrieving a video generation task. It is intended for developers who poll tasks until they finish.

## Endpoint

```http
GET https://api.engy.ai/v2/query/video_generation/{task_id}
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Path parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `task_id` | String | Yes | The task ID. |

## Response

### Response fields

The response contains a `task` object and a `base_resp` object. The `task` object has the following fields:

| Field | Type | Description |
| --- | --- | --- |
| `task_id` | String | The task ID. |
| `model` | String | The video model ID. |
| `status` | String | `queued`, `running`, `succeeded`, `failed`, or `cancelled`. |
| `resolution` | String | The resolution, such as `768P`. |
| `duration` | Integer | The requested duration, in seconds. |
| `ratio` | String | The aspect ratio. |
| `created_at` | Number | The creation time, as a Unix timestamp in seconds. |
| `updated_at` | Number | The time of the last status change, as a Unix timestamp in seconds. |
| `finished_at` | Number or null | The completion time, as a Unix timestamp in seconds. |
| `attempts` | Integer | The number of times the task was started. |
| `usage.video_seconds` | Number | The seconds of video produced. You are charged for this value. |
| `content.url` | String | A signed download link, valid for 24 hours. Present only if `status` is `succeeded`. |
| `content.content_type` | String | Always `video/mp4`. |
| `content.bytes` | Integer | The file size, in bytes. |
| `content.expires_at` | Integer | The expiry time of the link, as a Unix timestamp in seconds. |
| `error` | Object or null | The error, with `message` and `type`, if `status` is `failed`. |

## Errors

| Status | Cause |
| --- | --- |
| `401` | The API key is missing or invalid. |
| `404` | The task does not exist or belongs to another account. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v2/query/video_generation/$TASK_ID \
  -H "Authorization: Bearer $ENGY_API_KEY"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

task_id = os.environ["TASK_ID"]
response = httpx.get(
    f"https://api.engy.ai/v2/query/video_generation/{task_id}",
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
)
response.raise_for_status()

print(response.json()["task"]["status"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "task": {
    "task_id": "9f1c2b7e4d6a4f0b8e3c5a7d9b1f2e4c",
    "model": "MiniMax-H3",
    "status": "succeeded",
    "resolution": "768P",
    "duration": 6,
    "ratio": "16:9",
    "created_at": 1792281600,
    "updated_at": 1792281780,
    "finished_at": 1792281780,
    "attempts": 1,
    "usage": {"video_seconds": 6.0},
    "content": {
      "url": "https://api.engy.ai/v2/video_generation/9f1c2b7e4d6a4f0b8e3c5a7d9b1f2e4c/content?token=...",
      "content_type": "video/mp4",
      "bytes": 4194304,
      "expires_at": 1792368180
    },
    "error": null
  },
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```
