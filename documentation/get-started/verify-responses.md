---
description: Request a cryptographic proof with a response and verify that the advertised model produced the output.
---

# Verify responses

Engy can attach a cryptographic proof to any response. You can then recompute the proof against the model weights to confirm that the advertised model generated the output. This guide is intended for developers who want to verify individual responses independently.

## How verification works

When you request a proof, the GPU worker that serves your request records a fingerprint of the model's internal activations for each generated token. This fingerprint is a TOPLOC proof.

To verify the proof, you run the same model on your own hardware, recompute the activations for the same tokens, and compare them with the proof:

* If the activations match, the advertised model produced the output.
* If the worker used a different, smaller, or modified model, the activations do not match, and verification fails.

Engy passes the proof to you unchanged. Engy does not store the proof, so requesting one does not affect zero data retention.

## Request a proof

Set `provider.require_proof` to `true` in the request body:

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.engy.ai/v1/chat/completions \
  -H "Authorization: Bearer $ENGY_API_KEY" \
  -H "Content-Type: application/json" \
  -o response.json \
  -d '{
    "model": "qwen3.8-27b",
    "messages": [{"role": "user", "content": "Hello"}],
    "provider": {"require_proof": true}
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import json
import os

import httpx

response = httpx.post(
    "https://api.engy.ai/v1/chat/completions",
    headers={"Authorization": f"Bearer {os.environ['ENGY_API_KEY']}"},
    json={
        "model": "qwen3.8-27b",
        "messages": [{"role": "user", "content": "Hello"}],
        "provider": {"require_proof": True},
    },
    timeout=600,
)
response.raise_for_status()

with open("response.json", "w") as f:
    json.dump(response.json(), f)
```
{% endtab %}
{% endtabs %}

Engy routes the request only to workers that can produce proofs. If no such worker is available for the model, the request fails with HTTP status `503`.

## Where the proof appears

The proof is returned in a top-level `proof` field. Its location depends on the endpoint and on whether you stream the response:

| Endpoint | Non-streaming | Streaming |
| --- | --- | --- |
| `/v1/chat/completions` | Top-level `proof` field | Final chunk, before `[DONE]` |
| `/v1/completions` | Top-level `proof` field | Final chunk, before `[DONE]` |
| `/v1/messages` | Top-level `proof` field on the message | `message_delta` event |
| `/v1/responses` | Top-level `proof` field on the response | `response.completed` event |

If you did not request a proof, or the worker returned none, the `proof` field is absent.

## Proof format

```json
{
  "proof": {
    "model_root": "...",
    "output_token_ids": [...],
    "toploc": {
      "proofs": ["..."],
      "topk": 128,
      "decode_batching_size": 32,
      "skip_prefill": true,
      "n_tokens": 9,
      "prompt_ids": [...]
    }
  }
}
```

| Field | Description |
| --- | --- |
| `model_root` | The identifier of the model checkpoint that the worker committed to. |
| `output_token_ids` | The token IDs that the model generated. |
| `toploc.proofs` | The TOPLOC proof segments, encoded in Base64. |
| `toploc.topk` | The number of activation values that each proof segment covers. |
| `toploc.decode_batching_size` | The number of tokens in each proof segment. |
| `toploc.skip_prefill` | Whether the proof excludes the prompt positions. |
| `toploc.n_tokens` | The number of output tokens that the proof covers. |
| `toploc.prompt_ids` | The prompt token IDs, after the chat template was applied. |

## Verify a proof

### Prerequisites

Before you begin, make sure you have:

* A GPU with enough memory to run the model. For example, `qwen3.8-27b` runs on a single NVIDIA H200 GPU.
* The model checkpoint from Hugging Face. Each model is served from one fixed checkpoint, and you must verify against the same checkpoint. For `qwen3.8-27b`, use `Qwen/Qwen3.8-27B`. For other models, see [Models and pricing](models-and-pricing.md).
* Python 3.10 or later
* A saved response that contains a proof, such as `response.json` from the previous section

### Steps

{% stepper %}
{% step %}
### Install the dependencies

```bash
pip install "sglang[all]" toploc torch httpx numpy
```
{% endstep %}

{% step %}
### Start a local inference server

Start SGLang with hidden-state output enabled and the prefix cache disabled:

```bash
python -m sglang.launch_server \
  --model-path Qwen/Qwen3.8-27B \
  --enable-return-hidden-states \
  --disable-radix-cache \
  --port 30000
```

{% hint style="warning" %}
You must disable the prefix cache with `--disable-radix-cache`. Otherwise, the server returns activations for only part of the sequence, and verification fails.
{% endhint %}
{% endstep %}

{% step %}
### Run the verification script

Save the following script as `verify.py`:

{% code title="verify.py" %}
```python
import json
import sys

import httpx
import numpy as np
import torch
from toploc import verify_proofs_base64

SERVER_URL = "http://127.0.0.1:30000"
THRESHOLD = 40

with open(sys.argv[1]) as f:
    proof = json.load(f)["proof"]

toploc = proof["toploc"]
prompt_ids = toploc["prompt_ids"]
n_tokens = toploc["n_tokens"]
output_ids = proof["output_token_ids"][:n_tokens]
full_ids = prompt_ids + output_ids

# Recompute the activations with a single prefill pass.
result = httpx.post(
    f"{SERVER_URL}/generate",
    json={
        "input_ids": full_ids,
        "sampling_params": {"temperature": 0.0, "max_new_tokens": 0},
        "return_hidden_states": True,
    },
    timeout=600,
).json()
result = result[0] if isinstance(result, list) else result
block = np.asarray(result["meta_info"]["hidden_states"][0], dtype=np.float32)

# The returned block covers the last positions of the sequence.
block_start = len(full_ids) - block.shape[0]
prompt_len = len(prompt_ids)
activations = [
    torch.from_numpy(
        np.ascontiguousarray(block[prompt_len - 1 + t - block_start])
    ).to(torch.bfloat16)
    for t in range(n_tokens)
]

results = verify_proofs_base64(
    activations,
    toploc["proofs"],
    decode_batching_size=toploc["decode_batching_size"],
    topk=toploc["topk"],
    skip_prefill=toploc["skip_prefill"],
)

mismatches = float(np.mean([r.exp_mismatches for r in results]))
status = "PASS" if mismatches <= THRESHOLD else "FAIL"
print(f"{status}: {mismatches:.1f} of {toploc['topk']} values mismatched")
```
{% endcode %}

Run the script with your saved response:

```bash
python verify.py response.json
```
{% endstep %}
{% endstepper %}

## Interpret the result

The script reports the average number of mismatched activation values per proof segment, out of `topk`:

| Result | Typical mismatches (of 128) | Meaning |
| --- | --- | --- |
| `PASS` | About 14 | The advertised model produced the output. |
| `FAIL` | About 113 | A different model or modified weights produced the output. |

Small mismatches are expected, because different GPUs produce slightly different floating-point results. The default threshold of 40 separates the two cases with a wide margin.

## Limitations

* Requesting a proof adds compute and bandwidth to the request, so proofs are not generated by default.
* Only workers that can produce proofs serve these requests. As a result, capacity for some models might be limited.
* Verification requires hardware that can run the full model.
