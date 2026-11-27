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
| Storage | Enough disk space for the Engy image (about 14 GB) and the model weights. Keep the runtime state and the model weights on persistent host directories, so that they survive a container rebuild. |
| Model disk (preferred hardware) | The model weights must be on an NVMe drive of at least 0.96 TB for 8 B200 GPUs, or at least 1.92 TB for 8 B300 GPUs. The sizes are the drive's nominal capacity, as sold. Engy checks the total size of the filesystem that holds the model weights, and accepts a filesystem of at least 95% of the nominal capacity: 912 GB for 8 B200 GPUs, or 1824 GB for 8 B300 GPUs. This allows for formatting overhead. For example, a 1.92 TB drive usually shows about 1870 GB in `df`. |

## Network

| Direction | Requirement |
| --- | --- |
| Outbound | HTTPS and secure WebSocket (WSS) access to the internet, to reach the provider portal and the Engy network. |
| Inbound | One SSH port that Engy can reach. Engy uses it to verify the worker and bring serving up. |

The worker does not need a public IP address that it knows itself. Engy records the address from which the worker registers.

## Software

You run workers with the Engy image, which the onboarding script sets for you. For details, see [Quickstart](../onboarding/quickstart.md).

The image contains the inference engine, the Engy worker software, and the management tools. You can run it on any host or GPU cloud that can start a container with NVIDIA GPU access. For details, see [Boot a worker](../onboarding/boot-a-worker.md).

## Verification

Engy verifies the server before it serves traffic:

* Servers with 8 B200 or 8 B300 GPUs onboard without a CPU trusted execution environment (TEE).
* Every other server needs a CPU TEE: Intel TDX or AMD SEV-SNP.
* GPU confidential-computing mode is not required.

For details, see [TEE workers overview](../tee-workers/overview.md).
