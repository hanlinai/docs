---
description: Instructions for an agent that onboards a GPU server for a miner with the Miner Key, including the portal API reference.
---

# Onboard with an agent

This page tells an agent how to onboard a GPU server for a miner with the Miner Key (shown in the portal as **Gateway key**). It is intended for agents, and for miners who want to know what an agent does on their behalf.

Raw link for agents: `https://raw.githubusercontent.com/hanlinai/docs/main/providers/onboarding/onboard-with-an-agent.md`

## Concepts

* The miner onboards a **server**. A server can run several workers, for example with some DeepSeek layouts.
* Sibling workers appear only after the first worker is verified. The server is **done** when every current worker of the server is active (`provisioned`).
* Worker IDs are outputs. Read them from a response. Never choose or invent one.
* The script currently onboards preferred hardware only: 8 B200 or 8 B300 GPUs, with enough model disk (see [Requirements](../overview/requirements.md#hardware)). A TEE host (Intel TDX or AMD SEV-SNP) with other GPUs is refused at preflight. For a TEE host, follow [Prepare a TEE host](../tee-workers/prepare-a-tee-host.md) instead.

## Inputs

Ask the miner for these before you start:

| Input | Details |
| --- | --- |
| Miner Key | The key that starts with `mk-`, shown in the portal as **Gateway key** (`/dashboard/key`). Set it as `ENGY_MINER_KEY`. |
| Root access | Root, or `sudo`, on the GPU server. |
| Model | The model to serve. Run the script without `--model` to list the choices. |

Optional: `HF_TOKEN` for gated Hugging Face weights, and `ENGY_PROVIDER_URL` (default `https://provider.engy.ai`).

## Safety rules

* Never print the Miner Key. Never commit it, write it to a log, or put it in a file in a repository.
* Never delete a server or worker unless the miner asked you to.
* Do not run the script on a machine that serves other workloads without the miner's consent. The script can install Docker and the NVIDIA container toolkit, and it starts a container that uses all GPUs.
* When this page does not cover a situation, stop and report it to the miner.

## Procedure

1. Download the script and check its checksum:

   ```bash
   curl -fsSLO https://provider.engy.ai/engy-onboard.sh
   curl -fsSLO https://provider.engy.ai/engy-onboard.sh.sha256
   sha256sum -c engy-onboard.sh.sha256
   ```

2. Check the server without changing it:

   ```bash
   ENGY_MINER_KEY="$ENGY_MINER_KEY" sh engy-onboard.sh --preflight-only
   ```

3. List the models, if the miner has not chosen one:

   ```bash
   sh engy-onboard.sh --json
   ```

   The output is `{"models": [...]}`.

4. Run the script with the model, as root:

   ```bash
   sh engy-onboard.sh --model <model> --json
   ```

5. Read the summary. Poll the server until it is done. See [Polling](#polling).

### Script options

| Option | Default | Purpose |
| --- | --- | --- |
| `--model <model>` | none | The model to serve. Without it, the script prints the available models and exits. |
| `--ssh-port <n>` | `2222` | The host port that maps to the container's SSH port. Engy must reach it from the internet. |
| `--ssh-host <host>` | recorded IP | Address Engy uses for SSH, if it differs from the address the server registered from. |
| `--data-dir <dir>` | `/data/engy` | Persistent directory for the runtime state and the model weights. Use an absolute path. |
| `--shm-size <size>` | half of RAM, up to `64g` | Shared memory for the container, such as `64g`. |
| `--switch-model` | off | Confirm that you switch a server that already runs another model. This retires its current workers. |
| `--json` | off | Print machine-readable output. |
| `--preflight-only` | off | Check the server and exit without starting anything. |

| Environment variable | Purpose |
| --- | --- |
| `ENGY_MINER_KEY` | Required. The Miner Key. |
| `ENGY_PROVIDER_URL` | Optional. Default `https://provider.engy.ai`. |
| `HF_TOKEN` | Optional. A Hugging Face token for gated weights. |

The script is safe to run again when the server is healthy: it refreshes the summary, and does nothing new when the server already runs the model. Do not rerun it to poll. When the first worker is `failed` and the model matches, a rerun re-declares the worker, which acts as a retry without checking the reason.

### Summary

With `--json`, the script prints one JSON object:

| Field | Meaning |
| --- | --- |
| `server_id` | The server's ID. |
| `state` | The server's state. See [States](#states). |
| `model` | The model of the server's first worker. |
| `ssh_host`, `ssh_port` | The SSH endpoint that Engy uses. |
| `done` | `true` when every current worker is `provisioned`, at the time the script ran. The server detail does not have this field. |
| `workers` | One entry per current worker: `slot`, `worker_id`, `model`, `provisioning`. |
| `status_url` | The API URL for this server. See `GET /api/portal/servers/<serverId>`. |

## Polling

Call `GET /api/portal/servers/<serverId>` (the `status_url` from the summary) every 60 seconds. Do not poll by rerunning the script. The response has no `done` field. Check `server.workers` instead:

* The server is done when `workers` is not empty and every `workers[].provisioning` is `provisioned`.
* Stop when any worker's `provisioning` is `failed`. Read its reason with `GET /api/portal/tee-workers/<workerId>`, because the server's `events` cover only the first worker. See [Failed workers](#failed-workers).
* Stop when the miner tells you to stop.

Typical durations, which are not guaranteed:

* Engy verifies the machine in minutes.
* Serving setup, which includes the weight download and compile, takes one to several hours, depending on the model.

Sibling workers appear after the first worker is verified. A server with one worker in `provisioned` is not done until its sibling workers are also `provisioned`.

## States

The server's `state` follows its first worker while onboarding runs. After the worker is `provisioned`, it shows the worker's routing state. For the routing states, see [Worker states](../operations/worker-states.md).

| Worker state | Meaning | Your action |
| --- | --- | --- |
| `booted` | The server registered. | The script starts the container and registers the model and endpoint. |
| `declared` | The model and endpoint are registered. | Wait. |
| `reachable`, `verifying` | Engy is verifying the machine. | Wait. |
| `verified` | The machine passed verification. | Wait. |
| `starting` | Engy is setting up serving. | Wait. |
| `awaiting_verification`, `awaiting_review` | Engy is reviewing the worker. | Wait. |
| `provisioned` | The worker is active. | None. |
| `failed` | Onboarding failed. The `reason` explains why. | See [Failures](#failures). |
| `superseded` | A new model on the same server replaced this worker. | None. |
| `deleted` | The worker was deleted. | None. |

## API reference

Every route below except self-register, which takes the key in the body, accepts `Authorization: Bearer mk-...`. Key claim, key rotation (`POST /api/portal/key`), and sign-in accept a wallet signature only.

All routes use `https://provider.engy.ai` as the base URL (or `ENGY_PROVIDER_URL`). Send JSON bodies with `Content-Type: application/json`.

Errors common to all routes except self-register:

| Status | Body | Meaning |
| --- | --- | --- |
| `401` | `{"error":"login required"}` | The key is wrong, rotated, revoked, or the miner is banned. |
| `404` | `{"error":"no such worker"}` or `{"error":"no such server"}` | The ID does not exist, or it belongs to another miner. |

### Self-register

`POST /api/portal/self-register`. The worker container calls this on boot. The script does it for you, so call it only if you run the image yourself. It sends the Miner Key in the body, not in a header.

Request fields: `hardware_sig` (required), `miner_key` (required), `gpu_model`, `gpu_count`, `gpu_ids`, `model`, `caps`.

Response `200` (existing server) or `201` (new server): `{"ok": true, "worker_id", "model", "created", "workers": [...]}`.

Errors: `400` for a missing `hardware_sig` or an unknown client address, `401` for a missing `miner_key`, `403` when the key is refused.

### List servers

`GET /api/portal/servers`

```bash
curl -s -H "Authorization: Bearer $ENGY_MINER_KEY" https://provider.engy.ai/api/portal/servers
```

Response: `{"servers": [...], "epoch": {"index": n}}`. Each server has `serverId`, `legacy`, `ip`, `gpu` (`model`, `count`), `provisioning`, `reason`, `state`, `registered`, `updatedAt`, `sshHost`, `sshPort`, and `workers`. Each worker has `workerId`, `model`, `status`, `live`, `metrics`, `slot`, `gpus`, `servePort`, and `provisioning`.

### Get a server

`GET /api/portal/servers/<serverId>`

Response: `{"server": {...}, "events": [...]}`. The `server` object has the fields above. `events` is the timeline of the server's first worker, oldest first. Each event has `from`, `to`, `actor`, `reason`, and `createdAt`.

Errors: `404` `{"error":"no such server"}`.

### Get a worker

`GET /api/portal/tee-workers/<workerId>`

Response: `{"worker": {...}, "events": [...]}`. The `worker` object has `workerId`, `model`, `sshHost`, `sshPort`, `gpuModel`, `gpuCount`, `gpuIds`, `provisioning`, `reason`, `createdAt`, `updatedAt`, and `retryable`. The GPU fields are what the server reported when it booted.

### Register or change the endpoint

`POST /api/portal/tee-workers/<workerId>/endpoint`

Request:

```json
{"model": "<model>", "ssh_host": "203.0.113.10", "ssh_port": 2222}
```

`model` and `ssh_port` (1 to 65535) are required. `ssh_host` is optional and defaults to the address the server registered from. The script calls this for you.

Response: `{"ok": true, "worker_id": "...", "worker_ids": [...], "provisioning": "declared"}`. When the call switches the model, the response also has `supersedes`, the IDs of the retired workers.

Errors:

| Status | Meaning |
| --- | --- |
| `400` | `model` is missing, `ssh_port` is invalid, or no address is recorded and `ssh_host` is missing. |
| `409` | The worker is not in a state that accepts an endpoint, the server already runs this model (choose another to switch), or the SSH endpoint belongs to another server. |
| `422` | The server's hardware is not accepted for this model. The body lists `allowed_hardware`. |

Choosing a different model on a server retires its current workers and creates new ones.

### Retry

`POST /api/portal/tee-workers/<workerId>/retry`

Call it only for a worker in `failed`.

Response: `{"ok": true, "worker_id": "...", "provisioning": "declared"}`.

Errors: `409` `{"error": "not retryable from '<state>'", "provisioning": "<state>"}` for `allocated`, `booted`, `declared`, `provisioned`, `deleted`, and `superseded`. `422` when the hardware is not accepted for the worker's model.

### Delete

`POST /api/portal/tee-workers/<workerId>/delete`

Response: `{"ok": true, "worker_id": "...", "provisioning": "deleted"}`. Deleting a server's first worker deletes the whole server. A worker that is already deleted returns the same response.

Errors: `409` when the worker, or a sibling worker of the server, is `provisioned`. Delete only when the miner asked.

### List models

`GET /api/portal/models`

Response: `{"models": ["<model>", ...]}`, the models that currently accept new workers.

## Failures

| Exit code | Meaning | What to do |
| --- | --- | --- |
| `0` | Success. | Read the summary. |
| `2` | Bad input, such as a missing or malformed Miner Key, or an invalid option. | Fix the input, and rerun. |
| `3` | Preflight failed. The message names the check: root access, GPUs and NVIDIA driver (580 or later), verification policy (preferred hardware and its model-disk floor), free disk space, SSH port in use, reachability of the gateway and the provider, or `python3`. | Report it to the miner. Fix only what the miner approves. |
| `4` | Docker or the worker container failed. | Read the Docker output. Check `docker compose -p engy logs`. |
| `5` | A provider API call failed. | Check the Miner Key and the network, and rerun. |
| `6` | The model and endpoint were refused: `422` means the hardware is not accepted for that model, and `409` means a conflict, such as the server already running another model. | For `409` on a switch, rerun with `--switch-model` only if the miner agreed. For `422`, tell the miner which hardware the response allows. |

In your own `curl` calls, `-H "Authorization: Bearer $ENGY_MINER_KEY"` is acceptable. Do not echo the value.

### Failed workers

A worker in `failed` shows its `reason` in `GET /api/portal/tee-workers/<workerId>`: the `reason` field and the worker's `events`. Call retry only for a worker in `failed`. Never call it for a worker in another state, because retry on an in-flight state resets it to `declared`. After you or the miner correct the cause, call retry. Retryable `failed` reasons you can correct:

* The SSH port is not reachable from the internet. Open it, or register a different one through the endpoint route, and then retry.
* `health check: unreachable after 3 consecutive probes`. Restore SSH access on the registered port, and then retry.

If the reason points to anything else, such as an internal error or a hardware mismatch, stop and report it to the miner. For the miner-side flow, see [Manage workers](../operations/manage-workers.md) and [Troubleshooting](../operations/troubleshooting.md).
