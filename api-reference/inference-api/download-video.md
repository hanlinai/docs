---
description: Reference for GET /v2/video_generation/{task_id}/content, which downloads the video of a finished task.
---

# Download a video

This page is the reference for downloading the video of a finished task. It is intended for developers who save generated clips.

## Endpoint

```http
GET https://api.engy.ai/v2/video_generation/{task_id}/content
```

## Authentication

Use one of the following methods:

* Send the signed link from `task.content.url`, which includes a `token` query parameter. The link does not require an API key and is valid for 24 hours.
* Send your API key in the `Authorization` header as a bearer token. This method works at any time until you delete the task.

## Parameters

### Path parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `task_id` | String | Yes | The task ID. |

### Query parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | String | No | The signature from the signed link. Not required if you send your API key. |

## Response

The endpoint returns status `302` with a redirect to a short-lived storage link for the MP4 file. Configure your client to follow redirects.

## Errors

| Status | Cause |
| --- | --- |
| `403` | The link is invalid or expired, and no valid API key for the task's account was sent. |
| `404` | The task does not exist, has not succeeded, or has no video. |

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl -L -o clip.mp4 https://api.engy.ai/v2/video_generation/$TASK_ID/content \
  -H "Authorization: Bearer $ENGY_API_KEY"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

task_id = os.environ["TASK_ID"]
response = httpx.get(
    f"https://api.engy.ai/v2/video_generation/{task_id}/content",
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
    follow_redirects=True,
)
response.raise_for_status()

with open("clip.mp4", "wb") as f:
    f.write(response.content)
```
{% endtab %}
{% endtabs %}
