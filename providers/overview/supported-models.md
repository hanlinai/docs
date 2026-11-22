---
description: The models that providers can serve on Engy, their fixed checkpoints, and the minimum hardware for each.
---

# Supported models

This page lists the models that you can serve on the Engy network. It is intended for providers who choose which model to run on their hardware.

## Models

Each model is served from one fixed Hugging Face checkpoint. Serve exactly this checkpoint. Serving a different or modified checkpoint under a model's ID is not allowed.

| Model ID | Checkpoint | Minimum hardware |
| --- | --- | --- |
| `qwen3.6-35b-a3b` | `Qwen/Qwen3.6-35B-A3B-FP8` | One GPU with 48 GB of memory, or two RTX 4090 or RTX 5090 GPUs |
| `qwen3.8-27b` | `Qwen/Qwen3.8-27B` | 8 × H100 |
| `deepseek-v4-flash-0731` | `deepseek-ai/DeepSeek-V4-Flash-0731` | 8 × H100 |
| `deepseek-v4.1-flash` | `deepseek-ai/DeepSeek-V4.1-Flash` | 8 × H200 |
| `glm-5.2` | `zai-org/GLM-5.2` | 8 × H200 |
| `glm-5.3` | `zai-org/GLM-5.3` | 8 × H200 |
| `glm-5.3-flash` | `zai-org/GLM-5.3-Flash` | 8 × H100 |
| `kimi-k3` | `moonshotai/Kimi-K3` | 8 × B300 |
| `MiniMax-H3` | `MiniMaxAI/MiniMax-H3` | Video generation model |

The GPU class is a minimum. A newer class also qualifies, in the order A100, H100, H200, B200, B300. Engy checks the hardware during verification. The GPU class must be the listed class or newer, and the GPU count must match exactly.

When you register a worker, the portal lists the models that are currently accepting new workers. The list can differ from this table.

## Score rates

Each model has its own score rate, which determines how much your work on that model is worth. The score rate is based on the per-token prices that users pay, which are listed in [Pricing](https://engy.ai/pricing).

## Choose a model

* Check the demand for each model on the [Requests](https://provider.engy.ai/requests) page of the network explorer.
* Choose a model that your hardware can serve with low latency. Slow responses and timeouts can fail the quality gates.
* You can switch a server to another model later. The switch creates a new worker with a new worker ID. For details, see [Manage workers](../operations/manage-workers.md#switch-the-model).
