---
description: Choose the model for a booted server and register the SSH endpoint that Engy uses to verify it.
---

# Register the endpoint

This page explains how to choose the model for a server and register its SSH endpoint. It is intended for providers whose server is in the `booted` state.

## Prerequisites

Before you begin, make sure you have:

* A server in the `booted` state. For details, see [Boot a worker](boot-a-worker.md)
* An SSH port on the server that Engy can reach from the internet

## Register the endpoint

{% stepper %}
{% step %}
### Open the server

In the portal, go to **Dashboard** > **Workers**, and click the ID of the server, or click **declare** next to it.
{% endstep %}

{% step %}
### Enter the model and endpoint

In the **Register endpoint** form, set the following values:

| Field | Value |
| --- | --- |
| **model** | The model to serve. The list shows the models that currently accept new workers. |
| **ssh_host** | The address at which Engy reaches the server. Enter the IP address shown on the server page, unless Engy must use a different address. |
| **ssh_port** | The external port that maps to port 22 of the container, such as `2222`. |

For help choosing a model, see [Supported models](../overview/supported-models.md).
{% endstep %}

{% step %}
### Submit the form

Click **register**. The button is available after you fill in all three fields. The server moves to the `declared` state, and Engy starts verification.
{% endstep %}
{% endstepper %}

The worker is named after the model and the server's hardware, in the form `<model>@<hardware-id>`. The hardware ID is derived from the server's GPUs, so it stays the same across reboots.

## Register with the Miner Key

The script and your agent can do this step without the browser. Send the same values to the endpoint route with your Miner Key. For details, see [Onboard with an agent](onboard-with-an-agent.md#register-or-change-the-endpoint):

```bash
curl -s -X POST \
  -H "Authorization: Bearer $ENGY_MINER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "<model>", "ssh_port": 2222}' \
  https://provider.engy.ai/api/portal/tee-workers/<worker-id>/endpoint
```

Read the worker ID from the server page or from `GET /api/portal/servers`. It is an output, not a value that you choose.

## Correct a failed registration

If verification fails, for example because the SSH port is not reachable, the form changes to **Update endpoint** and shows the values that you entered. Correct the values, and click **update & re-drive** to restart verification. If you select a different model, Engy creates a new worker for it on the same server.

## Next step

Engy verifies and activates the worker automatically. For details, see [Verification and activation](verification-and-activation.md).
