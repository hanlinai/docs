---
description: Reference for GET /v2/query/video_generation, which lists your video tasks from the last 7 days.
---

# List video tasks

This page is the reference for listing video generation tasks. It is intended for developers who track several tasks at once.

## Endpoint

```http
GET https://api.engy.ai/v2/query/video_generation
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Query parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `page_num` | Integer | No | The page number, starting at `1`. Defaults to `1`. |
| `page_size` | Integer | No | The number of tasks per page, from 1 to 100. Defaults to `10`. |
| `status` | String | No | Returns only tasks with this status: `queued`, `running`, `succeeded`, `failed`, or `cancelled`. |
| `model` | String | No | Returns only tasks for this model. |
| `task_ids` | String | No | Returns only the tasks with these IDs, separated by commas. At most 100 IDs are used. |

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `items` | Array | The tasks, newest first. Each task has the fields described in [Get a video task](get-video-task.md#response-fields). |
| `total` | Integer | The total number of matching tasks. |
| `page_num` | Integer | The current page number. |
| `page_size` | Integer | The page size. |
| `base_resp` | Object | The status object. `status_code` is `0` on success. |

Only tasks created in the last 7 days are listed.

## Errors

| Status | Cause |
| --- | --- |
| `400` | The `status` value is invalid. |
| `401` | The API key is missing or invalid. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl "https://api.engy.ai/v2/query/video_generation?status=succeeded&page_size=20" \
  -H "Authorization: Bearer $ENGY_API_KEY"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.get(
    "https://api.engy.ai/v2/query/video_generation",
    params={"status": "succeeded", "page_size": 20},
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
)
response.raise_for_status()

for task in response.json()["items"]:
    print(task["task_id"], task["status"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "items": [
    {
      "task_id": "9f1c2b7e4d6a4f0b8e3c5a7d9b1f2e4c",
      "model": "MiniMax-H3",
      "status": "succeeded",
      "duration": 6,
      "usage": {"video_seconds": 6.0}
    }
  ],
  "total": 1,
  "page_num": 1,
  "page_size": 20,
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```

The task objects in this example are shortened.
