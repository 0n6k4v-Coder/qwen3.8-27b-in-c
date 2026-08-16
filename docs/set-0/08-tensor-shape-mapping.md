# SET 0 — Tensor Shape Mapping

## Document Status

* Document: `08-tensor-shape-mapping.md`
* SET: `SET 0 — Model Reconnaissance`
* Source Task: `SET0-T15`
* Status: VERIFIED
* Purpose: Canonical record of the verified and derived tensor-shape mapping of Qwen3.8-27B
* Responsibility: 🧠 LUNA

---

# 1. Source and Provenance

Model:

```text
Qwen3.8-27B
```

Official repository:

```text
Qwen/Qwen3.8-27B
```

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary configuration artifact:

```text
model/official/config.json
```

Verified tensor index:

```text
model/official/model.safetensors.index.json
```

Related research checkpoints:

```text
docs/set-0/01-artifact-provenance.md
docs/set-0/02-model-identity.md
docs/set-0/04-core-architecture.md
docs/set-0/05-attention-architecture.md
docs/set-0/06-mlp-architecture.md
docs/set-0/07-vision-and-mtp.md
docs/set-0/08-layer-topology.md
```

The configuration artifact and tensor index were previously verified against
the official artifact at the pinned revision.

---

# 2. Scope

This document records the tensor-shape and dimension mapping established by
`SET0-T15`.

It establishes the bridge:

```text
CONFIGURATION
      ↓
ARCHITECTURE
      ↓
TENSOR FAMILY
      ↓
TENSOR SHAPE
```

This document does not establish:

```text
parameter count
tensor byte totals
checkpoint memory footprint
runtime memory
hardware placement
benchmark performance
```

Those are deferred to subsequent tasks.

---

# 3. Global Language Tensor Shapes

| Tensor                                     |            Shape | Classification |
| ------------------------------------------ | ---------------: | -------------- |
| `model.language_model.embed_tokens.weight` | `[248320, 5120]` | VERIFIED FACT  |
| `model.language_model.norm.weight`         |         `[5120]` | VERIFIED FACT  |
| `lm_head.weight`                           | `[248320, 5120]` | VERIFIED FACT  |

Configuration basis:

```text
vocab_size = 248320
hidden_size = 5120
tie_word_embeddings = false
```

---

# 4. Language MLP Tensor Shapes

The language model uses the same MLP tensor family across all 64 language
layers.

| Tensor Family          |           Shape | Coverage | Classification |
| ---------------------- | --------------: | -------: | -------------- |
| `mlp.gate_proj.weight` | `[17408, 5120]` |    64/64 | VERIFIED FACT  |
| `mlp.up_proj.weight`   | `[17408, 5120]` |    64/64 | VERIFIED FACT  |
| `mlp.down_proj.weight` | `[5120, 17408]` |    64/64 | VERIFIED FACT  |

Configuration basis:

```text
hidden_size = 5120
intermediate_size = 17408
hidden_act = silu
```

Dimensional structure:

```text
5120
  ↓
17408
  ↓
5120
```

No parameter-count calculation is performed in this document.

---

# 5. Full-Attention Tensor Shapes

Full-attention layers occur in 16 language layers.

Verified configuration:

```text
num_attention_heads = 24
num_key_value_heads = 4
head_dim = 256
```

Therefore:

```text
Q dimension:
24 × 256 = 6144

KV dimension:
4 × 256 = 1024
```

The Qwen3.5-derived full-attention implementation includes the output gate in
the Q projection, therefore the stored Q projection has double the normal
query projection width.

| Tensor Family             |           Shape | Coverage | Classification |
| ------------------------- | --------------: | -------: | -------------- |
| `self_attn.q_proj.weight` | `[12288, 5120]` |    16/16 | VERIFIED FACT  |
| `self_attn.k_proj.weight` |  `[1024, 5120]` |    16/16 | VERIFIED FACT  |
| `self_attn.v_proj.weight` |  `[1024, 5120]` |    16/16 | VERIFIED FACT  |
| `self_attn.o_proj.weight` |  `[5120, 6144]` |    16/16 | VERIFIED FACT  |
| `self_attn.q_norm.weight` |         `[256]` |    16/16 | VERIFIED FACT  |
| `self_attn.k_norm.weight` |         `[256]` |    16/16 | VERIFIED FACT  |

### GQA Relationship

```text
24 Q heads / 4 KV heads = 6
```

Therefore:

```text
6 query heads per KV head
```

This is consistent with the tensor dimensions.

### Important Finding

The Q projection is:

```text
[12288, 5120]
```

rather than:

```text
[6144, 5120]
```

because the implementation combines:

```text
query representation
+
output gate representation
```

within the Q projection.

---

# 6. Linear-Attention Tensor Shapes

The 48 linear-attention layers use the
`Qwen3_5GatedDeltaNet` tensor family.

Verified configuration:

```text
linear_num_key_heads = 16
linear_num_value_heads = 48

linear_key_head_dim = 128
linear_value_head_dim = 128
```

Therefore:

```text
key_dim:
16 × 128 = 2048

value_dim:
48 × 128 = 6144
```

The fused QKV projection therefore has:

```text
2048 + 2048 + 6144
= 10240
```

| Tensor Family                    |           Shape | Coverage | Classification |
| -------------------------------- | --------------: | -------: | -------------- |
| `linear_attn.in_proj_qkv.weight` | `[10240, 5120]` |    48/48 | VERIFIED FACT  |
| `linear_attn.in_proj_z.weight`   |  `[6144, 5120]` |    48/48 | VERIFIED FACT  |
| `linear_attn.in_proj_b.weight`   |    `[48, 5120]` |    48/48 | VERIFIED FACT  |
| `linear_attn.in_proj_a.weight`   |    `[48, 5120]` |    48/48 | VERIFIED FACT  |
| `linear_attn.out_proj.weight`    |  `[5120, 6144]` |    48/48 | VERIFIED FACT  |
| `linear_attn.conv1d.weight`      | `[10240, 1, 4]` |    48/48 | VERIFIED FACT  |
| `linear_attn.A_log`              |          `[48]` |    48/48 | VERIFIED FACT  |
| `linear_attn.dt_bias`            |          `[48]` |    48/48 | VERIFIED FACT  |
| `linear_attn.norm.weight`        |         `[128]` |    48/48 | VERIFIED FACT  |

### Linear-Attention Structural Summary

```text
Key dimension:
2048

Value dimension:
6144

Fused QKV projection:
10240

Z projection:
6144

Output projection:
6144 → 5120

Convolution:
10240 channels
kernel size = 4
```

The linear-attention family is therefore fundamentally different from
ordinary full attention.

It must not be represented as a conventional Q/K/V attention layer.

---

# 7. Linear-Attention Recurrent State Dimension

The implementation establishes a recurrent state structure:

```text
[num_v_heads, head_k_dim, head_v_dim]
=
[48, 128, 128]
```

This is a runtime-state dimension, not a weight-tensor shape.

It is recorded here because it is directly relevant to later operator and
runtime analysis.

### Classification

```text
DERIVED FINDING
```

---

# 8. Language Normalization Tensor Shapes

| Tensor Family                      |    Shape | Coverage | Classification |
| ---------------------------------- | -------: | -------: | -------------- |
| `input_layernorm.weight`           | `[5120]` |    64/64 | VERIFIED FACT  |
| `post_attention_layernorm.weight`  | `[5120]` |    64/64 | VERIFIED FACT  |
| `self_attn.q_norm.weight`          |  `[256]` |    16/16 | VERIFIED FACT  |
| `self_attn.k_norm.weight`          |  `[256]` |    16/16 | VERIFIED FACT  |
| `linear_attn.norm.weight`          |  `[128]` |    48/48 | VERIFIED FACT  |
| `model.language_model.norm.weight` | `[5120]` |   global | VERIFIED FACT  |

---

# 9. Vision Tensor Shapes

The verified vision subsystem contains 27 visual blocks.

Major learned tensor families:

| Tensor Family                           |                  Shape | Coverage | Classification |
| --------------------------------------- | ---------------------: | -------: | -------------- |
| `model.visual.patch_embed.proj.weight`  | `[1152, 3, 2, 16, 16]` |   global | VERIFIED FACT  |
| `model.visual.patch_embed.proj.bias`    |               `[1152]` |   global | VERIFIED FACT  |
| `model.visual.pos_embed.weight`         |         `[2304, 1152]` |   global | VERIFIED FACT  |
| `visual.blocks.N.attn.qkv.weight`       |         `[3456, 1152]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.attn.qkv.bias`         |               `[3456]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.attn.proj.weight`      |         `[1152, 1152]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.attn.proj.bias`        |               `[1152]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.mlp.linear_fc1.weight` |         `[4304, 1152]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.mlp.linear_fc2.weight` |         `[1152, 4304]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.norm1.weight`          |               `[1152]` |    27/27 | VERIFIED FACT  |
| `visual.blocks.N.norm2.weight`          |               `[1152]` |    27/27 | VERIFIED FACT  |
| `visual.merger.linear_fc1.weight`       |         `[4608, 4608]` |   global | VERIFIED FACT  |
| `visual.merger.linear_fc2.weight`       |         `[5120, 4608]` |   global | VERIFIED FACT  |
| `visual.merger.norm.weight`             |               `[1152]` |   global | VERIFIED FACT  |

Vision dimension relationships:

```text
vision hidden:
1152

vision heads:
16

vision head dimension:
1152 / 16 = 72

vision intermediate:
4304

vision merger input:
1152 × 2 × 2 = 4608

vision merger output:
5120
```

---

# 10. MTP Tensor Shapes

The tensor index verifies that MTP tensors exist.

The exact MTP shape map is currently supported through implementation
evidence rather than direct tensor metadata.

Therefore the following are classified as **DERIVED FINDINGS**.

| Tensor Family                          |           Shape | Coverage | Classification  |
| -------------------------------------- | --------------: | -------- | --------------- |
| `mtp.fc.weight`                        | `[5120, 10240]` | global   | DERIVED FINDING |
| `mtp.layers.0.self_attn.q_proj.weight` | `[12288, 5120]` | 1        | DERIVED FINDING |
| `mtp.layers.0.self_attn.k_proj.weight` |  `[1024, 5120]` | 1        | DERIVED FINDING |
| `mtp.layers.0.self_attn.v_proj.weight` |  `[1024, 5120]` | 1        | DERIVED FINDING |
| `mtp.layers.0.self_attn.o_proj.weight` |  `[5120, 6144]` | 1        | DERIVED FINDING |
| `mtp.layers.0.self_attn.q_norm.weight` |         `[256]` | 1        | DERIVED FINDING |
| `mtp.layers.0.self_attn.k_norm.weight` |         `[256]` | 1        | DERIVED FINDING |
| `mtp.layers.0.mlp.gate_proj.weight`    | `[17408, 5120]` | 1        | DERIVED FINDING |
| `mtp.layers.0.mlp.up_proj.weight`      | `[17408, 5120]` | 1        | DERIVED FINDING |
| `mtp.layers.0.mlp.down_proj.weight`    | `[5120, 17408]` | 1        | DERIVED FINDING |
| `mtp.pre_fc_norm_embedding.weight`     |        `[5120]` | global   | DERIVED FINDING |
| `mtp.pre_fc_norm_hidden.weight`        |        `[5120]` | global   | DERIVED FINDING |
| `mtp.norm.weight`                      |        `[5120]` | global   | DERIVED FINDING |

The MTP namespace itself is independently verified in the checkpoint:

```text
15 indexed MTP tensors
```

but these shape values remain implementation-derived until direct tensor
metadata confirms them.

---

# 11. Configuration Cross-Check

## Language MLP

```text
hidden_size = 5120
intermediate_size = 17408

5120 → 17408 → 5120
```

Consistent.

## Full Attention

```text
Q:
24 × 256 = 6144

KV:
4 × 256 = 1024

Q projection:
6144 × 2 = 12288
```

Consistent with the gated full-attention implementation.

## Linear Attention

```text
K:
16 × 128 = 2048

V:
48 × 128 = 6144

Fused QKV:
2048 + 2048 + 6144 = 10240
```

Consistent.

## Vision

```text
hidden = 1152
heads = 16
intermediate = 4304
depth = 27
output = 5120
```

Consistent with the learned parameter shapes.

## MTP

```text
mtp_num_hidden_layers = 1
```

corresponds structurally to:

```text
mtp.layers.0.*
```

but exact runtime semantics remain unresolved.

---

# 12. Uniformity

The verified language tensor structure is uniform by architectural family:

```text
64 language layers
├── 48 × linear attention
├── 16 × full attention
├── 64 × MLP
├── 64 × input LayerNorm
└── 64 × post-attention LayerNorm
```

Each attention family uses a consistent tensor shape pattern across its
respective layers.

The vision tensor families are likewise consistent across the 27 visual
blocks.

MTP contains one configured layer.

---

# 13. Important Findings

## Finding A — Full Attention Q Projection Includes Output Gate

The stored tensor is:

```text
[12288, 5120]
```

not:

```text
[6144, 5120]
```

The extra factor of 2 corresponds to the query plus output-gate path.

This must be preserved during later inference-engine design.

---

## Finding B — Linear Attention Is Structurally Non-Standard

The principal projection is:

```text
[10240, 5120]
```

with:

```text
K dimension = 2048
V dimension = 6144
```

and a separate Z projection:

```text
[6144, 5120]
```

This confirms that the linear-attention mechanism should not be modeled as
ordinary multi-head Q/K/V attention.

---

## Finding C — Linear-Attention Runtime State Is Separate From Weight Storage

The recurrent state is:

```text
[48, 128, 128]
```

This is not represented by an ordinary KV-cache tensor family.

---

## Finding D — Vision Merger Expands Before Language Projection

The learned vision merger has:

```text
1152 × 2 × 2 = 4608
```

input width and produces:

```text
5120
```

output width.

This matches the language hidden dimension.

---

## Finding E — MTP Shape Evidence Has Lower Confidence

MTP tensors definitely exist.

Their current shape mapping is derived from implementation evidence rather
than direct tensor metadata.

This distinction must be retained during byte accounting.

---

# 14. Evidence Classification Summary

## VERIFIED FACT

```text
Global language tensor shapes
Language MLP shapes
Full-attention tensor shapes
Linear-attention tensor shapes
Language normalization shapes
Major vision tensor shapes
MTP tensor presence
```

## DERIVED FINDING

```text
Linear-attention recurrent state [48,128,128]
MTP tensor shapes based on implementation evidence
Vision head dimension = 72
Vision merger input = 4608
```

## INFERENCE

```text
Linear-attention state cannot be represented as ordinary KV-cache
Full-attention q_proj includes query + output gate
```

## UNKNOWN

```text
Exact per-tensor storage dtype
Exact storage representation of certain state parameters
Direct artifact-level confirmation of every MTP shape
Checkpoint shard byte contribution per tensor family
Exact parameter counts
```

---

# 15. Families Ready for Byte Accounting

The following major families are now sufficiently characterized for
parameter and byte accounting:

```text
✅ Global language tensors
✅ Language MLP
✅ Full attention
✅ Linear attention
✅ Language normalization
✅ Vision
⚠ MTP — shape mapping is implementation-derived
```

MTP should be separately labelled in the next accounting document rather
than silently merged with the confidence level of the main language
families.

---

# 16. Research Boundary

This document does not establish:

```text
parameter count
tensor byte totals
checkpoint shard sizes by family
runtime memory footprint
KV-cache memory
linear-attention recurrent-state memory budget
vision runtime memory
MTP runtime memory
CPU/GPU/NPU placement
```

These belong to subsequent analysis.

---

# 17. Canonical Tensor Shape Statement

> **The verified Qwen3.8-27B artifact maps its configuration to a
> structurally explicit tensor layout. The 64-layer language stack uses
> 48 Gated DeltaNet layers and 16 gated full-attention layers. The MLP is
> parameterized as 5120→17408→5120. Full attention uses 24 query heads and
> 4 KV heads with head dimension 256, while its stored Q projection is
> 12288-wide because the implementation combines query and output-gate
> representations. Linear attention uses 16 key heads and 48 value heads,
> producing 2048 key width, 6144 value width, a 10240-wide fused QKV
> projection, and a 6144-wide output path. The vision subsystem uses
> 1152-dimensional representations and projects them to the 5120-wide
> language representation. MTP tensors are present in the checkpoint, but
> their current detailed shapes remain implementation-derived until direct
> tensor metadata confirms them.**

---

# 18. Final Acceptance

```text
TENSOR SHAPE / DIMENSION MAPPING:
PASS

MAJOR LANGUAGE TENSOR FAMILIES:
VERIFIED

FULL ATTENTION:
VERIFIED

LINEAR ATTENTION:
VERIFIED

MLP:
VERIFIED

VISION:
VERIFIED

MTP:
DERIVED / REQUIRES DIRECT CONFIRMATION

READY FOR BYTE ACCOUNTING:
YES
```