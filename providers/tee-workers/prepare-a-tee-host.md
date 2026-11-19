---
description: Configure an Intel TDX host with multi-GPU NVIDIA confidential computing for an Engy TEE worker.
---

# Prepare a TEE host

This page explains how to configure a confidential-computing host before you boot a TEE worker. It is intended for providers who operate servers with Intel TDX.

## Prerequisites

Before you begin, make sure you have:

* A server with an Intel CPU that supports TDX
* For frontier models, 8 NVIDIA data-center GPUs, such as H100, H200, B200, or B300
* A TDX-enabled host operating system and firmware
* Administrator access to the host

## Configure the host

{% stepper %}
{% step %}
### Enable multi-GPU protected mode

For servers with 8 GPUs, use the Protected PCIe (PPCIe) multi-GPU mode. This mode protects the GPUs as a group and keeps NVLink peer-to-peer communication available.

```bash
nvidia-smi conf-compute -mgm
```

Confirm that the output reports the multi-GPU mode as Protected PCIe.

{% hint style="warning" %}
Do not enable per-GPU confidential-computing mode on an 8-GPU server. It disables NVLink peer-to-peer communication, and large models cannot be served across the GPUs.
{% endhint %}
{% endstep %}

{% step %}
### Check the confidential-computing environment

Confirm the following settings:

| Setting | Expected value |
| --- | --- |
| Multi-GPU mode | Protected PCIe |
| Per-GPU CC state | `OFF` |
| CC environment | `PRODUCTION` |

To check the per-GPU CC state and the CC environment, run the following commands:

```bash
nvidia-smi conf-compute -f
nvidia-smi conf-compute -e
```

In Protected PCIe mode, the per-GPU CC state is `OFF`. This value is correct.
{% endstep %}

{% step %}
### Start a TDX guest

Run the worker in a TDX guest with the GPUs assigned to it. In the guest, confirm that the CPU reports the `tdx_guest` flag:

```bash
grep -o tdx_guest /proc/cpuinfo | head -n 1
```

The worker must be able to request a TDX quote from inside the container:

* **dstack-based confidential VM**: Mount the guest agent socket into the container with `-v /var/run/dstack.sock:/var/run/dstack.sock`.
* **Other TDX guests**: The guest kernel must support configfs-tsm quote generation, and the container needs access to `/sys/kernel/config/tsm/report`.
{% endstep %}

{% step %}
### Boot the worker

In the TDX guest, boot the Engy image as described in [Boot a worker](../onboarding/boot-a-worker.md). Engy collects the attestation during verification.
{% endstep %}
{% endstepper %}

## Confirm the worker type

After verification, open the server in the portal and check its **Provisioning timeline**:

| Event | Meaning |
| --- | --- |
| `cpu-tee: tdx` | Engy verified the TDX quote. The worker is a TEE worker. |
| `cpu-tee: none` | Engy did not receive a valid TDX quote. The worker is verified as a standard worker. |

If the event is `cpu-tee: none`, check the configuration above, and then click **retry**. Retry is available until the worker is activated.
