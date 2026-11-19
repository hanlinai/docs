---
description: The hotkey, hardware, network, and software requirements for running Engy workers.
---

# Requirements

This page lists what you need before you onboard a worker. It is intended for providers who are preparing hardware for the Engy network.

## Account

* A Bittensor wallet with a hotkey registered on SN53. For details, see [Register your hotkey](subnet-53.md#register-your-hotkey).
* A browser wallet extension that holds the hotkey: polkadot{.js}, Talisman, or SubWallet. You use it to sign in to the provider portal.

## Hardware

| Requirement | Details |
| --- | --- |
| GPU | NVIDIA GPUs. The GPU class and count depend on the model. For details, see [Supported models](supported-models.md). |
| Architecture | x86-64 (amd64). Arm64 systems, such as GB200 and GB300, are not supported. |
| Storage | Enough disk space for the Engy image (about 14 GB) and the model weights. Store the model weights on a persistent volume, so that they survive a container rebuild. |

## Network

| Direction | Requirement |
| --- | --- |
| Outbound | HTTPS and secure WebSocket (WSS) access to the internet, to reach the provider portal and the Engy network. |
| Inbound | One SSH port that Engy can reach. Engy uses it to verify the worker and bring serving up. |

The worker does not need a public IP address that it knows itself. Engy records the address from which the worker registers.

## Software

You run workers with the Engy image:

```text
ghcr.io/hanlinai/engy-golden-serve:prod
```

The image contains the inference engine, the Engy worker software, and the management tools. You can run it on any host or GPU cloud that can start a container with NVIDIA GPU access. For details, see [Boot a worker](../onboarding/boot-a-worker.md).

## Confidential computing

Confidential computing is optional for every model. Hosts with Intel TDX are verified with a hardware attestation. For details, see [TEE workers overview](../tee-workers/overview.md).
