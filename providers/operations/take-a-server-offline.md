---
description: Take a server out of service before maintenance and bring it back, with the Miner Key. Written for agents; the portal buttons do the same thing.
---

# Take a server offline

This page tells an agent how to take a server out of service before it stops, reboots, or changes it, and how to bring the server back afterwards. It is written for agents that operate servers for a miner, and for miners who script their maintenance. The portal buttons **Take offline** and **Bring back online** on a server's page do the same thing.

Raw link for agents: `https://raw.githubusercontent.com/hanlinai/docs/main/providers/operations/take-a-server-offline.md`

## Why it matters

* Servers may leave and come back at any time. Offline time is unpaid, but it is never penalised.
* A server that goes away **without** being taken offline first, so that buyer requests fail on it, loses **one hour** of pool uptime for that hour. It is charged at most once per clock hour.
* Taking the server offline stops new requests within about 30 seconds and lets in-flight requests finish. After that, nothing fails when the server stops.

## Rules

* **Take the server offline before** you stop the worker container, reboot, upgrade drivers, change GPUs or disks, or move the server. This includes "quick" restarts.
* **Wait for `safeToStop`** before you stop anything. This usually takes under 2 minutes; do not wait more than 10.
* **Bring back the same machine.** Engy pins the server's identity when it goes offline: the SSH endpoint, the GPU class, GPU UUIDs and serials, the SSH host key, machine-id, and NIC MACs. A return with a different SSH endpoint or different GPUs is refused, and the server must be onboarded again. Changes to the host key, machine-id, or MACs (for example after an OS reinstall) are reviewed by hand.
* **Come back within 7 days.** A server that is still offline after 7 days must be onboarded again.
* The operation applies to the whole **server**: every worker on it stops taking requests, including workers that appear while it is offline. Do not use it to stop a single worker.
* Never print the Miner Key, and never write it to a log or a file in a repository.
* When this page does not cover a situation, stop and report it to the miner.

## Inputs

| Input | Details |
| --- | --- |
| Miner Key | The key that starts with `mk-`, shown in the portal as **Gateway key**. Set it as `ENGY_MINER_KEY`. |
| Server ID | `srv-...`. It is the `server_id` in the onboarding script's summary. You can also find it with `GET /api/portal/servers`; see [Find the server ID](#find-the-server-id). |

The base URL is `https://provider.engy.ai`. Every call below sends the key in the `Authorization: Bearer` header.

## Procedure

1. **Take the server offline.**

   ```bash
   curl -sS -X POST "https://provider.engy.ai/api/portal/servers/$SERVER_ID/availability" \
     -H "Authorization: Bearer $ENGY_MINER_KEY" -H "Content-Type: application/json" \
     -d '{"action": "offline", "note": "driver upgrade"}'
   ```

   On success the response is `{"ok": true, "view": {...}}` and `view.availability` is `"offline"`. The call is idempotent: repeating it while the server is offline changes nothing.

2. **Wait until it is safe to stop.** Every 10 seconds, call:

   ```bash
   curl -sS -H "Authorization: Bearer $ENGY_MINER_KEY" \
     "https://provider.engy.ai/api/portal/servers/$SERVER_ID/availability"
   ```

   Continue when `safeToStop` is `true`, which means no requests are in flight.

   **Expected wait: usually under 2 minutes.** Most requests finish within seconds; only about 2% run longer than 2 minutes, and a few long generations run 5–10 minutes. If `safeToStop` is still `false` after 2 minutes, keep polling. After **10 minutes** you may stop the server anyway. Requests that fail while the server is offline are never penalised; only those few buyers are affected.

3. **Do the maintenance.**

4. **Bring the server back** once the worker container is running again on the same machine:

   ```bash
   curl -sS -X POST "https://provider.engy.ai/api/portal/servers/$SERVER_ID/availability" \
     -H "Authorization: Bearer $ENGY_MINER_KEY" -H "Content-Type: application/json" \
     -d '{"action": "online"}'
   ```

   `view.availability` becomes `"returning"`.

5. **Poll until the server is back.** Every 60 seconds, call the same `GET`. Stop polling and report to the miner when either of these happens:
   * `availability` is `"online"`: the server is serving again.
   * `availability` is `"rejected"`: read `reason`. The server must be onboarded again, so follow [Onboard with an agent](../onboarding/onboard-with-an-agent.md).

   A return is reviewed by Engy and is not instant. Engy approves it, runs an SSH hardware and identity re-check, and on pool servers waits for each worker to pass a liveness probe. `returnApprovedAt` is `null` until the return is approved. Do not repeat the `online` call to speed it up.

To cancel a pending return and stay offline, send `{"action": "offline"}` again.

## States

| `availability` | Meaning | What to do |
| --- | --- | --- |
| `online` | The server serves requests. | Take it offline before maintenance. |
| `offline` | Out of service. New requests stopped; in-flight requests finish. | Wait for `safeToStop`, do the maintenance, then send `online`. |
| `returning` | Return requested and under review. | Poll. `returnApprovedAt` shows the approval; `reason` explains a hold. |
| `rejected` | Not accepted back: identity changed, more than 7 days offline, or rejected by Engy. | Read `reason`, then onboard the server again. |

## API reference

All routes accept `Authorization: Bearer mk-...`. A server that the key does not own returns `404`.

### Get the availability

`GET /api/portal/servers/<serverId>/availability`

| Field | Meaning |
| --- | --- |
| `serverId` | The server. |
| `availability` | `online`, `offline`, `returning`, or `rejected`. See [States](#states). |
| `since` | Unix time when the server went offline. While `online`, it is when the server last came back. |
| `reason` | Your note when you took it offline, or why a return is held or was rejected. |
| `returnRequestedAt`, `returnApprovedAt` | Unix times of the return request and of the approval. `null` when there is none. |
| `expiresAt` | While `offline` or `returning`, the time after which the server must be onboarded again. |
| `inflight` | While `offline` or `returning`, the requests still running on the server's workers. Otherwise `null`. |
| `safeToStop` | `true` when `inflight` is `0`. |
| `penalties` | The server's pool penalties in the last 30 days: `hourStart`, `trigger`, `nFailed`, `firstRequestId`, `voided`. |

### Change the availability

`POST /api/portal/servers/<serverId>/availability`

| Body | Effect |
| --- | --- |
| `{"action": "offline", "note": "..."}` | Take the server offline now. `note` is optional, up to 500 characters. It also cancels a pending return. |
| `{"action": "online"}` | Request the return. This is only valid while `offline`. |

The response is `{"ok": true, "view": {...}}`, where `view` has the fields above without `penalties`.

| Error | Meaning |
| --- | --- |
| `400` | Bad `action`. |
| `401` | Missing or invalid Miner Key. |
| `404` | Unknown server, or not your server. |
| `409` with `code: "state"` | Not allowed in the current state, for example `online` while the server is `online`, or any action on a `rejected` server. |
| `409` with `code: "conflict"` | The server changed during the call. Read it again and retry once. |

### From the server itself

A shutdown hook on the server can use `POST /api/miner/offline`. It identifies the server by its `hardware_sig` (or `server_id`) and sends the key in the body:

```json
{"miner_key": "mk-...", "hardware_sig": "<sha256>", "action": "offline" | "online" | "status"}
```

`hardware_sig` is the SHA-256 of `gpu:` followed by the comma-joined, sorted GPU UUIDs, the same value the onboarding script registers. `"status"` returns the same fields as the `GET` above. Prefer the Bearer routes when you know the server ID.

### Find the server ID

```bash
curl -sS -H "Authorization: Bearer $ENGY_MINER_KEY" https://provider.engy.ai/api/portal/servers
```

Each entry has `serverId`, `sshHost`, `sshPort`, and the server's `workers`. Match the server on `sshHost` and `sshPort`.
