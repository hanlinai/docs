---
description: How Engy verifies a registered server, brings serving up, and activates the worker for user traffic.
---

# Verification and activation

This page explains what Engy does after you register a worker's endpoint, and how to confirm that the worker is live. It is intended for providers who are waiting for a worker to go live.

## Verification

After you register the endpoint, Engy verifies the server automatically. The server moves through the following states:

| State | Description |
| --- | --- |
| `declared` | The server is queued for verification. |
| `reachable` | Engy has reached the server over SSH. |
| `verifying` | Engy is verifying the server. |
| `verified` | The server passed verification and is queued for deployment. |
| `starting` | Engy is downloading the model and starting the inference engine. For large models, this stage can take a long time. |

Engy checks the following:

* Engy can reach the server over the SSH port that you registered.
* The gateway key on the server matches the key that you claimed.
* The CPU TEE evidence (Intel TDX or AMD SEV-SNP) is valid, and the hardware meets the GPU class and count that the selected model requires. Servers with 8 B200 or 8 B300 GPUs need no CPU TEE. GPU confidential-computing mode is not required. For details, see [TEE workers overview](../tee-workers/overview.md).

Engy takes the result from the verified evidence, not from the server's own claim.

## Activation

After verification, Engy deploys the inference engine for the model that you selected. When serving is up, onboarding is complete, and the portal shows the worker as `onboarding`. When the worker connects to the Engy network, Engy activates it automatically. Its state changes to `active`, and it starts to receive user traffic.

## Confirm that the worker is live

* **Portal**: The server shows the state of the live worker, such as `active`, in the **Servers** table.
* **Logs**: On the server, `/var/log/engy/miner.log` shows `ADMITTED` when the worker connects. The inference engine logs to `/var/log/engy/serve.log`.

## If verification fails

The server moves to the `failed` state, and the server page shows the reason. Common causes are:

* The SSH port is not reachable from the internet.
* The gateway key on the server does not match your claimed key.
* The server has no valid CPU TEE (Intel TDX or AMD SEV-SNP) and is not an 8 B200 or 8 B300 server, or the evidence is invalid, or the hardware does not meet the model's requirements.

Engy does not retry a failed verification automatically. Correct the problem, and then click **retry** or **update & re-drive**. For details, see [Manage workers](../operations/manage-workers.md).
