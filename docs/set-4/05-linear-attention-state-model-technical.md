# SET 4 — Linear-Attention State Model — Orchestrator Technical Model

## Document Status

- **Document:** `docs/set-4/05-linear-attention-state-model-technical.md`
- **SET:** `SET 4 — Runtime Memory Model`
- **Source Task:** `SET4-T4.5`
- **Role:** ORCHESTRATOR technical reasoning / derivation / calculation / acceptance
- **Status:** **TECHNICAL PASS — CONTROL CLOSURE PENDING**
- **Dependency:** `SET4-T4.4 PASS`
- **Raw Executor Evidence:** `docs/set-4/05-linear-attention-state-model.md`
- **Model:** `Qwen/Qwen3.8-27B` / Qwen3.5-27B architecture family

This document is the Orchestrator-owned technical model. The Executor artifact remains
unchanged and retains its evidence-only status.

```text
EXECUTOR EVIDENCE
≠
ORCHESTRATOR DERIVATION
≠
CONDITIONAL RUNTIME MODEL
≠
TECHNICAL ACCEPTANCE
≠
CONTROL AUTHORIZATION
```

---

## 1. Purpose and Responsibility Boundary

T4.5 models the persistent state associated with the 48 linear-attention layers.
The state domain is structurally distinct from the 16 full-attention KV-cache layers.

The base model therefore contains two different persistent attention-state families:

```text
Full Attention
→ KV Cache

Linear Attention
→ Recurrent State + Convolution State
```

The project evidence establishes this distinction directly. SET3 identifies the
linear-attention operator as `Qwen3_5GatedDeltaNet`, with verified projection/tensor
shapes and a state-model distinction, while the exact runtime allocator remains a
separate implementation question. [VERIFIED FACT]

The official Hugging Face Qwen3.5 implementation independently corroborates the
same architecture and exposes the state representation used by the reference
implementation: recurrent state is represented as a tensor with dimensions
`[batch, num_value_heads, key_head_dim, value_head_dim]`, and causal-convolution
state preserves the previous `kernel_size - 1` time positions. This is implementation
source documentation, not a runtime measurement of the present environment.
[VERIFIED FACT — reference implementation source]

---

## 2. Authoritative Structural Inputs

| Variable | Value | Classification | Meaning |
|---|---:|---|---|
| `L_LA` | 48 | VERIFIED FACT | Language-model linear-attention layers |
| `H_K` | 16 | VERIFIED FACT | Key/query head count before V-head expansion |
| `H_V` | 48 | VERIFIED FACT | Value-head count |
| `D_K` | 128 | VERIFIED FACT | Key head dimension |
| `D_V` | 128 | VERIFIED FACT | Value head dimension |
| `K_conv` | 4 | VERIFIED FACT | Causal convolution kernel size |
| `C_conv` | 10240 | DERIVED FINDING | Conv channels / QKV width |
| `mamba_ssm_dtype` | `float32` | VERIFIED FACT | Configuration field; not, by itself, runtime state allocation evidence |
| checkpoint dtype | BF16 | VERIFIED FACT | Checkpoint tensor dtype |
| `B` | UNKNOWN | UNKNOWN | Runtime/request batch size |
| `E_r` | UNKNOWN | UNKNOWN | Runtime recurrent-state bytes/element |
| `E_c` | UNKNOWN | UNKNOWN | Runtime convolution-state bytes/element |
| allocation strategy | UNKNOWN | UNKNOWN | Physical runtime allocation policy |
| layout | UNKNOWN | UNKNOWN | Physical tensor layout/striding |
| quantization | UNKNOWN | UNKNOWN | Runtime state quantization |

The configuration source explicitly contains `linear_num_key_heads=16`,
`linear_num_value_heads=48`, `linear_key_head_dim=128`,
`linear_value_head_dim=128`, `linear_conv_kernel_dim=4`, and
`mamba_ssm_dtype="float32"`; it also contains `use_cache=true`. [VERIFIED FACT]

The project evidence correctly separates the `mamba_ssm_dtype` configuration field
from checkpoint tensor dtype and does not use it alone to prove live runtime state
allocation.

---

## 3. Classification Correction from Executor Evidence

The Executor evidence characterized the proposition:

```text
mamba_ssm_dtype=float32
→ metadata only
→ VERIFIED FACT
```

The correction is narrower and more precise:

1. `mamba_ssm_dtype="float32"` is a **VERIFIED FACT as a configuration field**.
2. The proposition that the field is **not by itself proof of runtime state allocation**
is a **DERIVED FINDING / boundary conclusion**, because no runtime was executed.
3. The present environment's actual runtime recurrent-state dtype remains **UNKNOWN**.
4. The official Hugging Face reference implementation uses explicit float conversion
in the gated-delta computation and exposes a recurrent-state representation; that is
**VERIFIED FACT about the documented reference implementation**, not a local runtime
measurement.

Therefore this task does **not** promote:

```text
checkpoint BF16
→ runtime recurrent state BF16
```

or:

```text
mamba_ssm_dtype=float32
→ local runtime allocation observed as FP32
```

Both would be unsupported runtime claims.

---

## 4. State-Architecture Reconstruction

### 4.1 Layer topology

The accepted checkpoint/config evidence gives 64 language-model layers with the
repeating pattern:

```text
[LA → LA → LA → FA] × 16
```

Therefore:

```text
L_LA = 48
L_FA = 16
```

[VERIFIED FACT / DERIVED FINDING]

### 4.2 Linear-attention operator identity

The project operator model maps `linear_attention` to `Qwen3_5GatedDeltaNet`.
The verified tensor set contains:

```text
in_proj_qkv   [10240, 5120]
in_proj_z     [6144, 5120]
in_proj_b     [48, 5120]
in_proj_a     [48, 5120]
out_proj      [5120, 6144]
conv1d        [10240, 1, 4]
A_log         [48]
dt_bias       [48]
norm          [128]
```

[VERIFIED FACT]

### 4.3 State-family boundary

The linear-attention state domain contains:

```text
1. recurrent state
2. convolution state
```

The state-family separation from T4.4 is preserved:

```text
T4.4 full-attention KV cache
≠
T4.5 recurrent state
≠
T4.5 convolution state
```

[VERIFIED FACT]

### 4.4 Recurrent-state structural representation

The official Hugging Face Qwen3.5 implementation defines the gated-delta recurrence
with:

```text
num_v_heads = 48
head_k_dim = 128
head_v_dim = 128
```

and its recurrent update function carries state with shape:

```text
[B, H_V, D_K, D_V]
```

For this model:

```text
[B, 48, 128, 128]
```

The project checkpoint tensor `in_proj_qkv` supplies Q and K at 16 heads and V at
48 heads. The reference implementation expands Q/K to the value-head multiplicity
before the recurrent update. [VERIFIED FACT — reference implementation]

Thus the recurrent state contains one `D_K × D_V` matrix per value head.

---

## 5. Recurrent-State Model

### 5.1 Elements per layer

The structural recurrent-state shape is:

```text
[B, H_V, D_K, D_V]
```

Therefore:

```text
R_elements_per_layer
=
B × H_V × D_K × D_V
```

Substitute:

```text
H_V = 48
D_K = 128
D_V = 128
```

Therefore:

```text
R_elements_per_layer
=
B × 48 × 128 × 128
=
786,432 × B
```

[DERIVED FINDING]

### 5.2 Across all 48 linear-attention layers

```text
R_elements_total
=
L_LA × R_elements_per_layer
```

```text
=
48 × 786,432 × B
```

```text
=
37,748,736 × B elements
```

[DERIVED FINDING]

### 5.3 Recurrent-state memory

Let:

```text
E_r = runtime recurrent-state bytes/element
```

Then:

```text
M_recurrent_bytes
=
37,748,736 × B × E_r
```

[CONDITIONAL MODEL]

No runtime value is assigned to `E_r`.

---

## 6. Convolution-State Model

### 6.1 Convolution channel width

The fused QKV projection width is:

```text
C_conv
=
2 × (H_K × D_K) + (H_V × D_V)
```

Substitute:

```text
H_K = 16
D_K = 128
H_V = 48
D_V = 128
```

```text
C_conv
=
2 × (16 × 128) + (48 × 128)
```

```text
=
2 × 2048 + 6144
```

```text
=
10240
```

This matches the verified `in_proj_qkv` and `conv1d` output width.

[DERIVED FINDING]

### 6.2 Runtime causal-convolution state length

The reference implementation maintains previous causal-convolution context of:

```text
K_conv - 1
```

positions.

With:

```text
K_conv = 4
```

this becomes:

```text
state_length = 4 - 1 = 3
```

Therefore the reference implementation's structural convolution-state shape is:

```text
[B, 10240, 3]
```

[VERIFIED FACT — reference implementation]

This is a state-buffer representation, not the checkpoint kernel parameter shape.

Specifically:

```text
conv1d.weight
=
[10240, 1, 4]
```

is parameter storage, whereas:

```text
conv_state
=
[B, 10240, 3]
```

is the preserved causal context in the reference implementation.

The two must not be conflated.

### 6.3 Convolution-state elements per layer

```text
C_elements_per_layer
=
B × C_conv × (K_conv - 1)
```

Substitution:

```text
=
B × 10240 × 3
```

```text
=
30,720 × B elements
```

[DERIVED FINDING]

### 6.4 Across all 48 linear-attention layers

```text
C_elements_total
=
48 × 30,720 × B
```

```text
=
1,474,560 × B elements
```

[DERIVED FINDING]

Let:

```text
E_c = runtime convolution-state bytes/element
```

Then:

```text
M_conv_bytes
=
1,474,560 × B × E_c
```

[CONDITIONAL MODEL]

---

## 7. Combined Linear-Attention State Memory Model

The T4.5 state domain is:

```text
recurrent state
+
convolution state
```

Therefore:

```text
M_T4.5_bytes
=
M_recurrent_bytes + M_conv_bytes
```

Substitute the derived coefficients:

```text
M_T4.5_bytes
=
B × (37,748,736 × E_r + 1,474,560 × E_c)
```

This is the primary parameterized T4.5 state-memory equation.

[CONDITIONAL MODEL]

The equation exposes independently the two unresolved state element sizes.
This is preferable to silently assuming a common dtype.

### 7.1 Equivalent element-count form

```text
M_T4.5_elements
=
B × (37,748,736 + 1,474,560)
```

```text
=
39,223,296 × B elements
```

This element count is only meaningful as the combined reference-implementation
state representation; it does not assert a common runtime dtype.

[DERIVED FINDING]

---

## 8. Conditional Dtype Cases

### 8.1 Common BF16 conditional case

As a pure scaling case:

```text
E_r = 2 bytes
E_c = 2 bytes
```

Then:

```text
M_T4.5_bytes
=
39,223,296 × B × 2
```

```text
=
78,446,592 × B bytes
```

For one batch element:

```text
B = 1
→
78,446,592 bytes
```

```text
≈ 74.8125 MiB
≈ 0.07306 GiB
```

[CONDITIONAL MODEL]

This is **not** a claim that the runtime stores both state families in BF16.

### 8.2 Reference-implementation-style FP32 recurrent / BF16 convolution case

A second conditional case keeps the two state domains separate:

```text
E_r = 4 bytes
E_c = 2 bytes
```

Then:

```text
M_T4.5_bytes
=
37,748,736 × 4 × B
+
1,474,560 × 2 × B
```

```text
=
153,944,064 × B bytes
```

For:

```text
B = 1
```

```text
153,944,064 bytes
≈ 146.8125 MiB
≈ 0.14337 GiB
```

[CONDITIONAL MODEL]

This case is motivated by the documented reference implementation's float32
recurrent computation/state handling, but remains a **conditional implementation
case**, not a live observation of this repository's runtime.

### 8.3 Fully FP32 conditional case

```text
E_r = 4
E_c = 4
```

```text
M_T4.5_bytes
=
156,893,184 × B bytes
```

For `B=1`:

```text
≈ 149.625 MiB
≈ 0.14612 GiB
```

[CONDITIONAL MODEL]

---

## 9. Token/Sequence-Length Semantics

A critical distinction from T4.4:

The persistent T4.5 state representation is **not inherently proportional to the
entire retained sequence length `S`**.

For a stateful recurrent implementation, the persistent state is fixed-size with
respect to sequence length after the state shape is established. Sequence length
still affects transient activations/workspace and execution work, but it does not
add another persistent recurrent-state matrix for every prior token.

Thus the base persistent state model is:

```text
M_T4.5_state = f(B, E_r, E_c)
```

not:

```text
f(B, S, E_state)
```

[DERIVED FINDING]

The actual runtime may allocate or reuse these state buffers according to a
framework-specific cache/allocator strategy, which remains a separate UNKNOWN.

---

## 10. Dimensional Validation

### 10.1 Recurrent state

```text
batch
×
V heads
×
K-head elements
×
V-head elements
×
bytes/element
=
bytes
```

```text
B × H_V × D_K × D_V × E_r
=
bytes
```

PASS.

### 10.2 Convolution state

```text
batch
×
conv channels
×
retained causal-context positions
×
bytes/element
=
bytes
```

```text
B × C_conv × (K_conv - 1) × E_c
=
bytes
```

PASS.

### 10.3 Combined state

```text
B × [H_V × D_K × D_V × E_r
   + C_conv × (K_conv - 1) × E_c]
```

Both terms reduce to bytes, so:

```text
M_T4.5_bytes = bytes
```

**DIMENSIONAL VALIDATION = PASS**

---

## 11. Unknown / Limitation Register

The following remain UNKNOWN for the present repository environment because no
Qwen3.5 runtime was executed here:

| Unknown | Status | Consequence |
|---|---|---|
| Live recurrent-state dtype in this environment | UNKNOWN | `E_r` remains parameterized |
| Live conv-state dtype in this environment | UNKNOWN | `E_c` remains parameterized |
| Actual allocator implementation | UNKNOWN | No physical-allocation claim |
| Actual state layout/strides | UNKNOWN | No layout claim |
| Actual state sharing/reuse policy | UNKNOWN | No sharing claim |
| Actual quantization/compression | UNKNOWN | No quantization claim |
| Runtime batch size `B` | UNKNOWN | Batch parameter remains explicit |
| Runtime state lifecycle | UNKNOWN | No preallocation/reuse conclusion |
| Whether this local environment exercised `use_cache` | UNKNOWN | Config flag is not runtime evidence |
| MTP state contribution | UNKNOWN / CONDITIONAL | Excluded from base T4.5 |

The reference implementation source reduces the **architectural** uncertainty around
the state representation, but it does not turn this repository's unexecuted runtime
into an observed runtime. [VERIFIED FACT — reference implementation] / [UNKNOWN — local runtime]

---

## 12. MTP Boundary

The model contains one MTP layer in the checkpoint configuration, and MTP self-attention
tensors exist in checkpoint metadata. However, T4.5 models the base 48 language-model
linear-attention layers.

Therefore:

```text
BASE T4.5
=
48 linear-attention layers
```

and:

```text
MTP state contribution
=
UNKNOWN / CONDITIONAL
```

No MTP self-attention state is silently merged into the 48-layer base state formula.

---

## 13. Domain-Separation Validation

T4.5 is intentionally kept separate from all adjacent SET4 domains:

```text
T4.2 = weight residency
T4.3 = activation lifetime / boundary accounting
T4.4 = full-attention KV state
T4.5 = linear-attention recurrent + convolution state
T4.6 = workspace / transient runtime memory
```

In particular:

```text
T4.5 state memory
≠
T4.4 KV memory
```

and:

```text
T4.5 state memory
≠
T4.3 activation memory
```

and:

```text
T4.5 state memory
≠
TOTAL RUNTIME MEMORY
```

[VERIFIED FACT / DERIVED FINDING]

---

## 14. Hardware Reconciliation Boundary

No workload placement or fit conclusion is made here.

The hardware available in the prior evidence environment is not a live accelerator
runtime for the model. Therefore this task does not claim that the model fits, runs,
or meets performance requirements on that hardware.

The state equations are useful inputs to the later aggregate memory reconciliation,
but they are not themselves a placement or performance result.

---

## 15. Technical Acceptance

### Acceptance Matrix

| Criterion | Status | Rationale |
|---|---|---|
| Linear-attention architecture established | PASS | 48-layer Gated DeltaNet family established |
| 48 LA layers established | PASS | config/layer topology verified |
| Recurrent-state domain established | PASS | project architecture + reference implementation |
| Convolution-state domain established | PASS | project architecture + reference implementation |
| Structural state dimensions resolved where possible | PASS | reference implementation exposes `[B,H_V,D_K,D_V]` and causal context `K-1` |
| State-memory equations derived | PASS | recurrent + convolution terms derived independently |
| Dimensions checked | PASS | both terms reduce to bytes |
| Runtime parameters parameterized | PASS | `B`, `E_r`, `E_c`, allocation/layout remain explicit |
| Runtime UNKNOWNs preserved | PASS | no local runtime behavior promoted |
| Checkpoint dtype separated from runtime state dtype | PASS | explicit distinction preserved |
| Parameter tensors separated from state buffers | PASS | `A_log`/`dt_bias`/`conv1d.weight` not conflated with state |
| MTP boundary preserved | PASS | MTP excluded from base model |
| T4.4 KV state kept separate | PASS | state families not merged |
| Unsupported allocator claim avoided | PASS | no physical allocation claim |
| Unsupported performance claim avoided | PASS | no throughput/latency claim |

### Technical Verdict

```text
T4.5 TECHNICAL ACCEPTANCE = PASS
```

The acceptance means the **parameterized structural state-memory model** required by
T4.5 has been established. It does not mean that a live runtime allocator or runtime
state dtype was observed in this environment.

---

## 16. Documentation and Persistence

The Executor artifact remains unchanged:

```text
docs/set-4/05-linear-attention-state-model.md
```

This document is the separate Orchestrator technical model:

```text
docs/set-4/05-linear-attention-state-model-technical.md
```

The technical model must remain separately identifiable from the raw evidence artifact.

---

## 17. Roadmap / Control Implication

This action establishes:

```text
T4.5 TECHNICAL STATE
=
PASS
```

It does **not** automatically establish:

```text
T4.5 CONTROL CLOSED
```

It does **not** authorize:

```text
T4.6
```

It does **not** modify SET5 or historical SET2 state.

The next control decision is a separate Orchestrator action: reconcile/persist the
T4.5 technical acceptance and determine whether the T4.5 control state can be closed.

---

## 18. Final Decision

```text
TECHNICAL MODEL
= DERIVED

TECHNICAL ACCEPTANCE
= PASS

RUNTIME STATE DTYPE
= UNKNOWN LOCALLY

RUNTIME ALLOCATION
= UNKNOWN LOCALLY

MTP CONTRIBUTION
= UNKNOWN / CONDITIONAL

CURRENT SET
= SET4

T4.6
= NOT STARTED / NOT AUTHORIZED

SET5
= NOT STARTED / NOT AUTHORIZED

SET2
= HISTORICAL ONLY
```

The Orchestrator STOP condition for this task is satisfied:

```text
T4.5 TECHNICAL MODEL DERIVED
+
TECHNICAL ACCEPTANCE DETERMINED
+
DOCUMENTATION CREATED
+
CURRENT CONTROL IMPLICATION DETERMINED
```
