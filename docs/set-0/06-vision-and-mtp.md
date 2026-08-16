# SET 0 — Vision and MTP Architecture

## Document Status

* Document: `06-vision-and-mtp.md`
* SET: `SET 0 — Model Reconnaissance`
* Source Tasks: `SET0-T09`, updated by `SET0-T14`
* Status: VERIFIED
* Purpose: Canonical record of the Vision subsystem and MTP-related
  configuration and checkpoint evidence of Qwen3.8-27B
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
docs/set-0/08-layer-topology.md
```

The primary configuration artifact and tensor index have been verified
against the official artifact at the pinned revision.

---

# 2. Vision Architecture Summary

Qwen3.8-27B is explicitly configured as a multimodal model.

The configuration contains:

```text
language_model_only = false
vision_config = present
```

The vision subsystem is configured with:

```text
depth:
27

hidden_size:
1152

num_heads:
16

intermediate_size:
4304

out_hidden_size:
5120
```

Input/patch configuration:

```text
in_channels:
3

patch_size:
16

spatial_merge_size:
2

temporal_patch_size:
2
```

Vision positional configuration:

```text
num_position_embeddings:
2304
```

Activation:

```text
gelu_pytorch_tanh
```

---

# 3. Vision Depth

Verified configuration:

```text
vision_config.depth = 27
```

Tensor-index evidence independently contains:

```text
model.visual.blocks.0
...
model.visual.blocks.26
```

Therefore the vision subsystem is configured with and stores:

```text
27 vision blocks/layers
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 4. Vision Hidden Dimension

Verified configuration:

```text
vision_config.hidden_size = 1152
```

Therefore the internal vision representation width is:

```text
1152
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 5. Vision Attention Heads

Verified configuration:

```text
vision_config.num_heads = 16
```

Therefore the vision subsystem is configured with:

```text
16 attention heads
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 6. Vision Intermediate Dimension

Verified configuration:

```text
vision_config.intermediate_size = 4304
```

Therefore the configured vision feed-forward intermediate width is:

```text
4304
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 7. Vision Input Configuration

The artifact specifies:

```text
in_channels = 3
```

The patch parameters are:

```text
patch_size = 16

spatial_merge_size = 2

temporal_patch_size = 2
```

These values configure the conversion of visual input into the vision
representation.

The tensor index confirms a dedicated patch embedding:

```text
model.visual.patch_embed.proj.weight
model.visual.patch_embed.proj.bias
```

### Important Boundary

These values alone do not establish a complete patch-count formula for
arbitrary image/video dimensions. Exact token counts depend on the actual
input dimensions and implementation path.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 8. Vision Positional Configuration

The vision configuration specifies:

```text
num_position_embeddings = 2304
```

The tensor index contains:

```text
model.visual.pos_embed.weight
```

The associated implementation uses dedicated vision positional processing.

### Evidence Classification

```text
VERIFIED FACT
```

The exact complete positional tensor interpretation remains deferred to
tensor-level shape analysis.

---

# 9. Vision Output Interface

The vision configuration specifies:

```text
vision_config.out_hidden_size = 5120
```

The language configuration specifies:

```text
text_config.hidden_size = 5120
```

Therefore:

```text
vision output width
=
language hidden width
=
5120
```

### Derived Finding

The vision subsystem is configured to produce representations whose width
matches the language model hidden representation.

### Evidence Classification

```text
DERIVED FINDING
```

---

# 10. Vision-to-Language Integration

The official implementation associated with the artifact's
`Qwen3_5ForConditionalGeneration` architecture identity establishes the
following conceptual path:

```text
Image / Video
      ↓
Vision Encoder
      ↓
Visual Features
      ↓
Special Image / Video Token Positions
      ↓
Language Input Embeddings
      ↓
Language Model
```

Visual features are inserted into the language embedding sequence at
corresponding image/video token positions.

Therefore the examined implementation path uses visual feature injection
into the language sequence rather than requiring a separately configured
cross-attention stack for visual consumption.

### Important Boundary

This statement describes the examined official implementation path.

It does not claim that every possible runtime or optimization backend must
implement multimodal execution identically.

### Evidence Classification

```text
VERIFIED FACT
```

at the implementation level.

---

# 11. Special Vision / Video Tokens

The verified artifact contains:

```text
image_token_id:
248056

video_token_id:
248057

vision_start_token_id:
248053

vision_end_token_id:
248054
```

These identifiers are used by the official implementation to identify
visual positions in the language input sequence.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 12. Vision Feature Injection

Conceptually, the multimodal input flow is:

```text
Text Tokens
    +
Image / Video Tokens
        ↓
Initial Language Embeddings
        +
Visual Features
        ↓
Multimodal Embedding Sequence
        ↓
64-Layer Language Model
```

The visual features therefore become part of the sequence processed by the
language model's existing attention and MLP stack.

This is important because the examined language architecture does not
require a separately configured cross-attention stack merely to consume
visual embeddings in this implementation path.

---

# 13. Multimodal Position Handling

The language configuration contains:

```text
mrope_interleaved = true

mrope_section = [11, 11, 10]
```

These fields participate in multimodal positional handling.

The implementation constructs multimodal position information for visual
inputs and integrates it into the language model's positional processing.

This connects:

```text
temporal
+
height
+
width
```

position information with the multimodal language sequence.

### Evidence Classification

```text
VERIFIED FACT
```

at the implementation/configuration level.

The exact tensor representation of these position IDs remains deferred.

---

# 14. Vision Architecture Boundary

The current evidence establishes:

```text
27 layers
1152 hidden
16 heads
4304 intermediate
3 input channels
patch_size = 16
spatial_merge_size = 2
temporal_patch_size = 2
output width = 5120
vision positional encoding
patch embedding
visual transformer blocks
visual merger
```

It does not yet establish:

```text
exact vision tensor shapes
exact vision parameter count
exact vision shard-byte contribution
exact runtime vision memory
exact optimized vision kernels
exact image/video preprocessing pipeline
exact complete visual feature layout
```

Those require later tensor-level and implementation-level verification.

---

# 15. MTP Configuration

The verified language configuration contains:

```text
mtp_num_hidden_layers = 1

mtp_use_dedicated_embeddings = false
```

Therefore the artifact contains explicit MTP-related configuration metadata.

### Configuration Finding

```text
MTP layer count:
1

Dedicated MTP embeddings:
false
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 16. MTP Checkpoint Presence

The verified tensor index independently contains an MTP parameter set under
the `mtp.*` namespace.

Verified tensors include:

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

MTP checkpoint presence:
VERIFIED

MTP checkpoint tensor count:
15
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 17. MTP Runtime Status

The presence of MTP configuration and checkpoint tensors establishes that
MTP parameters are present in the released checkpoint.

However, checkpoint presence alone does not establish that the current
official inference implementation executes the MTP path during ordinary
generation.

The examined implementation evidence therefore supports:

```text
MTP configuration:
VERIFIED

MTP checkpoint tensors:
VERIFIED

Active MTP execution path:
NOT YET ESTABLISHED
```

This distinction must be preserved.

### Evidence Classification

```text
VERIFIED FACT:
MTP configuration exists

VERIFIED FACT:
MTP checkpoint tensors exist

UNKNOWN:
Whether MTP is an active execution feature of the released runtime path
```

---

# 18. MTP Embedding Configuration

The artifact specifies:

```text
mtp_use_dedicated_embeddings = false
```

This is a verified configuration value.

Because the active MTP execution path has not yet been established, this
document does not infer a concrete runtime embedding-sharing mechanism
beyond the literal configuration field.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 19. MTP Boundary

The current evidence establishes:

```text
MTP configuration
MTP checkpoint tensors
MTP layer 0 tensor group
```

The following remain unresolved:

```text
MTP operator execution path
MTP runtime activation during generation
MTP scheduling behavior
MTP runtime memory behavior
MTP performance contribution
MTP optimized hardware placement
```

These require further official implementation/runtime verification.

---

# 20. Verified Facts

The following findings are established:

```text
✅ language_model_only = false

✅ vision_config is present

✅ vision depth = 27

✅ vision hidden size = 1152

✅ vision heads = 16

✅ vision intermediate size = 4304

✅ vision input channels = 3

✅ patch_size = 16

✅ spatial_merge_size = 2

✅ temporal_patch_size = 2

✅ num_position_embeddings = 2304

✅ vision activation = gelu_pytorch_tanh

✅ vision out_hidden_size = 5120

✅ language hidden_size = 5120

✅ image_token_id = 248056

✅ video_token_id = 248057

✅ vision_start_token_id = 248053

✅ vision_end_token_id = 248054

✅ mtp_num_hidden_layers = 1

✅ mtp_use_dedicated_embeddings = false
```

Implementation-level findings:

```text
✅ Dedicated vision model exists in the associated implementation

✅ Visual features are inserted into the language embedding sequence

✅ Image/video token positions are used for visual feature placement

✅ Multimodal positional information is explicitly handled
```

Checkpoint-level findings:

```text
✅ model.visual.* tensors exist

✅ 27 vision blocks exist in the tensor index

✅ model.visual.merger.* tensors exist

✅ model.visual.patch_embed.* tensors exist

✅ model.visual.pos_embed.* tensor exists

✅ mtp.* tensors exist

✅ 1 MTP layer tensor group exists

✅ MTP checkpoint tensor count = 15
```

Runtime-level finding:

```text
⚠ Active MTP execution path is not yet established
```

---

# 21. Derived Findings

## Vision / Language Width

```text
vision_config.out_hidden_size
=
5120

text_config.hidden_size
=
5120
```

Therefore:

```text
vision output width
=
language hidden width
```

---

## Vision Tensor-Configuration Correspondence

Configuration:

```text
vision depth = 27
```

Tensor index:

```text
model.visual.blocks.0
...
model.visual.blocks.26
```

Therefore the configured vision depth is structurally reflected in the
checkpoint.

---

## Multimodal Sequence Integration

The implementation establishes:

```text
Vision Features
      ↓
Language Embedding Sequence
      ↓
Language Decoder
```

Thus visual information enters the language model through the embedding
sequence in the examined implementation path.

---

## MTP Configuration / Checkpoint Correspondence

Configuration:

```text
mtp_num_hidden_layers = 1
```

Tensor index:

```text
mtp.layers.0.*
```

Therefore the configured one-layer MTP structure has corresponding
checkpoint tensors.

---

# 22. Inferences

### I1 — Vision is upstream of the language decoder

Visual feature extraction takes place before the 64-layer language model
processes the multimodal sequence.

### I2 — Vision and language share a dimensional interface

The matching 5120-dimensional output/input representation simplifies the
interface between the vision and language subsystems.

### I3 — Multimodal inference requires additional state and compute

A multimodal inference engine will need to account for:

```text
vision preprocessing
vision execution
visual feature storage
token-to-feature placement
multimodal position information
language inference
```

### I4 — MTP should remain conditional in the runtime design

MTP parameters are present in the checkpoint, but until its execution path
is verified, MTP should not be assumed to be a mandatory component of the
ordinary generation path.

---

# 23. Unknown / Further Verification

## Vision

```text
UQ-VIS-001
Exact tensor shapes for all vision layers.

UQ-VIS-002
Exact vision parameter count.

UQ-VIS-003
Exact vision shard-byte contribution.

UQ-VIS-004
Exact visual feature tensor layout before language-sequence insertion.

UQ-VIS-005
Exact image/video feature-to-token mapping at tensor level.

UQ-VIS-006
Exact runtime vision memory requirements.

UQ-VIS-007
Exact optimized vision kernels for the target hardware.
```

## MTP

```text
UQ-MTP-001
Exact tensor shapes and parameterization of the MTP tensors.

UQ-MTP-002
Exact MTP forward/execution path in the released implementation.

UQ-MTP-003
Whether ordinary text generation invokes the MTP subnetwork.

UQ-MTP-004
Why MTP configuration and checkpoint parameters are present while the
examined runtime path does not yet establish active execution.

UQ-MTP-005
Exact MTP runtime memory and scheduling requirements.
```

---

# 24. Implications for Later Research

## SET 1 — Tensor / Byte-Level Audit

The tensor audit must separately account for:

```text
Vision tensors
MTP tensors
Image/video embedding-related tensors
Multimodal projection/merger tensors
```

MTP must no longer be treated as configuration-only.

## SET 3 — Operator / Computation Model

The operator model will need to represent:

```text
Vision Encoder
Visual Feature Projection / Formatting
Visual Token Injection
Multimodal Position Handling
Language Model
```

MTP should remain conditional until its active execution path is verified.

## SET 4 — Runtime Memory Model

The runtime memory model will eventually need to distinguish:

```text
Vision Working Set
Language Model Working Set
Visual Feature Buffers
Multimodal Position State
MTP Working Set
```

with MTP activated only if the verified execution path requires it.

---

# 25. Architecture Diagram

```text
                         Qwen3.8-27B
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
          Vision Subsystem             Text / Language
                │                           │
           27 layers                    Token Embeddings
                │                           │
        1152 hidden                       5120
                │                           │
        Visual Features                   │
                │                         │
                └────────────┐            │
                             ▼            │
                   Image / Video Token Slots
                             │
                             ▼
                  Multimodal Input Sequence
                             │
                             ▼
                    64-Layer Language Model
                             │
                             ▼
                           Output


MTP:
Configuration + checkpoint parameters verified
Active runtime execution path not yet established
```

---

# 26. Canonical Vision Statement

> **Qwen3.8-27B contains a dedicated 27-layer vision subsystem with a
> 1152-dimensional internal representation, 16 attention heads, 4304
> intermediate width, 16×16 spatial patch configuration, temporal patch
> size 2, spatial merge factor 2, and 5120-dimensional output. The visual
> representation is integrated into the language embedding sequence at
> image/video token positions before processing by the 64-layer language
> model. The checkpoint independently contains the corresponding visual
> tensor structure, including 27 visual blocks, patch embedding,
> positional embedding, and visual merger tensors.**

---

# 27. Canonical MTP Statement

> **The Qwen3.8-27B artifact declares one MTP hidden layer and disables
> dedicated MTP embeddings. The verified Safetensors tensor index
> independently confirms the presence of a 15-tensor MTP parameter set,
> including `mtp.layers.0.*`, `mtp.fc.weight`, normalization tensors, and
> MTP embedding-normalization tensors. Therefore MTP is definitively
> present in the released checkpoint. However, the active MTP execution
> path in the released inference implementation has not yet been
> established and must remain an unresolved runtime question.**

---

# 28. Research Boundary

This document establishes the Vision and MTP architecture visible from the
verified configuration, tensor index, and associated official
implementation evidence.

It does NOT establish:

```text
- complete vision tensor shapes
- complete MTP tensor shapes
- vision parameter count
- MTP parameter count
- exact runtime memory
- optimized hardware kernels
- CPU/GPU/NPU placement
- multimodal performance characteristics
- active MTP execution during ordinary generation
```

Those require separate verification.

---

# 29. Final Acceptance

```text
VISION ARCHITECTURE STATUS:
VERIFIED

VISION DEPTH:
27

VISION HIDDEN SIZE:
1152

VISION OUTPUT SIZE:
5120

VISION → LANGUAGE INTERFACE:
VISUAL EMBEDDING INJECTION INTO LANGUAGE SEQUENCE

MTP CONFIGURATION:
1 LAYER

MTP CHECKPOINT PRESENCE:
VERIFIED

MTP CHECKPOINT TENSORS:
15

MTP ACTIVE RUNTIME PATH:
NOT YET ESTABLISHED

VISION + MTP ANALYSIS:
PASS
```

This document is the canonical `SET0-T09` Vision + MTP architecture
checkpoint, reconciled with the later checkpoint-level evidence from
`SET0-T14`.