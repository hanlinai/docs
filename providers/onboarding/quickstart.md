---
description: Onboard a GPU server with one script and your Miner Key, or hand the job to your own agent.
---

# Quickstart

This page explains the fastest way to onboard a GPU server. It is intended for providers who have a Miner Key and root access to the server.

A Miner Key is the key that the portal shows under **Gateway key**. For details, see [Create your provider account](create-your-provider-account.md).

You onboard a server, not a single worker. A server can run several workers, for example with some DeepSeek layouts. Sibling workers appear after the first worker is verified. The server is done when every worker of the server is active.

## Onboard a server

{% stepper %}
{% step %}
### Download the script and check it

On the server, as root, download the script and its checksum, and verify it:

```bash
curl -fsSLO https://provider.engy.ai/engy-onboard.sh
curl -fsSLO https://provider.engy.ai/engy-onboard.sh.sha256
sha256sum -c engy-onboard.sh.sha256
```

The last command prints `engy-onboard.sh: OK` when the script is intact.
{% endstep %}

{% step %}
### Choose a model

Set your Miner Key and run the script without `--model`. It prints the models that currently accept new workers:

```bash
export ENGY_MINER_KEY="mk-..."
sh engy-onboard.sh
```
{% endstep %}

{% step %}
### Run the script with the model

```bash
sh engy-onboard.sh --model <model>
```

The script checks the server, starts the Engy worker container, registers the server, and registers the model and SSH endpoint for you. It then prints a summary and exits. Engy verifies the machine and sets up serving on its own.

The server is done when the summary shows `done: yes`. Watch the server under **Servers** in the portal. Do not rerun the script to check progress. Verification typically takes minutes. Serving setup, which includes the weight download and compile, typically takes one to several hours, depending on the model.
{% endstep %}
{% endstepper %}

For the options and exit codes, see [Onboard with an agent](onboard-with-an-agent.md#failures).

## Let an agent do it

Give your agent this link and your Miner Key:

```text
https://raw.githubusercontent.com/hanlinai/docs/main/providers/onboarding/onboard-with-an-agent.md
```

The page tells the agent how to run the script, read the result, poll for completion, and handle failures. For details, see [Onboard with an agent](onboard-with-an-agent.md).
