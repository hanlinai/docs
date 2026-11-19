---
description: Reference for GET /api/v1/usage, which returns your requests and spending by day and by model.
---

# Get usage

This page is the reference for the usage endpoint of the Management API. It is intended for developers who track spending or build cost reports.

## Endpoint

```http
GET https://engy.ai/api/v1/usage
```

## Authentication

Send your account token in the `Authorization` header as a bearer token. For details, see [Management API overview](overview.md#authentication).

## Parameters

### Query parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `days` | Integer | No | The window in days. Accepted values are `7` and `30`. Defaults to `30`. |
| `model` | String | No | Returns usage for this model ID only. An unknown model ID returns empty results, not an error. |

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `data.total` | Object | The totals for the whole window. |
| `data.by_day` | Array | The totals per UTC day, in ascending order. |
| `data.by_day[].day_start` | Integer | The start of the day, as a Unix timestamp in seconds. |
| `data.by_model` | Array | The totals per model, sorted by cost in descending order. |
| `data.by_model[].model` | String | The model ID. |
| `requests` | Integer | The number of requests. Appears in each total. |
| `cost_micro` | Integer | The amount spent, in micro-USD. Appears in each total. |
| `cost_usd` | String | The amount spent, in USD, with two decimal places. Appears in each total. |
| `meta.days` | Integer | The applied window. |
| `meta.model` | String or null | The applied model filter, or `null`. |

Usage covers complete UTC days only, up to and including the previous day.

## Errors

| Code | Status | Cause |
| --- | --- | --- |
| `invalid_request` | `400` | The value of `days` is not `7` or `30`. |

For the common errors, see [Management API overview](overview.md#errors).

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl "https://engy.ai/api/v1/usage?days=7&model=glm-5.2" \
  -H "Authorization: Bearer $ENGY_ACCOUNT_TOKEN"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.get(
    "https://engy.ai/api/v1/usage",
    params={"days": 7, "model": "glm-5.2"},
    headers={"Authorization": f"Bearer {os.environ['ENGY_ACCOUNT_TOKEN']}"},
)
response.raise_for_status()

print(response.json()["data"]["total"]["cost_usd"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "data": {
    "total": {"requests": 1200, "cost_micro": 3400000, "cost_usd": "3.40"},
    "by_day": [
      {"day_start": 1792281600, "requests": 180, "cost_micro": 510000, "cost_usd": "0.51"}
    ],
    "by_model": [
      {"model": "glm-5.2", "requests": 1200, "cost_micro": 3400000, "cost_usd": "3.40"}
    ]
  },
  "meta": {
    "days": 7,
    "model": "glm-5.2"
  }
}
```
