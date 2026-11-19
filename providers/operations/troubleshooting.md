---
description: Resolve common problems with account sign-in, worker registration, verification, and traffic.
---

# Troubleshooting

This page lists common problems and their solutions. It is intended for providers who are onboarding or operating workers.

## Account

| Problem | Solution |
| --- | --- |
| The portal says that the hotkey is not registered on netuid 53. | Register the hotkey on SN53 with `btcli subnet register`. For details, see [Register your hotkey](../overview/subnet-53.md#register-your-hotkey). |
| Claiming a key fails because the hotkey already has an account. | Each hotkey can have one provider account. Sign in with the same hotkey, and use **rotate key** if you lost the key. |
| The wallet extension does not show your hotkey. | Import the hotkey into polkadot{.js}, Talisman, or SubWallet, and allow the extension to access the portal. |

## Registration

| Problem | Solution |
| --- | --- |
| The server does not appear in the **Servers** table. | Check that the container received `ENGY_MINER_KEY` at startup, that the GPUs are visible in the container, and that the server can reach `provider.engy.ai` over HTTPS. |
| The registration is rejected because the key does not match. | Check that `ENGY_MINER_KEY` contains your current gateway key. If you rotated the key, update it on the server. |

## Verification

| Problem | Solution |
| --- | --- |
| The server fails because Engy cannot reach it over SSH. | Check that the SSH port is open to the internet and maps to port 22 of the container. Correct the port with **Update endpoint**. |
| The server fails because the key does not match the issued key. | Set `ENGY_MINER_KEY` to your current gateway key, restart the container, and click **retry**. |
| A TEE server fails because the GPU class is too low or the GPU count is wrong. | Check the [frontier model requirements](../tee-workers/overview.md#frontier-model-requirements), or choose a model that your hardware supports. |
| A TEE server fails because the hardware is already in use. | The GPUs are bound to another of your workers. Switch that server to another model, and then click **retry**. |
| The **Provisioning timeline** shows `cpu-tee: none` and verification fails. | Engy did not receive a valid CPU TEE quote, and the server is not an 8 B200 or 8 B300 server. Check the configuration in [Prepare a TEE host](../tee-workers/prepare-a-tee-host.md). |

## Traffic

| Problem | Solution |
| --- | --- |
| The worker is live but receives no requests. | Check that the worker is `active` in the **Servers** table. Traffic depends on user demand for the model. |
| A verified or live server moved to `failed` with the reason `health check: unreachable after 3 consecutive probes`. | Restore SSH access on the registered port, and then click **retry**. |
| The worker moved to `degraded`. | Check the inference engine log for errors and slow responses. The worker returns to `active` after it passes a test. |
| The worker is shown as offline. | Check that the container is running and that the server has outbound network access. |
| Your score for a model is zero. | Open the current epoch on the [Epochs](https://provider.engy.ai/epochs) page, and hover over your score to see which gate failed. For the gates, see [Quality gates](../overview/subnet-53.md#quality-gates). |
