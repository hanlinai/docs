---
description: Create, store, and delete the API keys that authenticate your inference requests.
---

# API keys

This page explains how to manage the API keys that authenticate requests to the Engy inference API. It is intended for developers who integrate Engy into applications.

## About API keys

An API key authorizes inference requests and charges their cost to your credit balance. API keys start with `sk-engy-`.

Send the key in the `Authorization` header of each request:

```bash
curl https://api.engy.ai/v1/chat/completions \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "glm-5.2", "messages": [{"role": "user", "content": "Hello"}]}'
```

{% hint style="info" %}
API keys are separate from account tokens. An account token gives read-only access to the Management API and cannot authorize inference. For details, see [Management API](management-api.md).
{% endhint %}

## Create an API key

{% stepper %}
{% step %}
### Open the dashboard

Sign in and open the **Dashboard** overview page.
{% endstep %}

{% step %}
### Create the key

In the **API keys** section, enter an optional label, and then click **Create key**.

Use labels to identify where each key is used, such as `production` or `ci`.
{% endstep %}

{% step %}
### Copy the key

Copy the key and store it in a secure location, such as a secrets manager or an environment variable.

{% hint style="warning" %}
The key is displayed only once. Engy stores only a hash of the key and cannot show it again.
{% endhint %}
{% endstep %}
{% endstepper %}

## Delete an API key

In the **API keys** section of the dashboard, click **Delete** next to the key, and then confirm.

{% hint style="danger" %}
Deleting a key takes effect immediately. Any application that uses the key stops working, and you cannot undo the action.
{% endhint %}

## Best practices

* Create a separate key for each application or environment, so that you can delete one key without affecting the others.
* Store keys in environment variables or a secrets manager. Do not commit keys to source control.
* If a key is exposed, delete it and create a new key.
