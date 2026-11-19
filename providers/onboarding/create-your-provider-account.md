---
description: Sign in to the provider portal with your SN53 hotkey and claim the gateway key that your workers use.
---

# Create your provider account

This page explains how to create your Engy provider account and claim your gateway key. It is intended for providers who are onboarding for the first time.

## Prerequisites

Before you begin, make sure you have:

* A hotkey registered on SN53. For details, see [Register your hotkey](../overview/subnet-53.md#register-your-hotkey)
* The hotkey in a browser wallet extension: polkadot{.js}, Talisman, or SubWallet

## Create the account

{% stepper %}
{% step %}
### Sign in with your hotkey

1. Go to [provider.engy.ai/login](https://provider.engy.ai/login).
2. Click **connect wallet**, and allow the extension to connect.
3. Under **pick your hotkey**, click **sign in** next to the hotkey that you registered on SN53.
4. Approve the signature request in the extension.

Signing a one-time challenge proves that you control the hotkey. It is not a transaction and costs no fees.

Your session lasts 7 days.
{% endstep %}

{% step %}
### Claim your gateway key

1. In the portal, go to **Dashboard** > **Gateway key**.
2. Click **claim gateway key**.
3. Copy the key. It starts with `mk-`.

{% hint style="warning" %}
The key is displayed only once. Engy stores only a hash of it. Store the key in a secure location, such as a secrets manager.
{% endhint %}

Claiming the key binds your hotkey to your provider account. Each hotkey can have one provider account.

If you lose the key, you can replace it. For details, see [Rotate your gateway key](../operations/rotate-your-gateway-key.md).
{% endstep %}
{% endstepper %}

## Next steps

* [Boot a worker](boot-a-worker.md) with your gateway key.
* Optionally, [apply as an approved provider](apply-as-an-approved-provider.md).
