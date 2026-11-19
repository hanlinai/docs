---
description: Boot a GPU server with the Engy image and your gateway key, so that it registers itself in the provider portal.
---

# Boot a worker

This page explains how to start a worker with the Engy image. It is intended for providers who have claimed a gateway key and are bringing their first server online.

## How onboarding works

Onboarding a worker takes five stages. You perform the first and third. Engy performs the rest automatically.

| Stage | Performed by | Resulting state |
| --- | --- | --- |
| 1. Boot the image with your gateway key | You | — |
| 2. The server registers itself | Engy image | `booted` |
| 3. Register the model and SSH endpoint | You | `declared` |
| 4. Verify the server | Engy | `verified` |
| 5. Bring serving up and activate the worker | Engy | `starting`, then `active` |

This page covers stages 1 and 2. For stage 3, see [Register the endpoint](register-the-endpoint.md). For stages 4 and 5, see [Verification and activation](verification-and-activation.md).

## Prerequisites

Before you begin, make sure you have:

* Your gateway key. For details, see [Create your provider account](create-your-provider-account.md)
* A server that meets the [requirements](../overview/requirements.md)

## Boot the image

{% hint style="info" %}
The fastest way to do stages 1 to 3 is the onboarding script. For details, see [Quickstart](quickstart.md). The steps below start the container yourself.
{% endhint %}

{% stepper %}
{% step %}
### Get the image reference

Engy pins the image by digest. Download the onboarding script, and read the pinned reference from it:

```bash
curl -fsSLO https://provider.engy.ai/engy-onboard.sh
grep '^ENGY_IMAGE=' engy-onboard.sh
```

The value after `:-` is the reference, in the form `ghcr.io/...@sha256:...`. Use it exactly. Do not use a tag.

```bash
docker pull "<image reference>"
```

The image is about 14 GB. The first pull can take several minutes.
{% endstep %}

{% step %}
### Start the container with your gateway key

Set `ENGY_MINER_KEY` to your gateway key in the container environment, give the container GPU access, and publish its SSH port. The image runs its SSH server on port 22 inside the container. The container needs two persistent host directories. The following example uses `/data/engy` and publishes the SSH port on host port `2222`:

```bash
mkdir -p /data/engy/runtime /data/engy/models

docker run -d --name engy-worker \
  --gpus all \
  --shm-size 64g \
  -p 2222:22 \
  -e ENGY_MINER_KEY="mk-..." \
  -v /data/engy/runtime:/root/engy \
  -v /data/engy/models:/root/models \
  "<image reference>"
```

* `/root/engy` keeps the runtime state and deployment files. Engy relies on them after a restart.
* `/root/models` keeps the model weights, so that they survive container rebuilds.

Note the host port. You enter it as the SSH port when you register the endpoint. On a GPU cloud, map container port 22 to an external port in the template, and use the external port.

`ENGY_MINER_KEY` is the only value that you set. Do not set a worker ID or a model. Engy assigns the worker ID, and you choose the model in the portal or with the script.

If the model weights are gated on Hugging Face, also set `HF_TOKEN` to a Hugging Face access token.

{% hint style="warning" %}
Set the variables in the container configuration before the container starts, for example in a GPU cloud template. Variables that you export later in an SSH session are not used.
{% endhint %}
{% endstep %}

{% step %}
### Confirm the registration

When the container starts, it registers the server with Engy. Engy records the server's GPUs and the IP address from which it registered.

In the portal, go to **Dashboard** > **Workers**. The **Servers** page lists the server with its IP address and the state `booted`.
{% endstep %}
{% endstepper %}

## Behavior to know

* **Reboots**: Engy identifies a server by a signature of its GPUs. When you reboot it, it registers as the same server, keeps its onboarding state, and resumes serving the model that you last registered.
* **Deleted servers**: If you deleted a server, booting the same hardware again restores it in the `booted` state.
* **SSH access**: The image accepts only Engy's management SSH key. SSH keys that a host or GPU cloud injects are ignored.
* **Servers behind NAT**: The server does not need to know its public IP address. Engy uses the address from which the registration arrived.

## Troubleshooting

If the server does not appear in the **Servers** table, check the following:

* The container received `ENGY_MINER_KEY` at startup.
* The server has an NVIDIA GPU that is visible in the container.
* The server can reach `provider.engy.ai` over HTTPS.

The image registers the server only when the container starts. After you correct the problem, restart the container so that it registers again.

## Next step

[Register the endpoint](register-the-endpoint.md) to choose the model and the SSH port.
