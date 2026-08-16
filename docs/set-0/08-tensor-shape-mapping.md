# SET 0 — Tensor Shape Mapping

## Document Status

* Document: `08-tensor-shape-mapping.md`
* SET: `SET 0 — Model Reconnaissance`
* Source Task: `SET0-T15`, reconciled by `SET0-CLOSE-FIX-FINALIZE`
* Status: **VERIFIED**
* Responsibility: 🧠 LUNA

---

# 1. Source and Provenance

Model: `Qwen3.8-27B`

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary tensor metadata:

```text
model/official/TENSOR-METADATA.md
```

Verified tensor index:

```text
model/official/model.safetensors.index.json
```

This document distinguishes exact raw-metadata facts from derived runtime
interpretations.

---

# 2. Major Verified Tensor Families

## Language MLP

| Tensor | Shape | Coverage | Classification |
|---|---:|---:|---|
| `mlp.gate_proj.weight` | `[17408, 5120]` | 64/64 | VERIFIED FACT |
| `mlp.up_proj.weight` | `[17408, 5120]` | 64/64 | VERIFIED FACT |
| `mlp.down_proj.weight` | `[5120, 17408]` | 64/64 | VERIFIED FACT |

## Full Attention

| Tensor | Shape | Coverage | Classification |
|---|---:|---:|---|
| `self_attn.q_proj.weight` | `[12288, 5120]` | 16/16 | VERIFIED FACT |
| `self_attn.k_proj.weight` | `[1024, 5120]` | 16/16 | VERIFIED FACT |
| `self_attn.v_proj.weight` | `[1024, 5120]` | 16/16 | VERIFIED FACT |
| `self_attn.o_proj.weight` | `[5120, 6144]` | 16/16 | VERIFIED FACT |
| `self_attn.q_norm.weight` | `[256]` | 16/16 | VERIFIED FACT |
| `self_attn.k_norm.weight` | `[256]` | 16/16 | VERIFIED FACT |

## Linear Attention

| Tensor | Shape | Coverage | Classification |
|---|---:|---:|---|
| `linear_attn.in_proj_qkv.weight` | `[10240, 5120]` | 48/48 | VERIFIED FACT |
| `linear_attn.in_proj_z.weight` | `[6144, 5120]` | 48/48 | VERIFIED FACT |
| `linear_attn.in_proj_b.weight` | `[48, 5120]` | 48/48 | VERIFIED FACT |
| `linear_attn.in_proj_a.weight` | `[48, 5120]` | 48/48 | VERIFIED FACT |
| `linear_attn.out_proj.weight` | `[5120, 6144]` | 48/48 | VERIFIED FACT |
| `linear_attn.conv1d.weight` | `[10240, 1, 4]` | 48/48 | VERIFIED FACT |
| `linear_attn.A_log` | `[48]` | 48/48 | VERIFIED FACT |
| `linear_attn.dt_bias` | `[48]` | 48/48 | VERIFIED FACT |
| `linear_attn.norm.weight` | `[128]` | 48/48 | VERIFIED FACT |

## Vision

The major verified vision tensor families include patch embedding,
positional embedding, 27 visual blocks, visual attention, visual MLP,
normalization, and merger tensors.

---

# 3. MTP Tensor Shapes — VERIFIED FACT

The exact MTP tensor metadata is directly supported by
`model/official/TENSOR-METADATA.md`. The previous **DERIVED FINDING**
classification is superseded.

| Tensor | Exact Shape | Coverage | Classification |
|---|---:|---:|---|
| `mtp.fc.weight` | `[5120, 10240]` | global | VERIFIED FACT |
| `mtp.layers.0.input_layernorm.weight` | `[5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.mlp.down_proj.weight` | `[5120, 17408]` | 1 | VERIFIED FACT |
| `mtp.layers.0.mlp.gate_proj.weight` | `[17408, 5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.mlp.up_proj.weight` | `[17408, 5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.post_attention_layernorm.weight` | `[5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.k_norm.weight` | `[256]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.k_proj.weight` | `[1024, 5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.o_proj.weight` | `[5120, 6144]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.q_norm.weight` | `[256]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.q_proj.weight` | `[12288, 5120]` | 1 | VERIFIED FACT |
| `mtp.layers.0.self_attn.v_proj.weight` | `[1024, 5120]` | 1 | VERIFIED FACT |
| `mtp.norm.weight` | `[5120]` | global | VERIFIED FACT |
| `mtp.pre_fc_norm_embedding.weight` | `[5120]` | global | VERIFIED FACT |
| `mtp.pre_fc_norm_hidden.weight` | `[5120]` | global | VERIFIED FACT |

Raw metadata also establishes:

```text
MTP tensor count = 15
MTP dtype = BF16
MTP shard = model-00018-of-00018.safetensors
```

### Evidence Classification

```text
VERIFIED FACT
```

for every exact tensor name, shape, dtype, and shard mapping above.

---

# 4. MTP Parameterization Boundary

The tensor shapes above are raw checkpoint facts. Parameter arithmetic and
byte accounting are recorded in `09-parameter-byte-accounting.md`.

Runtime semantics are intentionally separate:

```text
MTP active runtime execution = UNKNOWN
```

Checkpoint tensor metadata must not be used to infer that ordinary generation
necessarily executes MTP.

---

# 5. Historical Classification Correction

The former MTP section classified exact MTP shapes as:

```text
DERIVED FINDING
```

because direct tensor metadata had not yet been incorporated into the
canonical mapping.

That classification is now **HISTORICAL / SUPERSEDED**.

The current canonical classification for the raw-backed MTP tensor map is:

```text
VERIFIED FACT
```

---

# 6. Evidence Classification Summary

## VERIFIED FACT

```text
Raw-backed language tensor shapes
Raw-backed attention tensor shapes
Raw-backed vision tensor shapes
All 15 exact MTP tensor names and shapes
MTP dtype = BF16
MTP shard = model-00018-of-00018.safetensors
MTP tensor count = 15
```

## DERIVED FINDING

```text
Runtime-state dimensions and other arithmetic relationships not directly
represented as raw tensor metadata.
```

## UNKNOWN

```text
MTP active runtime execution semantics
MTP runtime scheduling
MTP runtime memory behavior
Hardware-specific placement
```

---

# 7. Final Acceptance

```text
TENSOR SHAPE / DIMENSION MAPPING:
PASS

MTP TENSOR COUNT:
15

MTP EXACT TENSOR METADATA:
VERIFIED FACT

MTP DTYPE:
BF16

MTP SHARD:
model-00018-of-00018.safetensors

MTP RUNTIME EXECUTION:
UNKNOWN
```
