---
description: The onboarding states of a server and the routing states of a live worker, and what each state means.
---

# Worker states

This page explains the states that a server and its worker go through. It is intended for providers who monitor their workers in the portal.

## Onboarding states

The **state** column of the **Servers** table shows the onboarding state until the worker is live:

| State | Description | Your action |
| --- | --- | --- |
| `booted` | The server registered itself after booting. | Register the endpoint. |
| `declared` | The model and SSH endpoint are registered. The server is queued for verification. | None |
| `reachable` | Engy has picked up the server for verification and is connecting to it over SSH. | None |
| `verifying` | Engy is verifying the server. | None |
| `verified` | The server passed verification and is queued for deployment. | None |
| `starting` | Engy is downloading the model and starting the inference engine. | None |
| `awaiting_verification`, `awaiting_review` | Engy is reviewing the worker. | None |
| `provisioned` | Serving is up. The column now shows the worker's routing state. | None |
| `failed` | Onboarding failed, or Engy lost SSH access to the server after it was verified. The server page shows the reason. | Correct the problem, and then click **retry** or use **Update endpoint**. |
| `superseded` | You chose another model for the server, and a new worker replaced this one. | None |
| `deleted` | The worker was deleted. | None |

Deleted servers are removed from the list. When you switch a server to another model, Engy retires the previous worker, and the server shows the new worker.

### Health checks

After a server is verified, Engy checks SSH access to it every 10 minutes. If 3 consecutive checks fail, the server moves to `failed` with the reason `health check: unreachable after 3 consecutive probes`.

## Routing states

After a worker is provisioned, its routing state determines whether it receives traffic:

| State | Receives traffic | Description |
| --- | --- | --- |
| `onboarding` | No | Serving is up. Engy activates the worker when it connects to the Engy network. |
| `active` | Yes | The worker is serving user requests. |
| `degraded` | No | The worker failed the quality checks on recent requests. Engy tests it again after a waiting period and restores it when it passes. |
| `draining` | No | Engy took the worker out of routing for maintenance. Requests in progress finish, and the connection stays open. Only Engy can return the worker to `active`. |
| `failed` | No | The worker failed a quality check. Switch the server to another model, or contact Engy support. |
| `revoked` | No | The worker was taken down. |

### Connection

The connection is shown separately from the state. A worker that disconnects keeps its state but receives no traffic. When it reconnects, it receives traffic again without a new test. The portal marks a worker as offline after about two minutes without a connection.

### Degraded workers

Engy checks the success rate and 99th percentile latency of live workers on consecutive blocks of 100 real requests. Errors caused by the user's request are excluded. If a worker fails three consecutive blocks, it moves to `degraded`.

After a 10-minute hold, Engy sends a test request. If the worker passes, it returns to `active`. If it fails, Engy tests it again at an increasing interval, up to about four hours.

### Revoked workers

A worker is revoked in the following cases:

* You revoke it in the portal.
* You switch the server to another model.
* Your hotkey has been missing from SN53 for about 15 minutes, and none of your workers is serving traffic.
* Your provider account is suspended.

## Notifications

To receive an email when a worker starts or stops serving, or when a verified server fails its health check, see [Monitor workers](monitor-workers.md#email-notifications).
