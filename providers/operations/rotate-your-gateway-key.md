---
description: Replace your gateway key if it is exposed, and reconnect your workers with the new key.
---

# Rotate your gateway key

This page explains how to replace your gateway key. It is intended for providers whose key might be exposed.

## Before you rotate

Rotating the key has the following effects:

* Your traffic ID, which is shown on the **Dashboard**, changes. Your rewards history and scoring stay with your account.
* Every worker must reconnect with the new key. Workers that still use the old key stop receiving traffic.
* Servers that boot with the old key cannot register.

## Rotate the key

{% stepper %}
{% step %}
### Create the new key

1. In the portal, go to **Dashboard** > **Gateway key**.
2. Click **rotate key**, read the warning, and click **rotate now**.
3. Copy the new key. It is displayed only once.
{% endstep %}

{% step %}
### Update your workers

On each worker, set `ENGY_MINER_KEY` to the new key in the container configuration, and then restart the container.
{% endstep %}

{% step %}
### Confirm the reconnection

Check that each server returns to a live state in the **Servers** table. If a server fails verification, use **Update endpoint** or **retry**. For details, see [Manage workers](manage-workers.md).
{% endstep %}
{% endstepper %}
