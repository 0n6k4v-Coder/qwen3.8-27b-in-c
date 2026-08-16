# SET 0 — Vision and MTP Architecture

## Document Status

* Document: `06-vision-and-mtp.md`
* SET: `SET 0 — Model Reconnaissance`
* Source Tasks: `SET0-T09`, `SET0-T14`, `SET0-CLOSE-FIX-FINALIZE`
* Status: **VERIFIED**
* Responsibility: 🧠 LUNA

---

# 1. Source and Provenance

Model: `Qwen3.8-27B`

Official repository: `Qwen/Qwen3.8-27B`

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary configuration:

```text
model/official/config.json
```

Primary tensor metadata:

```text
model/official/TENSOR-METADATA.md
```

Verified tensor index:

```text
model/official/model.safetensors.index.json
```

---

# 2. Vision Configuration

Verified configuration establishes:

```text
language_model_only = false
vision_config = present
vision depth = 27
vision hidden size = 1152
vision heads = 16
vision intermediate size = 4304
vision out_hidden_size = 5120
in_channels = 3
patch_size = 16
spatial_merge_size = 2
temporal_patch_size = 2
num_position_embeddings = 2304
activation = gelu_pytorch_tanh
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 3. Vision / Language Interface

The examined official implementation establishes the conceptual path:

```text
Image / Video
      ↓
Vision Encoder
      ↓
Visual Features
      ↓
Image / Video Token Positions
      ↓
Language Embedding Sequence
      ↓
64-Layer Language Model
```

The language hidden width is 5120 and the vision output width is 5120.

### Evidence Classification

```text
VERIFIED FACT
```

for the examined implementation/configuration path.

---

# 4. MTP Configuration

The verified language configuration contains:

```text
mtp_num_hidden_layers = 1
mtp_use_dedicated_embeddings = false
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 5. MTP Checkpoint Tensors

`model/official/TENSOR-METADATA.md` directly verifies the complete MTP
checkpoint tensor set.

All 15 tensors are BF16 and are stored in:

```text
model-00018-of-00018.safetensors
```

The complete checkpoint tensor set is:

```text
mtp.fc.weight
mtp.layers.0.input_layernorm.weight
mtp.layers.0.mlp.down_proj.weight
mtp.layers.0.mlp.gate_proj.weight
mtp.layers.0.mlp.up_proj.weight
mtp.layers.0.post_attention_layernorm.weight
mtp.layers.0.self_attn.k_norm.weight
mtp.layers.0.self_attn.k_proj.weight
mtp.layers.0.self_attn.o_proj.weight
mtp.layers.0.self_attn.q_norm.weight
mtp.layers.0.self_attn.q_proj.weight
mtp.layers.0.self_attn.v_proj.weight
mtp.norm.weight
mtp.pre_fc_norm_embedding.weight
mtp.pre_fc_norm_hidden.weight
```

Therefore:

```text
MTP configuration:
VERIFIED

MTP checkpoint tensors:
VERIFIED

MTP exact tensor metadata:
VERIFIED

MTP checkpoint tensor count:
15

MTP dtype:
BF16

MTP shard:
model-00018-of-00018.safetensors
```

### Evidence Classification

```text
VERIFIED FACT
```

The exact MTP tensor metadata is directly backed by raw tensor metadata and
must not be classified as unresolved implementation-derived information.

---

# 6. MTP Runtime Boundary

Checkpoint presence does not establish active runtime execution.

The current evidence therefore preserves the following boundary:

```text
MTP configuration:
VERIFIED

MTP checkpoint tensors:
VERIFIED

MTP exact tensor metadata:
VERIFIED

MTP active runtime execution path:
UNKNOWN
```

Do not infer from checkpoint presence that ordinary generation necessarily
executes the MTP subnetwork.

Runtime activation, scheduling, memory behavior, and performance remain
separate runtime questions.

---

# 7. MTP Tensor Metadata Boundary

The previous statement that exact MTP tensor shapes remained unresolved is
superseded by the direct raw metadata verification recorded in
`model/official/TENSOR-METADATA.md`.

The exact shape mapping is now a **VERIFIED FACT** at the checkpoint metadata
level. Runtime semantics remain **UNKNOWN**.

---

# 8. Canonical MTP Statement

> **The Qwen3.8-27B artifact declares one MTP hidden layer and disables
> dedicated MTP embeddings. Raw tensor metadata directly verifies a complete
> 15-tensor MTP checkpoint set, all BF16 and stored in
> `model-00018-of-00018.safetensors`. Exact MTP tensor metadata is therefore
> VERIFIED FACT. Whether the MTP subnetwork is actively executed during
> ordinary generation remains UNKNOWN and must not be inferred from
> checkpoint presence alone.**

---

# 9. Research Boundary

Established:

```text
Vision configuration: VERIFIED
Vision checkpoint presence: VERIFIED
MTP configuration: VERIFIED
MTP checkpoint tensors: VERIFIED
MTP exact tensor metadata: VERIFIED
MTP tensor count: 15
MTP dtype: BF16
MTP shard: model-00018-of-00018.safetensors
```

Not established:

```text
MTP active runtime execution path
MTP runtime scheduling
MTP runtime memory behavior
MTP performance contribution
hardware-specific MTP placement
```

---

# 10. Final Acceptance

```text
MTP CONFIGURATION:
VERIFIED

MTP CHECKPOINT TENSORS:
VERIFIED

MTP EXACT TENSOR METADATA:
VERIFIED

MTP CHECKPOINT TENSOR COUNT:
15

MTP DTYPE:
BF16

MTP SHARD:
model-00018-of-00018.safetensors

MTP ACTIVE RUNTIME EXECUTION:
UNKNOWN
```
