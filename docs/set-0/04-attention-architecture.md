# SET 0 — Attention Architecture

## Document Status

- Document: `04-attention-architecture.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T07`
- Status: VERIFIED
- Purpose: Canonical record of the attention mechanisms used by Qwen3.8-27B
- Responsibility: 🧠 LUNA

---

## 1. Source and Provenance

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

Primary configuration artifact:

```text
model/official/config.json
```

The configuration artifact has already been verified byte-for-byte against
the official file at the pinned revision.

Related documents:

```text
docs/set-0/01-artifact-provenance.md
docs/set-0/02-model-identity.md
docs/set-0/04-core-architecture.md
```

---

# 2. Attention Architecture Summary

Qwen3.8-27B contains two distinct attention families in its 64-layer
language stack:

```text
48 × linear_attention
16 × full_attention
```

The layer pattern is:

```text
[ LA → LA → LA → FA ] × 16
```

where:

```text
LA = linear_attention
FA = full_attention
```

The official implementation associated with the artifact's
`Qwen3_5ForConditionalGeneration` architecture identity maps these two
configuration types to two distinct implementations:

```text
linear_attention
        ↓
Qwen3_5GatedDeltaNet

full_attention
        ↓
Qwen3_5Attention
```

This is an implementation-level refinement of the architecture identified
in `04-core-architecture.md`.

---

# 3. Full Attention

## 3.1 Configuration

The full-attention configuration specifies:

```text
num_attention_heads:
24

num_key_value_heads:
4

head_dim:
256

attention_bias:
false

attention_dropout:
0.0

attn_output_gate:
true

output_gate_type:
swish
```

---

## 3.2 Query / Key / Value Organization

The full-attention path uses:

```text
24 query heads
4 key/value heads
256 dimensions per head
```

Therefore:

```text
24 / 4 = 6
```

Each key/value head is shared by 6 query heads.

This is a:

```text
Grouped-Query Attention (GQA)
```

configuration.

### Derived dimensions

Query representation:

```text
24 × 256 = 6144
```

Key representation:

```text
4 × 256 = 1024
```

Value representation:

```text
4 × 256 = 1024
```

These are configuration-derived projection dimensions and do not by
themselves constitute the complete tensor inventory.

---

## 3.3 Full-Attention Execution Characteristics

The official implementation associated with the artifact's architecture
identity establishes the following major stages:

```text
Hidden States
    ↓
Q / KV projection
    ↓
Q/K normalization
    ↓
Rotary Position Encoding
    ↓
KV Cache
    ↓
Causal Attention
    ↓
Output Gating
    ↓
Output Projection
```

Therefore full-attention layers use a conventional attention history
represented by a KV cache.

### Verified implementation-level findings

```text
QK normalization
RoPE
KV cache
causal attention
attention output gate
output projection
```

---

# 4. Full-Attention Rotary Configuration

The language configuration specifies:

```text
partial_rotary_factor:
0.25

rope_theta:
10000000

rope_type:
default

mrope_interleaved:
true

mrope_section:
[11, 11, 10]
```

With:

```text
head_dim = 256
partial_rotary_factor = 0.25
```

the configured rotary dimension is:

```text
256 × 0.25 = 64
```

Therefore:

```text
rotary dimension = 64
```

The exact semantics of each MRoPE section remain implementation details
and must not be generalized beyond the official implementation evidence.

---

# 5. Full-Attention Output Gating

The configuration explicitly enables attention output gating:

```text
attn_output_gate:
true

output_gate_type:
swish
```

The associated implementation applies a learned gate to the attention
output before the final output projection.

Therefore gating is part of the full-attention execution path.

---

# 6. Linear Attention

## 6.1 Configuration

The linear-attention configuration specifies:

```text
linear_num_key_heads:
16

linear_num_value_heads:
48

linear_key_head_dim:
128

linear_value_head_dim:
128

linear_conv_kernel_dim:
4
```

Therefore the linear-attention representation is structurally different
from the full-attention representation.

---

## 6.2 Linear-Attention Dimensions

Key representation:

```text
16 × 128 = 2048
```

Value representation:

```text
48 × 128 = 6144
```

The configuration therefore has:

```text
16 key heads
48 value heads
128 dimensions per head
```

The implementation expands the key/query head count to match the value
head count.

The ratio is:

```text
48 / 16 = 3
```

Thus the 16 key/query heads are repeated across 48 processing heads.

This must not be confused with the GQA structure of full attention.

---

# 7. Linear Attention Implementation

The implementation associated with the artifact architecture identity
maps:

```text
linear_attention
```

to:

```text
Qwen3_5GatedDeltaNet
```

The implementation contains the following major components:

```text
Input Projection
    ↓
Q / K / V-related projections
    ↓
Causal Convolution
    ↓
Gated Delta Rule
    ↓
Recurrent State Update
    ↓
Output Gating / Normalization
    ↓
Output Projection
```

The important architectural result is:

> The linear-attention layers are Gated DeltaNet-style recurrent attention
> layers rather than conventional full-attention layers with a reduced
> number of heads.

---

# 8. Linear-Attention Convolution

The configuration specifies:

```text
linear_conv_kernel_dim:
4
```

The associated implementation uses a causal convolution stage before the
gated-delta computation.

Therefore each linear-attention layer has a local convolution/state
component in addition to its recurrent attention state.

This is an important distinction for later inference-engine design.

---

# 9. Linear-Attention State

The Gated DeltaNet implementation maintains recurrent state.

Therefore the state model is fundamentally different from standard
full-attention KV caching.

Conceptually:

```text
Full Attention
    ↓
KV Cache

Gated DeltaNet
    ↓
Recurrent State
+
Convolution State
```

This distinction is critical for the future runtime memory model.

The linear-attention state must not be represented as a standard full
attention KV cache.

---

# 10. Gated Delta Rule

The implementation provides both chunk-oriented and recurrent execution
paths for the gated-delta rule.

Conceptually:

```text
Q / K / V
    ↓
Gating parameters
    ↓
Delta-rule update
    ↓
State
    ↓
Output
```

The recurrent path explicitly maintains state across token execution.

Therefore the linear-attention mechanism is stateful during autoregressive
execution.

---

# 11. Attention Family Comparison

| Property             | Full Attention                 | Linear Attention                                             |
| -------------------- | ------------------------------ | ------------------------------------------------------------ |
| Configuration type   | `full_attention`               | `linear_attention`                                           |
| Implementation class | `Qwen3_5Attention`             | `Qwen3_5GatedDeltaNet`                                       |
| Layer count          | 16                             | 48                                                           |
| Q heads              | 24                             | 16 before expansion                                          |
| K heads              | 4                              | 16                                                           |
| V heads              | 4                              | 48                                                           |
| Head dimension       | 256                            | 128                                                          |
| Q/K/V organization   | GQA                            | K/V head expansion                                           |
| Main state           | KV cache                       | Recurrent state                                              |
| Convolution          | Not the defining mechanism     | Kernel size 4                                                |
| Gated delta rule     | No                             | Yes                                                          |
| RoPE                 | Yes in the full-attention path | Exact treatment requires implementation-level interpretation |
| Output gating        | Yes                            | Gated output path                                            |
| Attention pattern    | Conventional causal attention  | Recurrent linear-attention mechanism                         |

---

# 12. Layer Placement

The 64-layer topology is:

```text
Layer 00  linear_attention
Layer 01  linear_attention
Layer 02  linear_attention
Layer 03  full_attention

Layer 04  linear_attention
Layer 05  linear_attention
Layer 06  linear_attention
Layer 07  full_attention

...

Layer 60  linear_attention
Layer 61  linear_attention
Layer 62  linear_attention
Layer 63  full_attention
```

Equivalent rule:

```text
layer_index % 4 == 3
    → full_attention

otherwise
    → linear_attention
```

This is consistent with:

```text
full_attention_interval = 4
```

and:

```text
[LA → LA → LA → FA] × 16
```

---

# 13. Attention Architecture Diagram

```text
Qwen3.8-27B
      │
      ▼
64-Layer Language Stack
      │
      ├───────────────────────────────────────┐
      │                                       │
      ▼                                       ▼
48 × Linear Attention                   16 × Full Attention
      │                                       │
      ▼                                       ▼
Qwen3_5GatedDeltaNet                  Qwen3_5Attention
      │                                       │
      ├── 16 K heads                          ├── 24 Q heads
      ├── 48 V heads                          ├── 4 KV heads
      ├── 128-d heads                         ├── 256-d heads
      ├── Causal Conv                         ├── Q/K normalization
      ├── Gated Delta Rule                    ├── RoPE
      ├── Recurrent State                     ├── KV Cache
      └── Gated Output                        └── Gated Output
      │                                       │
      └───────────────────┬───────────────────┘
                          ▼
                       Next Layer
```

---

# 14. Verified Facts

The following are considered verified from Qwen3.8-specific configuration
and the associated official implementation identity:

```text
✅ 64 language layers

✅ 48 linear-attention layers
✅ 16 full-attention layers

✅ Pattern:
   [LA → LA → LA → FA] × 16

✅ linear_attention maps to Qwen3_5GatedDeltaNet

✅ full_attention maps to Qwen3_5Attention

✅ Full Attention:
   24 Q heads
   4 KV heads
   head_dim = 256

✅ Full Attention:
   6 query heads per KV head

✅ Full Attention:
   Q/K normalization
   RoPE
   KV cache
   causal attention
   output gating

✅ Linear Attention:
   16 key heads
   48 value heads
   128-dimensional heads

✅ Linear Attention:
   K/query processing is expanded to the 48-value-head width

✅ Linear Attention:
   causal convolution
   kernel dimension = 4

✅ Linear Attention:
   gated delta rule

✅ Linear Attention:
   recurrent state

✅ Rotary dimension:
   256 × 0.25 = 64
```

---

# 15. Derived Findings

## 15.1 Two Different State Models

The language model does not have one universal attention-state mechanism.

It has:

```text
16 Full-Attention layers
    → KV cache

48 Gated DeltaNet layers
    → recurrent state + convolution state
```

This is a fundamental architectural distinction.

---

## 15.2 Attention Is Not Uniform Across Layers

The 64 layers cannot be implemented using one generic attention kernel.

At minimum, the engine needs separate operator paths for:

```text
Full Attention
```

and:

```text
Gated DeltaNet
```

---

## 15.3 Decode-Time Memory Is Heterogeneous

The attention-state memory model must distinguish:

```text
KV Cache
```

from:

```text
Recurrent State
+
Convolution State
```

This will become important in SET 4 — Runtime Memory Model.

---

# 16. Inferences

The following are reasonable engineering interpretations derived from the
verified architecture:

### I1

A future inference engine cannot model all 64 language layers as
conventional attention layers.

### I2

A uniform KV-cache allocator for all language layers would be architecturally
incorrect.

### I3

The linear-attention state should be modeled independently from full
attention's KV cache.

### I4

Operator scheduling and memory management will need layer-type awareness.

---

# 17. Unknown / Further Verification

The following remain unresolved:

```text
UQ-001
Exact tensor names and shapes for every attention projection.

UQ-002
Exact parameter distribution between Gated DeltaNet and full attention.

UQ-003
Exact recurrent-state tensor layout in the Qwen3.8-27B checkpoint.

UQ-004
Exact state/cache dtype behavior under the final deployment configuration.

UQ-005
Exact optimized CPU/GPU/NPU kernel implementation.

UQ-006
Exact runtime handling of the MRoPE sections in every execution path.

UQ-007
Exact attention FLOPs and memory traffic.

UQ-008
Exact weight tensors required by each attention family.

UQ-009
Exact multimodal interaction with attention state during image/video
generation or understanding workloads.
```

These questions are now implementation- and tensor-level questions rather
than model-identity questions.

---

# 18. Evidence Classification Boundary

### VERIFIED FACT

Directly established from the verified configuration or the identified
official implementation.

### DERIVED FINDING

Mathematically derived from verified configuration values.

Examples:

```text
24 / 4 = 6
256 × 0.25 = 64
48 / 16 = 3
```

### INFERENCE

Engineering interpretation derived from the verified architecture.

### UNKNOWN

Not yet established sufficiently to justify implementation decisions.

No UNKNOWN item should silently become a design assumption.

---

# 19. Implications for Future SETs

This attention finding establishes several dependencies for later work.

### SET 0

Required to complete:

```text
Exact layer topology
MLP formulation
Vision/MTP integration
Tensor-level mapping
```

### SET 1 — Tensor / Byte-Level Audit

Must identify:

```text
Full-attention tensors
Gated DeltaNet tensors
KV-related tensors
Recurrent-state-related tensors
Convolution tensors
Projection tensors
```

### SET 3 — Operator / Computation Model

Must model separately:

```text
Full Attention Operator
Gated DeltaNet Operator
```

### SET 4 — Runtime Memory Model

Must model separately:

```text
KV Cache
Recurrent State
Convolution State
```

---

# 20. Canonical Attention Statement

For current SET 0 purposes:

> **Qwen3.8-27B uses a 64-layer hybrid attention language stack consisting of 48 Gated DeltaNet-based linear-attention layers and 16 full-attention layers in a repeating `[linear, linear, linear, full] × 16` pattern. Full-attention layers use 24 query heads, 4 key/value heads, 256-dimensional heads, RoPE, KV caching, and output gating. Linear-attention layers use 16 key heads and 48 value heads with 128-dimensional heads, causal convolution, gated-delta recurrence, and recurrent state. The two layer families therefore require distinct operator and state-management models.**

---

# 21. Final Acceptance

```text
ATTENTION ARCHITECTURE STATUS:
VERIFIED

FULL ATTENTION:
16 layers

LINEAR / GATED DELTANET:
48 layers

PATTERN:
[LA → LA → LA → FA] × 16

FULL ATTENTION STATE:
KV CACHE

LINEAR ATTENTION STATE:
RECURRENT STATE + CONVOLUTION STATE

ATTENTION ANALYSIS:
PASS
```

This document is the canonical `SET0-T07` attention-architecture
checkpoint.