---
description: Run Engy workers on confidential-computing hardware, and understand how hardware attestation is verified.
---

# TEE workers overview

This page explains TEE workers, which run on confidential-computing hardware and are verified with a hardware attestation. It is intended for providers who operate hosts with Intel TDX or AMD SEV-SNP.

## What a TEE worker is

A TEE worker runs inside an Intel TDX or AMD SEV-SNP confidential virtual machine. During verification, Engy checks hardware evidence that the worker runs in a genuine TDX or SEV-SNP guest and, for frontier models, on genuine NVIDIA GPUs.

TEE workers use the same Engy image and the same onboarding steps as other workers. You do not need a separate procedure.

The onboarding script currently supports preferred hardware only (8 B200 or 8 B300 GPUs). It refuses a TEE host with other GPUs. On a TEE host, follow [Prepare a TEE host](prepare-a-tee-host.md) and then [Boot a worker](../onboarding/boot-a-worker.md).

| Aspect | Standard worker | TEE worker |
| --- | --- | --- |
| Image | Engy image | Engy image |
| Onboarding | [Boot a worker](../onboarding/boot-a-worker.md) | [Boot a worker](../onboarding/boot-a-worker.md) |
| Verification | SSH access and gateway key check | Hardware attestation, SSH access, and gateway key check |
| Provisioning timeline event | `cpu-tee: none` (8 B200 or 8 B300 only) | `cpu-tee: tdx` or SEV-SNP |

## How Engy determines the worker type

Engy sets the worker type from the evidence that it verifies, not from the server's own report. A worker is a TEE worker when Engy verifies an Intel TDX or AMD SEV-SNP report that is bound to that verification run. Engy determines the type again each time the worker is verified, including after a retry.

Every server needs a CPU TEE, Intel TDX or AMD SEV-SNP, except servers with 8 B200 or 8 B300 GPUs. If Engy does not receive valid CPU TEE evidence from any other server, verification fails. GPU confidential-computing mode is not required.

## Hardware requirements

A TEE host needs the following:

* A CPU with Intel TDX or AMD SEV-SNP, with the worker running in a TDX or SEV-SNP guest
* For frontier models, 8 NVIDIA data-center GPUs that produce NVIDIA-signed attestation reports, such as H100, H200, B200, or B300

For the host configuration, see [Prepare a TEE host](prepare-a-tee-host.md).

## Attestation

During verification, the Engy image collects attestation evidence from the CPU and the GPUs. Engy checks the following:

* The TDX quote or SEV-SNP report is genuine.
* The evidence is bound to this worker, your gateway key, and a fresh challenge, so that it cannot be replayed from another server.
* For frontier models, the GPU evidence is genuine, the GPU class meets the minimum for the model, and the server has exactly 8 GPUs.
* The GPUs and the platform are not bound to the current attestation of another worker.

When you switch a server to another model, Engy releases the attestation of the previous worker, so the last check does not block the new worker.

On a TDX or SEV-SNP host, if a frontier model fails the hardware checks, verification fails. The server page shows the reason.

### Frontier model requirements

| Model ID | Minimum GPU class (8 GPUs) |
| --- | --- |
| `qwen3.8-27b` | H100 |
| `deepseek-v4-flash-0731` | H100 |
| `glm-5.3-flash` | H100 |
| `deepseek-v4.1-flash` | H200 |
| `glm-5.2` | H200 |
| `glm-5.3` | H200 |
| `kimi-k3` | B300 |

A newer GPU class also qualifies, in the order A100, H100, H200, B200, B300.

`qwen3.6-35b-a3b` has no hardware requirement. On a TDX or SEV-SNP host, it is verified without the GPU class and count checks.

## Zero data retention

Users can request that only workers with zero data retention (ZDR) serve their requests. Engy certifies workers for ZDR individually. A TEE worker is not certified automatically.

To have a worker certified, [apply as an approved provider](../onboarding/apply-as-an-approved-provider.md). A certification applies to one worker. Switching a server to another model creates a new worker with a new worker ID, and certifications do not carry over to it.

## Limitations

* Consumer GPUs, such as the RTX 5090, do not produce GPU attestation reports. They cannot serve frontier models as TEE workers.
