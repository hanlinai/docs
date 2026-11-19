---
description: Supply GPU capacity to the Engy network on Bittensor subnet 53, serve paid inference, and earn emissions.
---

# Introduction

Engy providers supply the GPU capacity that serves inference requests from Engy users. This documentation explains how to become a provider, bring workers online, and operate them. It is intended for GPU operators who want to join the Engy network.

A provider is a miner on Bittensor subnet 53 (SN53). Throughout this documentation, the term "provider" is used.

## What providers do

* You run one or more workers. A worker is a GPU server that serves one model.
* Your workers serve real, paid inference requests from Engy users.
* Engy measures the work that you serve and converts it into weights on SN53. You earn emissions in proportion to your weight.

## Get started

1. [Check the requirements](overview/requirements.md) for hardware, network, and models.
2. [Create your provider account](onboarding/create-your-provider-account.md) with your [SN53 hotkey](overview/subnet-53.md#register-your-hotkey).
3. [Boot a worker](onboarding/boot-a-worker.md) with the Engy image. It appears in the portal automatically. Then [register its endpoint](onboarding/register-the-endpoint.md) to choose the model and SSH port.

## Documentation

| Section | Content |
| --- | --- |
| Overview | How the network works, subnet 53, requirements, and supported models |
| Onboarding | Account creation and the steps to bring a worker online |
| TEE workers | Workers on confidential-computing hardware |
| Operations | Worker states, management, monitoring, and troubleshooting |

## Provider portal

You manage your account and workers in the provider portal at [provider.engy.ai/dashboard](https://provider.engy.ai/dashboard). The portal has the following pages: **Workers**, **Gateway key**, **Rewards**, and **Notifications**.
