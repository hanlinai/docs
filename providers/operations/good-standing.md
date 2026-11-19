---
description: The rules that keep your provider account in good standing, and the consequences of breaking them.
---

# Good standing

This page explains the rules that providers must follow and what happens when a provider breaks them. It is intended for all providers.

## Rules

* **Serve the published checkpoint**: Serve each model from its fixed checkpoint. Do not serve a different, smaller, or modified model under a model's ID. For the checkpoints, see [Supported models](../overview/supported-models.md).
* **Declare honest capacity**: Declare only the concurrency that your workers can sustain.
* **Return real errors**: If a request fails, return an error. Do not return a successful response that contains an error message.
* **Stay connected**: Keep your workers connected. During planned network maintenance, Engy asks workers to reconnect. Finish the requests in progress, and then reconnect.
* **Keep your hotkey registered**: Keep your hotkey registered on SN53.

## Consequences

| Situation | Consequence |
| --- | --- |
| A worker fails the quality checks on recent requests | The worker moves to `degraded` and stops receiving traffic until it passes a new test. |
| A worker fails a quality check after it goes live | The worker moves to `failed`. |
| Your workers do not sustain the declared capacity | A confidence factor discounts your work for the epoch. |
| Your acceptance ratio or latency fails a gate | Your score for that model is zero for the epoch. |
| Your hotkey is deregistered | Your work is not scored, and your workers can be revoked. |
| Your account is suspended | Your workers receive no traffic, and you cannot onboard new workers. |

The **Dashboard** shows a notice if your hotkey is not registered on SN53, if a penalty is active on your account, or if Engy has disabled your account. While your account is disabled, you cannot revoke or onboard workers. To appeal, contact the subnet operators.

For details about scoring, see [Subnet 53](../overview/subnet-53.md#how-engy-scores-your-work).
