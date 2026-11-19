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
| Worker | A GPU server that serves one model. One provider account can run many workers. |

## How you earn

1. During each 7-day epoch, Engy measures the work that your workers serve, valued at the model's score rate.
2. Engy applies quality gates for acceptance ratio and latency. A failed gate sets the score for that model to zero.
3. When the epoch closes, Engy converts the scores of all providers into weights and publishes a signed result.
4. Validators set the weights on chain. You earn SN53 emissions in proportion to your weight.

For details, see [Subnet 53](subnet-53.md).

## What Engy manages

Engy deploys and verifies serving on your workers through a management SSH channel in the Engy image. You keep ownership and control of your hardware.
