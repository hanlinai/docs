---
description: Reference for GET /api/v1/balance, which returns your credit balance.
---

# Get balance

This page is the reference for the balance endpoint of the Management API. It is intended for developers who monitor their credit balance.

## Endpoint

```http
GET https://engy.ai/api/v1/balance
```

## Authentication

Send your account token in the `Authorization` header as a bearer token. For details, see [Management API overview](overview.md#authentication).

## Parameters

This endpoint takes no parameters.

## Response

### Response fields

| Field | Type | Description |
| --- | --- | --- |
| `data.balance_micro` | Integer | The total balance, in micro-USD. |
| `data.balance_usd` | String | The total balance, in USD, with two decimal places. |
| `data.available_micro` | Integer | The balance minus holds for requests in progress, in micro-USD. |
| `data.available_usd` | String | The balance minus holds for requests in progress, in USD, with two decimal places. |
| `meta.currency` | String | Always `USD`. |
| `meta.unit` | String | Always `micro`. |

## Errors

This endpoint returns the common errors only. For details, see [Management API overview](overview.md#errors).

## Example

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://engy.ai/api/v1/balance \
  -H "Authorization: Bearer $ENGY_ACCOUNT_TOKEN"
```
{% endtab %}

{% tab title="Python" %}
```python
import os

import httpx

response = httpx.get(
    "https://engy.ai/api/v1/balance",
    headers={"Authorization": f"Bearer {os.environ['ENGY_ACCOUNT_TOKEN']}"},
)
response.raise_for_status()

print(response.json()["data"]["available_usd"])
```
{% endtab %}
{% endtabs %}

Response:

```json
{
  "data": {
    "available_micro": 24500000,
    "available_usd": "24.50",
    "balance_micro": 25000000,
    "balance_usd": "25.00"
  },
  "meta": {
    "currency": "USD",
    "unit": "micro"
  }
}
```
