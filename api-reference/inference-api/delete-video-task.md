---
description: Reference for DELETE /v2/video_generation/{task_id}, which cancels a queued video task or deletes a finished one.
---

# Delete a video task

This page is the reference for cancelling or deleting a video generation task. It is intended for developers who manage their video tasks and stored files.

## Endpoint

```http
DELETE https://api.engy.ai/v2/video_generation/{task_id}
```

## Authentication

Send your API key in the `Authorization` header as a bearer token. For details, see [Inference API overview](overview.md#authentication).

## Parameters

### Path parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `task_id` | String | Yes | The task ID. |

The result depends on the status of the task:

| Task status | Result |
| --- | --- |
| `queued` | The task is cancelled. The reserved credit is released, and you are not charged. |
| `succeeded` or `failed` | The task and its video file are deleted. |
| `running` | The request fails. Wait for the task to finish. |

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `task_id` | String | The task ID. |
| `action` | String | `cancelled` or `deleted`. |
| `status` | String | The status of the task: `cancelled`, `succeeded`, or `failed`. |
| `base_resp` | Object | The status object. `status_code` is `0` on success. |

## Errors

| Status | Cause |
| --- | --- |
| `400` | The task is running and cannot be cancelled or deleted. |
| `401` | The API key is missing or invalid. |
| `404` | The task does not exist or belongs to another account. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl -X DELETE https://api.engy.ai/v2/video_generation/$TASK_ID \
  -H "Authorization: Bearer $ENGY_API_KEY"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

task_id = os.environ["TASK_ID"]
response = httpx.delete(
    f"https://api.engy.ai/v2/video_generation/{task_id}",
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
)
response.raise_for_status()

print(response.json()["action"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "task_id": "9f1c2b7e4d6a4f0b8e3c5a7d9b1f2e4c",
  "action": "cancelled",
  "status": "cancelled",
  "base_resp": {"status_code": 0, "status_msg": "success"}
}
```
