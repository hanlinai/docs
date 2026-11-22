---
description: Monitor your workers, epoch performance, and rewards in the provider portal, by email, and in the network explorer.
---

# Monitor workers

This page explains how to monitor the health and earnings of your workers. It is intended for providers who operate live workers.

## Provider portal

| Page | Content |
| --- | --- |
| **Dashboard** | Workers online and active, your traffic ID, current-epoch requests, successes, timeouts, and failures per model, and a table of live workers |
| **Workers** | The **Servers** table with server, IP address, model, state, requests, success rate, latency, tokens, and registration time |
| **Rewards** | One row per epoch that earned emission, with requests, success rate, settled value, score, and emission. Score and emission are blank until the epoch closes. |
| **Gateway key** | Your gateway key status and key rotation |
| **Notifications** | Email notification settings |

## Email notifications

Engy emails you when a worker starts or stops serving, that is, when it enters or leaves the `active` state. Engy also emails you when a verified or live server fails its health check.

1. In the portal, go to **Dashboard** > **Notifications**.
2. Enter an email address, and click **add & send code**.
3. Enter the 6-digit code from the email, and click **confirm**. The code expires after 15 minutes.

You can add up to three addresses. To pause all emails, clear **send me notifications**.

Engy sends at most one email every 15 minutes. Each email summarizes the net change for each worker.

## Logs on the server

| Log | Content |
| --- | --- |
| `/var/log/engy/miner.log` | The connection to the Engy network. `ADMITTED` means that the worker connected. |
| `/var/log/engy/serve.log` | The inference engine |

## Network explorer

You can view network data without signing in at [provider.engy.ai](https://provider.engy.ai):

| Page | Content |
| --- | --- |
| [Epochs](https://provider.engy.ai/epochs) | The leaderboard of each epoch, including scores and failed gates |
| [Miners](https://provider.engy.ai/miners) | Requests, success rate, timeouts, and latency per registered hotkey in the current epoch, with daily totals |

## What to watch

* **Acceptance ratio**: Keep it above the model's minimum, which the epoch page shows. Timeouts and worker errors lower it.
* **Latency**: Keep the 99th percentile time per output token below the model's target, if one applies.
* **Connection**: A worker receives traffic only while it is connected. It resumes when it reconnects.

For the scoring rules, see [Subnet 53](../overview/subnet-53.md#how-engy-scores-your-traffic).
