# SET 0 — Core Architecture

## Document Status

- Document: `04-core-architecture.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T06`
- Status: VERIFIED
- Purpose: Canonical record of the Qwen3.8-27B core architecture derived
  from the verified Qwen3.8-27B configuration artifact

---

## 1. Source

Model:

```text
Qwen3.8-27B
````

Official repository:

```text
Qwen/Qwen3.8-27B
```

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary source:

```text
model/official/config.json
```

The local `config.json` has already been verified byte-for-byte against the
official file at the pinned revision.

Related provenance document:

```text
docs/set-0/01-artifact-provenance.md
```

Related identity document:

```text
docs/set-0/02-model-identity.md
```

---

# 2. Executive Architecture Summary

Qwen3.8-27B is a multimodal conditional-generation model with:

```text
Language Model
    ↓
64-layer language stack
    ↓
Hybrid linear-attention + full-attention architecture
    ↓
48 linear-attention layers
16 full-attention layers
    ↓
Repeated 3:1 pattern

Vision Subsystem
    ↓
Dedicated 27-layer vision configuration
    ↓
Output dimension = 5120

MTP
    ↓
1 configured MTP layer
```

The verified configuration therefore establishes a **multimodal hybrid
attention architecture**.

The term "hybrid" in this document specifically refers to the coexistence
of:

```text
linear_attention
+
full_attention
```

It does NOT mean that the model has been established as "Mamba + Attention"
or another specific hybrid architecture unless additional official
evidence establishes that mechanism.

---

# 3. Model Modality

The model configuration is explicitly multimodal.

Verified fields:

```text
language_model_only = false
image_token_id = 248056
video_token_id = 248057
vision_start_token_id = 248053
vision_end_token_id = 248054
vision_config = present
```

### Finding

The artifact contains both:

```text
language-model configuration
+
vision configuration
```

Therefore the model is not represented as a language-only model.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 4. Language Model Core

The language configuration specifies:

```text
hidden_size:
5120

num_hidden_layers:
64

dtype:
bfloat16

vocab_size:
248320

max_position_embeddings:
262144
```

Therefore the language-side hidden representation width is:

```text
5120
```

and the language stack contains:

```text
64 layers
```

The configured language-side numeric type is:

```text
bfloat16
```

### Evidence

```text
text_config.hidden_size
text_config.num_hidden_layers
text_config.dtype
text_config.vocab_size
text_config.max_position_embeddings
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 5. Language-Layer Composition

The configuration contains a 64-element:

```text
text_config.layer_types
```

sequence containing only:

```text
linear_attention
full_attention
```

The pattern is:

```text
linear_attention
linear_attention
linear_attention
full_attention
```

repeated across the language stack.

Compact form:

```text
[LA → LA → LA → FA] × 16
```

where:

```text
LA = linear_attention
FA = full_attention
```

Because:

```text
64 / 4 = 16
```

the resulting layer counts are:

```text
48 linear-attention layers
16 full-attention layers
```

### Evidence Classification

```text
VERIFIED FACT:
- 64 total language layers
- 48 linear-attention layers
- 16 full-attention layers

DERIVED FACT:
- 3:1 linear/full layer ratio
- repeating 4-layer cadence
```

---

# 6. Full-Attention Configuration

The full-attention configuration contains:

```text
num_attention_heads:
24

num_key_value_heads:
4

head_dim:
256
```

This establishes a grouped query/key-value structure.

The relationship is:

```text
24 query heads
4 key/value heads
```

Therefore:

```text
24 / 4 = 6
```

query heads correspond to each key/value head.

### Finding

The full-attention layers use a:

```text
Grouped-Query Attention (GQA)
```

organization.

### Evidence Classification

```text
VERIFIED FACT:
24 attention heads
4 KV heads
head dimension = 256

DERIVED FINDING:
6 query heads per KV head
```

The exact internal implementation of the full-attention operator remains
outside the scope of this document.

---

# 7. Linear-Attention Configuration

The verified configuration contains a separate linear-attention parameter
group:

```text
linear_key_head_dim:
128

linear_value_head_dim:
128

linear_num_key_heads:
16

linear_num_value_heads:
48

linear_conv_kernel_dim:
4
```

This establishes that the `linear_attention` layers have their own
key/value head organization distinct from the full-attention configuration.

Configured dimensions:

```text
key head dimension:
128

value head dimension:
128

key heads:
16

value heads:
48

convolutional kernel dimension:
4
```

### Finding

The model contains a dedicated linear-attention mechanism with a
configuration distinct from the full-attention mechanism.

### Important Boundary

The current configuration does NOT, by itself, establish the exact
algorithm or kernel implementation used by the linear-attention operator.

In particular, the following must NOT be treated as verified merely from
these fields:

```text
Mamba
Mamba-2
Gated DeltaNet
DeltaNet
other specific linear-attention algorithm
```

The field:

```text
mamba_ssm_dtype = float32
```

is evidence that the configuration includes Mamba/SSM-related metadata,
but it is insufficient by itself to establish the exact operator used by
all `linear_attention` layers.

### Evidence Classification

```text
VERIFIED FACT:
linear_attention exists and has the above configuration.

UNKNOWN:
exact internal linear-attention algorithm and implementation semantics.
```

---

# 8. Attention Layer Placement

The field:

```text
full_attention_interval = 4
```

is consistent with the explicit layer sequence:

```text
[LA → LA → LA → FA] × 16
```

Thus full attention occurs every fourth language layer in the verified
topology.

The resulting structure is:

```text
L00  linear_attention
L01  linear_attention
L02  linear_attention
L03  full_attention

L04  linear_attention
L05  linear_attention
L06  linear_attention
L07  full_attention

...

L60  linear_attention
L61  linear_attention
L62  linear_attention
L63  full_attention
```

### Finding

The language model uses a regular interleaving of:

```text
3 linear-attention layers
followed by
1 full-attention layer
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 9. MLP / Feed-Forward Configuration

The language configuration specifies:

```text
intermediate_size:
17408

hidden_act:
silu
```

Therefore the language MLP/feed-forward configuration has:

```text
hidden size:
5120

intermediate size:
17408

activation:
SiLU
```

### Boundary

The configuration fields provided here establish the dimensions and
activation, but do not by themselves establish the exact internal MLP
formulation.

Therefore this document does not claim a specific gated-MLP structure
without additional implementation evidence.

### Evidence Classification

```text
VERIFIED FACT:
- intermediate size = 17408
- activation = SiLU

UNKNOWN:
exact internal MLP formulation
```

---

# 10. Positional Encoding

The configuration contains:

```text
rope_parameters:
{
    "mrope_interleaved": true,
    "mrope_section": [11, 11, 10],
    "partial_rotary_factor": 0.25,
    "rope_theta": 10000000,
    "rope_type": "default"
}
```

Additional top-level/text configuration:

```text
partial_rotary_factor:
0.25
```

Therefore the artifact explicitly configures a RoPE-based positional
mechanism with:

```text
rope_type:
default

rope_theta:
10000000

partial_rotary_factor:
0.25

mrope_interleaved:
true

mrope_section:
[11, 11, 10]
```

### Finding

The configuration establishes a multidimensional/interleaved RoPE
configuration.

### Boundary

The exact semantic interpretation of each MRoPE section and its complete
runtime implementation should be established from the corresponding
official implementation before implementation work begins.

### Evidence Classification

```text
VERIFIED FACT:
configured RoPE / MRoPE fields

PARTIALLY DEFERRED:
exact runtime positional implementation
```

---

# 11. Attention Output Gating

The language configuration contains:

```text
attn_output_gate:
true

output_gate_type:
swish
```

Therefore attention output gating is explicitly enabled, with:

```text
swish
```

specified as the output-gate type.

### Evidence Classification

```text
VERIFIED FACT
```

The exact location and tensor formulation of the gate remain implementation
details for later analysis.

---

# 12. Vision Subsystem

A dedicated:

```text
vision_config
```

is present.

Verified fields include:

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

in_channels:
3

patch_size:
16

spatial_merge_size:
2

temporal_patch_size:
2

num_position_embeddings:
2304

hidden_act:
gelu_pytorch_tanh
```

### Finding

The model contains a dedicated vision subsystem with:

```text
27 configured layers
hidden representation size = 1152
16 heads
intermediate size = 4304
output representation size = 5120
```

The value:

```text
out_hidden_size = 5120
```

matches the language model hidden size:

```text
text_config.hidden_size = 5120
```

This establishes compatible representation width at the configuration
level.

### Evidence Classification

```text
VERIFIED FACT:
vision configuration exists and contains the above dimensions.

DERIVED FINDING:
vision output width matches language hidden width.
```

The exact multimodal fusion mechanism is not established by this document.

---

# 13. MTP Subsystem

The configuration contains:

```text
mtp_num_hidden_layers:
1

mtp_use_dedicated_embeddings:
false
```

Therefore the artifact explicitly configures:

```text
1 MTP layer
```

and indicates that dedicated embeddings are not used for that MTP
configuration.

### Evidence Classification

```text
VERIFIED FACT
```

The exact MTP computation and integration path remain deferred.

---

# 14. Overall Architectural Model

The current verified architectural model is:

```text
                         Qwen3.8-27B
                              │
                 ┌────────────┴────────────┐
                 │                         │
           Vision Subsystem          Language Path
                 │                         │
            27 layers                      │
                 │                         │
          output width 5120                │
                 │                         │
                 └────────────┬────────────┘
                              │
                        64-Layer LM
                              │
             ┌────────────────┴────────────────┐
             │                                 │
      Linear Attention                    Full Attention
             │                                 │
      48 layers total                   16 layers total
             │                                 │
             └────────── 3 : 1 ───────────────┘
                              │
                         MLP / FFN
                              │
                           Output
                              │
                         1-layer MTP
```

Language-stack topology:

```text
[ Linear Attention
→ Linear Attention
→ Linear Attention
→ Full Attention ] × 16
```

---

# 15. Verified Architectural Facts

The following are considered established by SET0-T06:

```text
✅ Model identity = Qwen3.8-27B

✅ Multimodal configuration
✅ Language model hidden size = 5120
✅ Language layers = 64
✅ Language configuration dtype = BF16

✅ Linear attention is present
✅ Full attention is present
✅ Linear attention layers = 48
✅ Full attention layers = 16
✅ Layer ratio = 3:1
✅ Full-attention interval = 4

✅ Full-attention heads = 24
✅ Full-attention KV heads = 4
✅ Full-attention head dimension = 256
✅ GQA relationship = 24 Q heads / 4 KV heads

✅ Linear-attention key heads = 16
✅ Linear-attention value heads = 48
✅ Linear-attention key dimension = 128
✅ Linear-attention value dimension = 128
✅ Linear-attention convolutional kernel dimension = 4

✅ MLP intermediate size = 17408
✅ MLP activation = SiLU

✅ RoPE configuration is explicitly present
✅ rope_theta = 10000000
✅ partial_rotary_factor = 0.25
✅ mrope_interleaved = true
✅ mrope_section = [11, 11, 10]

✅ Attention output gating enabled
✅ output gate type = Swish

✅ Vision subsystem exists
✅ Vision depth = 27
✅ Vision hidden size = 1152
✅ Vision heads = 16
✅ Vision intermediate size = 4304
✅ Vision output size = 5120

✅ MTP layer count = 1
✅ MTP dedicated embeddings = false
```

---

# 16. Derived Findings

The following are directly derived from verified configuration:

```text
48 linear-attention layers
+
16 full-attention layers
=
64 language layers
```

Therefore:

```text
linear : full
=
48 : 16
=
3 : 1
```

The language-layer topology is:

```text
[LA → LA → LA → FA] × 16
```

Full attention uses:

```text
24 query heads
4 key/value heads
```

Therefore:

```text
6 query heads per KV head
```

Vision output width and language hidden width are both:

```text
5120
```

---

# 17. Inferences

The configuration supports the following architectural interpretation:

### 17.1 Hybrid attention language stack

The language model is best described as:

```text
hybrid linear-attention + full-attention
```

because both attention families explicitly exist in the layer topology.

### 17.2 Regular attention interleaving

Full attention is not used continuously across the entire stack. It is
periodically inserted after every three linear-attention layers.

### 17.3 Multimodal architecture

The model contains a dedicated vision subsystem whose configured output
width matches the language hidden width.

### 17.4 Qwen3.5-derived implementation lineage

The configuration uses:

```text
Qwen3_5ForConditionalGeneration
qwen3_5
qwen3_5_text
```

consistent with the already verified Qwen3.5 architectural foundation
finding from SET0-T03-R1.

This is a lineage finding, not a license to substitute Qwen3.5's complete
architecture for Qwen3.8.

---

# 18. Unknown / Requires Further Verification

The following remain intentionally unresolved:

```text
UQ-001
Exact linear-attention algorithm and kernel semantics

UQ-002
Whether the linear-attention mechanism should technically be classified
as a specific Mamba/SSM/DeltaNet-derived mechanism

UQ-003
Exact full-attention operator implementation

UQ-004
Exact MLP / FFN formulation

UQ-005
Exact normalization placement

UQ-006
Exact residual connection structure

UQ-007
Exact multimodal fusion mechanism

UQ-008
Exact MTP computation path

UQ-009
Tensor-level representation of every architectural component

UQ-010
Parameter distribution across all components
```

These unknowns must not be silently converted into assumptions.

---

# 19. Architecture Boundary

This document establishes the **architectural model visible from the
verified configuration**.

It does NOT establish:

```text
- complete tensor inventory
- exact parameter count
- checkpoint byte size
- tensor-to-shard mapping
- runtime memory requirements
- KV/state memory requirements
- operator kernel implementation
- CPU/GPU/NPU partitioning
- performance characteristics
- quantization behavior
- streaming strategy
```

Those require separate research and verification.

---

# 20. Relationship to SET 0 Research

Upstream:

```text
docs/set-0/01-artifact-provenance.md
    ↓
docs/set-0/02-model-identity.md
```

These establish:

```text
artifact origin
+
artifact identity
```

This document establishes:

```text
core architecture
```

Downstream architecture documents should refine this model rather than
silently contradict it.

If new evidence contradicts a finding here:

```text
Existing Finding
      ↓
New Evidence
      ↓
Conflict
      ↓
Reconciliation
      ↓
Explicit Revision
```

Do not silently overwrite an earlier research conclusion.

---

# 21. Canonical Architecture Statement

For current SET 0 purposes, the canonical architectural statement is:

> **Qwen3.8-27B is a multimodal model with a 64-layer language stack
> combining 48 linear-attention layers and 16 full-attention layers in a
> repeating 3:1 pattern. Full-attention layers use 24 query heads, 4
> key/value heads, and a head dimension of 256. The model also contains a
> dedicated 27-layer vision subsystem with 1152-dimensional hidden
> representations and 5120-dimensional output representations, plus a
> configured one-layer MTP component.**

The exact internal algorithm of the linear-attention mechanism remains
unresolved and must be established before implementation-level decisions
are made.

---

# 22. Final Acceptance

```text
CORE ARCHITECTURE STATUS:
VERIFIED

MODEL:
Qwen3.8-27B

LANGUAGE LAYERS:
64

LINEAR-ATTENTION:
48 layers

FULL-ATTENTION:
16 layers

ATTENTION PATTERN:
[LA → LA → LA → FA] × 16

VISION:
27-layer configured subsystem

MTP:
1 layer

ARCHITECTURE ANALYSIS:
PASS
```

This document is the canonical SET0-T06 architecture checkpoint.