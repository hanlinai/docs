---
description: Retry, update, switch, delete, and revoke your workers in the provider portal.
---

# Manage workers

This page explains the actions that you can take on your servers and workers in the provider portal. It is intended for providers who operate live workers.

All actions are available in the portal under **Dashboard** > **Workers**, either on the **Servers** page or on a server's page.

## Retry onboarding

Engy does not retry a failed onboarding automatically. To restart it with the same endpoint, click **retry** on the server. The server returns to the `declared` state.

**retry** is available while a server is onboarding or has failed. It is not available after the worker is live.

## Update the endpoint

If onboarding failed because of a wrong model or SSH endpoint, the server page shows the **Update endpoint** form:

1. Open the server.
2. In the **Update endpoint** form, correct the values. The form is pre-filled with the current values.
3. Click **update & re-drive**. Engy restarts verification. If you choose a different model, Engy creates a new worker on the server.

## Switch the model

You can switch a server to another model at any time, except while Engy is verifying it. No reboot is needed.

1. Open the server.
2. In the **Switch model** form, select a different model.
3. Check the SSH host and port, which are pre-filled.
4. Click **switch model**.

Engy retires the current worker, which moves to `superseded`, and creates a new worker with a new worker ID. The new worker is onboarded from the `declared` state.

{% hint style="warning" %}
Certifications, such as zero data retention and region, do not carry over to the new worker.
{% endhint %}

## Delete a server

To remove a server from your list, click **delete**, and then confirm. The server is removed from the list.

You can delete a server at any stage before its worker is live. You cannot delete a server whose worker has been provisioned.

If you boot a deleted server again, it registers again as the same server, in the `booted` state. Register the endpoint again.

## Revoke workers

Revoking a worker takes it out of service immediately.

* **One worker**: Click **revoke** on the server.
* **All workers**: Click **revoke all workers**, and then click **Revoke all**. Every pending, onboarding, or active worker of your account is revoked.

You cannot undo a revocation in the portal. To serve from the server again, switch it to another model, or contact Engy support.

While your account is disabled by Engy, you cannot revoke workers.

## View the provisioning timeline

A server's page shows the **Provisioning timeline**: each state change, its time, who made it, and the reason.
