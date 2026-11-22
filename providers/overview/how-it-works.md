---
description: How requests reach your workers, how Engy identifies you, and how your work turns into rewards.
---

# How it works

This page explains how the Engy network routes requests to providers and how providers earn. It is intended for prospective providers who want to understand the network before they onboard.

## Request flow

1. An Engy user sends an inference request to the Engy API.
2. Engy selects an available worker that serves the requested model.
3. Engy forwards the request to the worker over a connection that the worker opened.
4. The worker runs the model and streams the response back through Engy to the user.

Your workers connect outbound to Engy. You do not need to open an inbound port for request traffic.

## Accounts, keys, and workers

| Item | Description |
| --- | --- |
| Hotkey | Your Bittensor hotkey, registered on SN53. It identifies your provider account and receives your weight. |
| Gateway key | A secret key that starts with `mk-`. Your workers use it to authenticate to Engy. You claim it in the provider portal. |
| Provider ID | A public pseudonym in the form `mnr-` followed by 12 hexadecimal characters. It identifies your traffic in public data without revealing your key. It is shown as the traffic ID on the **Dashboard**, and it changes when you rotate your gateway key. |
| Server | A GPU machine that you register. In the baseline pool, the server is the unit that earns. |
| Worker | One model deployment on a server. Engy assigns the model, and it can switch a pool server's model as demand changes. One provider account can run many servers and workers. |

## How you earn

Each 7-day epoch's emission is split into two budgets:

| Budget | Share | Earned by |
| --- | --- | --- |
| Baseline pool | 80% | Servers in the current hardware classes (today 8x B300 and 8x B200) for staying online and serving the model that Engy assigns |
| Traffic | 20% | Every provider, for paid buyer traffic that passes the quality gates |

* **Baseline pool.** The pool is split by hardware class. Engy samples each enrolled server every 5 minutes; a server is up when its hardware still matches its class and one of its workers is active and passes a short liveness check. Your share of a class is your servers' uptime against the other servers in that class. Uptime is paid after 5 days without an outage, back pay included. You can schedule maintenance windows so that a planned drain does not count as an outage.
* **Traffic.** The traffic budget is split into a sub-pool for each of the five models with the most buyer spend over the last 30 days, plus one shared sub-pool for all other models. The shares are fixed when the epoch opens. Inside a model's sub-pool, providers split it by the work that they served, valued at the model's list prices.

When the epoch closes, Engy converts the results into weights and publishes a signed result. Validators set the weights on chain, and you earn SN53 emissions in proportion to your weight. The current split is on the [Allocation](https://provider.engy.ai/allocation) page.

For details, see [Subnet 53](subnet-53.md).

## What Engy manages

Engy deploys and verifies serving on your workers through a management SSH channel in the Engy image. On baseline-pool servers, Engy also assigns and switches the model, and sends a short liveness check every 5 minutes. You keep ownership and control of your hardware.
