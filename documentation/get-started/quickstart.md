---
description: Create an account, get an API key, and send your first request to Engy.
---

# Quickstart

This guide takes you from a new account to a successful inference request. It is intended for developers who are using Engy for the first time.

## Prerequisites

Before you begin, make sure you have:

* An email address, or a Google or GitHub account
* A payment method for adding credits
* `curl`, or Python 3.8 or later with the `openai` package installed

## Send your first request

{% stepper %}
{% step %}
### Create an account

Go to [engy.ai/signup](https://engy.ai/signup) and sign up with your email address, Google account, or GitHub account.

If you sign up with an email address, open the verification email and confirm your address before you sign in.
{% endstep %}

{% step %}
### Add credits

In the dashboard, go to **Credits** and click **Add credits**. Choose an amount and a payment method, then complete the payment.

Engy uses prepaid credits. Each request is charged to your credit balance.
{% endstep %}

{% step %}
### Create an API key

On the dashboard overview page, enter an optional label in the **API keys** section, and then click **Create key**.

Copy the key and store it securely. API keys start with `sk-engy-`.

{% hint style="warning" %}
The key is displayed only once. If you lose it, delete it and create a new key.
{% endhint %}

Set the key as an environment variable:

```bash
export ENGY_API_KEY="sk-engy-..."
```
{% endstep %}

{% step %}
### Send a request

Send a chat completion request to the Engy API:

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/chat/completions \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5.2",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import os

from openai import OpenAI

client = OpenAI(
    base_url="https://api.engy.ai/v1",
    api_key=os.environ["ENGY_API_KEY"],
)

response = client.chat.completions.create(
    model="glm-5.2",
    messages=[{"role": "user", "content": "Hello"}],
)

print(response.choices[0].message.content)
```
{% endtab %}
{% endtabs %}
{% endstep %}
{% endstepper %}

## Understand the response

The response follows the OpenAI chat completion format. Engy adds the following information:

| Field or header | Description |
| --- | --- |
| `x_engy.request_id` | The unique identifier of the request. Include it when you contact support. |
| `x_engy.charged_micro` | The amount charged for the request, in micro-USD. One USD equals 1,000,000 micro-USD. |
| `X-Engy-Request-Id` header | The same request identifier, returned as an HTTP header. |
| `X-Engy-Charged-Micro` header | The same charged amount, returned as an HTTP header. |

## Next steps

* Review your requests and spending on the **Usage** and **Logs** pages of the [dashboard](https://engy.ai/dashboard).
* Use any other model by changing the `model` parameter. For a list of available models, call `GET https://api.engy.ai/v1/models`.
