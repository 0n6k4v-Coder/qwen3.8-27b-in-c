# SET 0 — MLP / Feed-Forward Architecture

## Document Status

- Document: `05-mlp-architecture.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T08`
- Status: VERIFIED
- Purpose: Canonical record of the language-model MLP / Feed-Forward
  architecture of Qwen3.8-27B
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

Related research checkpoints:

```text
docs/set-0/01-artifact-provenance.md
docs/set-0/02-model-identity.md
docs/set-0/04-core-architecture.md
docs/set-0/05-attention-architecture.md
```

The configuration artifact has already been verified byte-for-byte against
the official file at the pinned revision.

---

# 2. MLP Architecture Summary

The language model uses a **dense gated SiLU MLP / Feed-Forward Network**
with three bias-free projections per language layer:

```text
Input
  │
  ├───────────────┐
  │               │
  ▼               ▼
gate_proj       up_proj
  │               │
  ▼               │
SiLU              │
  │               │
  └────── × ──────┘
         │
         ▼
      down_proj
         │
         ▼
       Output
```

The mathematical form is:

```text
MLP(x) =
    W_down(
        SiLU(W_gate(x))
        ⊙
        W_up(x)
    )
```

where:

```text
⊙ = elementwise multiplication
```

This is a gated feed-forward formulation commonly described as
**SwiGLU-style**, although this document uses the implementation-level
description "gated SiLU MLP" as the canonical terminology.

---

# 3. Hidden Dimension

Verified configuration:

```text
hidden_size = 5120
```

The language-model hidden representation therefore has width:

```text
5120
```

### Evidence Classification

```text
VERIFIED FACT
```

Source field:

```text
text_config.hidden_size
```

---

# 4. Intermediate Dimension

Verified configuration:

```text
intermediate_size = 17408
```

Therefore the MLP expands the hidden representation from:

```text
5120
```

to:

```text
17408
```

before projecting back to:

```text
5120
```

### Evidence Classification

```text
VERIFIED FACT
```

Source field:

```text
text_config.intermediate_size
```

---

# 5. Intermediate / Hidden Ratio

The dimensional expansion ratio is:

```text
17408 / 5120 = 3.4
```

Therefore:

```text
intermediate_size = 3.4 × hidden_size
```

### Important Boundary

This is a dimensional ratio only.

It is NOT itself:

* a parameter-count ratio
* a memory ratio
* a FLOP ratio

Those require the actual projection structure and tensor representation.

### Evidence Classification

```text
DERIVED FINDING
```

---

# 6. Activation Function

Verified configuration:

```text
hidden_act = silu
```

The implementation applies the configured activation to the gate branch:

```text
SiLU(gate_proj(x))
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 7. Gating Structure

The MLP is explicitly gated.

The two input projections operate in parallel:

```text
gate_proj(x)
up_proj(x)
```

The gate branch receives SiLU:

```text
SiLU(gate_proj(x))
```

and is then multiplied elementwise with the second projected branch:

```text
SiLU(gate_proj(x)) ⊙ up_proj(x)
```

The resulting tensor is passed through `down_proj`.

Therefore the MLP is not a simple single-projection activation block.

### Canonical Equation

```text
MLP(x) =
W_down(
    SiLU(W_gate(x))
    ⊙
    W_up(x)
)
```

### Evidence Classification

```text
VERIFIED FACT
```

---

# 8. Projection Structure

The implementation defines three bias-free linear projections.

## Gate Projection

```text
5120 → 17408
```

## Up Projection

```text
5120 → 17408
```

## Down Projection

```text
17408 → 5120
```

All three projections are configured without bias.

Conceptually:

```text
gate_proj:
W_gate ∈ R^(17408 × 5120)

up_proj:
W_up ∈ R^(17408 × 5120)

down_proj:
W_down ∈ R^(5120 × 17408)
```

These matrix orientations describe the mathematical operation and do not
yet establish the physical tensor layout used by the checkpoint.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 9. Per-Layer MLP Parameterization

Based on the verified projection structure:

```text
gate_proj:
5120 × 17408

up_proj:
5120 × 17408

down_proj:
17408 × 5120
```

All three matrices contain the same number of elements:

```text
5120 × 17408
=
89,128,960
```

Therefore:

```text
MLP parameters per language layer
=
3 × 89,128,960
=
267,386,880
```

Approximately:

```text
267.39M parameters per layer
```

### Evidence Classification

```text
DERIVED FINDING
```

This is derived from the verified dimensions and verified three-projection
implementation structure.

It is NOT a checkpoint metadata value.

---

# 10. Aggregate Language-Stack MLP Parameterization

The language stack contains:

```text
64 layers
```

Assuming the verified MLP structure applies uniformly to those layers:

```text
267,386,880 × 64
=
17,112,760,320
```

Therefore the aggregate MLP weight parameter count implied by this model
structure is:

```text
17,112,760,320
≈ 17.11B parameters
```

### Evidence Classification

```text
DERIVED FINDING
```

This value must later be reconciled against the complete official tensor
inventory.

It must NOT replace official checkpoint parameter metadata.

---

# 11. Theoretical Raw BF16 MLP Weight Payload

The configuration specifies:

```text
dtype = bfloat16
```

Using 2 bytes per parameter:

```text
17,112,760,320 × 2
=
34,225,520,640 bytes
```

Approximately:

```text
34.23 GB
```

This represents only the theoretical raw BF16 payload of the three MLP
projection weights.

It does NOT represent:

* total checkpoint size
* total model memory
* resident RAM
* runtime working set
* activation memory
* temporary buffers

### Evidence Classification

```text
DERIVED FINDING
```

---

# 12. MLP Layer Uniformity

The language model contains:

```text
num_hidden_layers = 64
```

and the configuration does not provide a per-layer array of differing
`intermediate_size` values.

The associated decoder-layer implementation constructs the MLP from the
common configured `intermediate_size`.

Therefore the current evidence supports:

> All 64 language layers use the same dense gated MLP structure with
> `intermediate_size = 17408`.

This includes both layer families:

```text
48 × linear_attention
16 × full_attention
```

The token-mixing mechanism changes between those layer types, but the
language MLP remains the common feed-forward component.

### Evidence Classification

```text
VERIFIED FACT
```

at the configuration/implementation level.

---

# 13. MLP Placement in the Decoder Layer

The language decoder structure places the MLP after the token-mixing
operation and its residual path.

Conceptually:

```text
Input
  │
  ▼
Input LayerNorm
  │
  ▼
Token Mixer
  ├── Linear Attention
  └── Full Attention
  │
  ▼
Residual Connection
  │
  ▼
Post-Attention LayerNorm
  │
  ▼
Gated SiLU MLP
  │
  ▼
Residual Connection
  │
  ▼
Next Layer
```

Thus:

```text
Attention family
```

and:

```text
MLP family
```

are separate architectural dimensions.

A layer being:

```text
linear_attention
```

does not imply a different MLP.

A layer being:

```text
full_attention
```

does not imply a different MLP.

---

# 14. Relationship to Attention Architecture

The verified language architecture is therefore:

```text
64 Decoder Layers
│
├── 48 × Gated DeltaNet / Linear Attention
├── 16 × Full Attention
│
└── Common Dense Gated SiLU MLP
      │
      ├── gate_proj
      ├── up_proj
      └── down_proj
```

The attention mechanism changes by layer type.

The MLP remains the common dense feed-forward component.

This separation is important for both:

* tensor accounting
* operator implementation

---

# 15. MLP Is Dense, Not Sparse MoE

The current Qwen3.8-27B language configuration and the associated
implementation construct a dense `Qwen3_5MLP` rather than a sparse expert
MLP block.

Therefore this artifact should NOT be classified as an MoE model merely
because its total parameter count is large.

The current architecture evidence supports:

```text
Dense MLP
```

rather than:

```text
Sparse MoE MLP
```

### Important Boundary

This conclusion is specifically about the **verified language-model MLP
architecture**.

It must not be extended to unrelated Qwen checkpoints or variants.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 16. Implementation Evidence

The implementation lineage is:

```text
Qwen3.8 config
    ↓
Qwen3_5ForConditionalGeneration
    ↓
Qwen3_5DecoderLayer
    ↓
Qwen3_5MLP
    ↓
Qwen3NextMLP
```

The MLP implementation defines:

```text
gate_proj
up_proj
down_proj
```

and computes:

```text
down_proj(
    SiLU(gate_proj(x)) * up_proj(x)
)
```

The implementation also distinguishes this dense MLP from separate sparse
MoE blocks.

This provides implementation-level support for the MLP structure described
in this document.

---

# 17. Verified Facts

The following findings are established:

```text
✅ hidden_size = 5120

✅ intermediate_size = 17408

✅ hidden_act = SiLU

✅ MLP is gated

✅ gate_proj exists

✅ up_proj exists

✅ down_proj exists

✅ gate_proj:
   5120 → 17408

✅ up_proj:
   5120 → 17408

✅ down_proj:
   17408 → 5120

✅ All three projections are bias-free

✅ Forward structure:
   down_proj(SiLU(gate_proj(x)) ⊙ up_proj(x))

✅ Common MLP structure is used across the language layers

✅ MLP is a dense feed-forward component

✅ MLP occurs after the token-mixing residual path
```

---

# 18. Derived Findings

```text
17408 / 5120
=
3.4
```

```text
5120 × 17408
=
89,128,960
```

```text
3 × 89,128,960
=
267,386,880
```

```text
267,386,880 × 64
=
17,112,760,320
```

Therefore:

```text
Per-layer MLP:
267.39M parameters

64-layer MLP:
17.11B parameters
```

And theoretical BF16 payload:

```text
17,112,760,320 × 2
=
34,225,520,640 bytes
≈ 34.23 GB
```

These are derived values and must later be reconciled against the official
tensor inventory.

---

# 19. Inferences

### I1 — MLP is a major parameter component

The MLP alone contributes approximately:

```text
17.11B parameters
```

under the current verified projection structure.

Therefore MLP weights are expected to be one of the largest contributors
to model storage and memory bandwidth requirements.

This is an engineering inference, not a benchmark result.

### I2 — MLP computation is structurally uniform across layer types

Both:

```text
linear_attention
```

and:

```text
full_attention
```

layers feed into the same dense MLP architecture.

Therefore operator specialization is primarily required for the token mixer,
not for the MLP structure itself.

### I3 — MLP must be treated separately during tensor accounting

The three projection tensors should eventually be classified separately:

```text
gate_proj
up_proj
down_proj
```

rather than treating the MLP as one undifferentiated tensor block.

---

# 20. Unknown / Further Verification

The following remain unresolved:

```text
UQ-MLP-001
Exact tensor names in the Qwen3.8-27B checkpoint.

UQ-MLP-002
Exact checkpoint tensor shapes and physical storage layout.

UQ-MLP-003
Whether gate/up weights are stored separately or in a fused/packed form.

UQ-MLP-004
Exact shard placement of gate/up/down projection tensors.

UQ-MLP-005
Exact quantized representation for future deployment experiments.

UQ-MLP-006
Whether the final inference engine should use fused MLP kernels.

UQ-MLP-007
Actual MLP memory bandwidth and compute cost on Intel Core Ultra 7 155H.

UQ-MLP-008
Exact reconciliation of the derived MLP parameter count against the
official tensor index.
```

These questions belong to later tensor, runtime, and performance analysis.

---

# 21. MLP Operator Model

For future operator analysis, the canonical MLP model is:

```text
                x
                │
        ┌───────┴───────┐
        │               │
        ▼               ▼
   gate_proj          up_proj
        │               │
        ▼               │
      SiLU              │
        │               │
        └───────┬───────┘
                │
        Elementwise Multiply
                │
                ▼
           down_proj
                │
                ▼
                y
```

Dimensions:

```text
x:
5120

gate_proj(x):
17408

up_proj(x):
17408

elementwise product:
17408

down_proj(...):
5120
```

Therefore a future reference implementation will require at least:

```text
GATE PROJECTION
UP PROJECTION
SILU
ELEMENTWISE MULTIPLICATION
DOWN PROJECTION
```

The optimized implementation may later fuse these operations, but such
optimization is outside SET 0.

---

# 22. Relationship to Later Research

## SET 1 — Tensor / Byte-Level Audit

Must eventually identify:

```text
gate_proj tensors
up_proj tensors
down_proj tensors
```

and reconcile their exact:

```text
shape
dtype
parameter count
storage bytes
shard
```

## SET 3 — Operator / Computation Model

Must define:

```text
Gated SiLU MLP
```

as a distinct operator pipeline.

## SET 4 — Runtime Memory Model

Must account for:

```text
gate projection weights
up projection weights
down projection weights
intermediate activation
temporary multiplication buffer
```

or equivalent fused representations.

---

# 23. Research Boundary

This document establishes the **MLP architecture** visible from the
verified configuration and associated implementation.

It does NOT establish:

```text
- complete tensor inventory
- checkpoint byte size
- exact shard distribution
- optimized kernel implementation
- CPU/GPU/NPU placement
- quantization implementation
- inference performance
- runtime memory footprint
```

Those topics require separate verification.

---

# 24. Canonical MLP Statement

For current SET 0 purposes:

> **Qwen3.8-27B uses a dense gated SiLU MLP across its 64 language layers. Each MLP expands the 5120-dimensional hidden state to a 17408-dimensional intermediate representation through separate bias-free `gate_proj` and `up_proj` matrices, applies SiLU to the gate branch, performs elementwise multiplication with the up branch, and projects the result back to 5120 dimensions through `down_proj`. This implies 267,386,880 MLP weight parameters per language layer and approximately 17.11B MLP weight parameters across 64 layers, pending reconciliation with the official tensor inventory.**

---

# 25. Final Acceptance

```text
MLP ARCHITECTURE STATUS:
VERIFIED

MLP TYPE:
Dense Gated SiLU

HIDDEN SIZE:
5120

INTERMEDIATE SIZE:
17408

PROJECTION COUNT:
3

PROJECTION STRUCTURE:
5120 → 17408
5120 → 17408
17408 → 5120

LANGUAGE LAYERS:
64

DERIVED MLP PARAMETERS:
17,112,760,320

ARCHITECTURE ANALYSIS:
PASS
```

This document is the canonical `SET0-T08` MLP / FFN architecture
checkpoint.