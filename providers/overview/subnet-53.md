---
description: How your provider account relates to Bittensor subnet 53, how to register your hotkey, and how Engy turns your work into rewards.
---

# Subnet 53

This page explains what you need to know about Bittensor subnet 53 (SN53) as an Engy provider. It covers hotkey registration, how Engy scores your work, and how scores become rewards. It is intended for providers who are onboarding or who want to understand their rewards.

## Register your hotkey

Your provider account is identified by a Bittensor hotkey that is registered on SN53. Registration follows the standard Bittensor process and is permissionless. Use `btcli` to register your hotkey:

```bash
btcli subnet register --netuid 53 --wallet.name <wallet> --wallet.hotkey <hotkey>
```

For details about wallets, registration costs, and `btcli`, see the [Bittensor documentation](https://docs.learnbittensor.org).

You bind the hotkey to your provider account when you sign in to the provider portal and claim your gateway key. For details, see [Create your provider account](../onboarding/create-your-provider-account.md). Each hotkey can be bound to one provider account only.

Engy counts the requests that your workers serve from the time of binding. Requests served before the binding do not count toward your score.

## How emission is split

Engy measures your work in epochs. Each epoch lasts 7 days and starts on Wednesday at 08:00 UTC. Each epoch is scored independently.

Each epoch's emission is split into budgets, and each budget is shared only among the providers that earn from it:

| Budget | Share of emission | How it is shared |
| --- | --- | --- |
| Baseline pool, per hardware class | 80% in total (today 8x B300 70%, 8x B200 30% of the pool) | By each enrolled server's uptime in that class |
| Traffic, per model | 20% in total | One sub-pool for each of the five models with the most buyer spend over the 30 days before the epoch, and one shared sub-pool for all other models. Each sub-pool's share is its share of that spend. |

* The shares are fixed when the epoch opens and do not change during the epoch.
* A sub-pool that nobody earns from in an epoch is burned. It is not moved to another model or class.
* A model added during an epoch is scored in the shared sub-pool for other models.

The [Allocation](https://provider.engy.ai/allocation) page in the network explorer shows the split of each epoch, the spend that each traffic share came from, and how much of each sub-pool was paid or burned.

## How Engy scores your traffic

For each model that you serve in an epoch:

1. Engy measures your work: the value of the tokens that your workers served in successful paid requests. Input, cached input, and output tokens are valued separately at the model's list prices, not at the price that a particular user paid.
2. Engy applies the quality gates. If you fail a gate, your score for that model is zero for the epoch.
3. Your traffic weight for the model is your share of the scores in that model's sub-pool.

Test traffic does not count toward your score.

### Quality gates

| Gate | Passes when |
| --- | --- |
| Acceptance ratio | At least 95% of your requests for the model succeed. Requests that the user cancels are excluded. |
| Time to first token | The 99th percentile time to first token is at most the model's target, if the model has one. |
| Time per output token | The 99th percentile time per output token is at most the model's target, if the model has one. |

Each gate applies only after it has at least 1,000 requests to evaluate for the model in the epoch. Requests that Engy rejects before they reach a worker are not counted. These values can change. The page of each epoch in the network explorer shows the values that apply.

### Capacity confidence

Engy can apply a confidence factor to your account if your workers do not sustain the capacity that they declare. The factor discounts all of your work in that epoch, and it resets at the start of the next epoch.

## How you earn rewards

When an epoch closes, Engy converts the results into weights on SN53. Your weight is the sum of what you earn from each baseline class and each traffic sub-pool. Validators set the weights on chain, and you earn SN53 emissions in proportion to your weight.

Scores are provisional while the epoch is open. The result becomes final about 1 hour after the epoch ends.

## Track your performance

| Where | Content |
| --- | --- |
| **Dashboard** in the provider portal | Requests, successes, timeouts, and failures per model for the current epoch |
| **Rewards** in the provider portal | One row per epoch with requests, success rate, score, and emission, split by baseline class and traffic sub-pool |
| [Allocation](https://provider.engy.ai/allocation) in the network explorer | How each epoch's emission is split, and how much of each pool was paid or burned |
| [Epochs](https://provider.engy.ai/epochs) in the network explorer | The leaderboard of each epoch. Hover over a score to see whether a gate failed. |
| [Miners](https://provider.engy.ai/miners) in the network explorer | Traffic and latency per hotkey in the current epoch |

## Deregistration

If your hotkey is deregistered from SN53, the following happens:

* **Scoring**: Your work is not scored.
* **Traffic**: Your workers are not assigned traffic until you register the hotkey again.
* **Workers**: If your hotkey is missing from SN53 for about 15 minutes and your workers are not serving traffic, Engy revokes your workers.

To continue serving, register the hotkey again. If Engy revoked your workers, switch each server to another model, or contact Engy support.
