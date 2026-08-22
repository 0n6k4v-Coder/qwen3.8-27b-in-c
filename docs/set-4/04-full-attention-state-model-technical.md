# SET 4 — Full-Attention State Model — Orchestrator Technical Model

## Document Status

- **Document:** `docs/set-4/04-full-attention-state-model-technical.md`
- **SET:** `SET 4 — Runtime Memory Model`
- **Source Task:** `SET4-T4.4`
- **Role:** ORCHESTRATOR technical reasoning / derivation / acceptance
- **Status:** **TECHNICAL PASS — CONTROL CLOSURE PENDING**
- **Dependency:** `SET4-T4.3 PASS`
- **Raw Executor Evidence:** `docs/set-4/04-full-attention-state-model.md`
- **Model revision:** `Qwen/Qwen3.8-27B`, pinned upstream revision `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

---

## 1. Purpose and Responsibility Boundary

This document is the Orchestrator-owned technical model derived from the persisted
Executor evidence in:

```text
docs/set-4/04-full-attention-state-model.md
```

The Executor artifact remains the raw evidence record. It is not rewritten or
reclassified by this document.

This document performs:

```text
EVIDENCE
→
VERIFICATION
→
CLASSIFICATION
→
DERIVATION
→
DIMENSIONAL CHECK
→
TECHNICAL INTERPRETATION
→
TECHNICAL ACCEPTANCE
```

This document does not establish runtime allocator behavior that is absent from the
available evidence.

Critical distinction:

```text
STRUCTURAL KV-STATE MODEL
≠
RUNTIME KV-ALLOCATOR IMPLEMENTATION
```

and:

```text
KV STATE MEMORY
≠
TOTAL RUNTIME MEMORY
```

---

## 2. Authoritative Inputs

The following structural values are established from accepted project evidence.

| Variable | Value | Classification | Meaning |
|---|---:|---|---|
| `L_FA` | 16 | VERIFIED FACT | Number of language-model full-attention layers |
| `H_Q` | 24 | VERIFIED FACT | Query heads per full-attention layer |
| `H_KV` | 4 | VERIFIED FACT | Key/value heads per full-attention layer |
| `D` | 256 | VERIFIED FACT | Head dimension |
| `G` | 6 | DERIVED FINDING | GQA grouping ratio `H_Q / H_KV = 24 / 4` |
| checkpoint dtype | BF16 | VERIFIED FACT | Checkpoint tensor dtype |
| BF16 element size | 2 bytes | DERIVED FINDING | Bytes per BF16 element |
| `use_cache` | `true` | VERIFIED FACT | Configuration field only |
| `B` | UNKNOWN | UNKNOWN | Runtime batch size |
| `S` | UNKNOWN | UNKNOWN | Runtime sequence length |
| `E_state` | UNKNOWN | UNKNOWN | Runtime bytes per KV-cache element |

The Executor evidence also establishes that the actual runtime KV-cache dtype,
allocation strategy, layout, preallocation policy, quantization state, allocator reuse,
and runtime effect of `use_cache` were not observed.

---

## 3. Full-Attention State Mechanism

Accepted structural evidence establishes that full-attention uses a KV-cache state
mechanism rather than the recurrent/convolutional state model used by the linear-attention
layers.

The full-attention layer set contains 16 layers.

The cache therefore has a K state and a V state for each of those 16 full-attention
layers.

The base T4.4 model intentionally excludes the MTP self-attention contribution because
MTP runtime execution remains UNKNOWN in the accepted evidence.

Thus:

```text
BASE T4.4 STATE DOMAIN
=
16 language-model full-attention layers
×
K state
×
V state
```

MTP state is treated separately as:

```text
MTP CONTRIBUTION = UNKNOWN / CONDITIONAL
```

---

## 4. First-Principles Derivation

### 4.1 K-State Elements Per Layer

For one full-attention layer, the K state has:

```text
B batches
×
S tokens
×
H_KV key/value heads
×
D elements/head
```

Therefore:

```text
K_elements_per_layer
=
B × S × H_KV × D
```

Substitute the verified structural constants:

```text
H_KV = 4
D    = 256
```

Therefore:

```text
K_elements_per_layer
=
B × S × 4 × 256
=
1024 × B × S
```

Classification:

```text
DERIVED FINDING
```

---

### 4.2 V-State Elements Per Layer

The V state has the same structural extent:

```text
V_elements_per_layer
=
B × S × H_KV × D
```

Therefore:

```text
V_elements_per_layer
=
1024 × B × S
```

Classification:

```text
DERIVED FINDING
```

---

### 4.3 Combined K+V Elements Per Layer

The state contains both K and V:

```text
KV_elements_per_layer
=
K_elements_per_layer + V_elements_per_layer
```

Therefore:

```text
KV_elements_per_layer
=
2 × B × S × H_KV × D
```

Substitution:

```text
=
2 × B × S × 4 × 256
```

Therefore:

```text
KV_elements_per_layer
=
2048 × B × S
```

Classification:

```text
DERIVED FINDING
```

---

### 4.4 Total K+V Elements Across All Full-Attention Layers

There are:

```text
L_FA = 16
```

full-attention layers.

Therefore:

```text
M_KV_elements
=
L_FA × KV_elements_per_layer
```

Substitution:

```text
M_KV_elements
=
16 × 2048 × B × S
```

Therefore:

```text
M_KV_elements
=
32768 × B × S
```

Equivalent first-principles form:

```text
M_KV_elements
=
2 × L_FA × B × S × H_KV × D
```

with:

```text
L_FA = 16
H_KV = 4
D    = 256
```

Classification:

```text
DERIVED FINDING
```

---

## 5. Runtime KV-State Byte Model

Let:

```text
E_state = runtime bytes per KV-cache element
```

The evidence does not establish `E_state` at runtime.

Therefore the authoritative conditional byte model is:

```text
M_KV_bytes
=
M_KV_elements × E_state
```

Substitute the derived element count:

```text
M_KV_bytes
=
32768 × B × S × E_state
```

This is the principal T4.4 structural memory equation.

Classification:

```text
CONDITIONAL MODEL
```

because the runtime quantities `B`, `S`, and `E_state` are not established as fixed
runtime observations.

---

## 6. Dimensional Validation

The underlying dimensional product is:

```text
layers
×
batch
×
tokens
×
KV heads
×
(elements/head)
×
(bytes/element)
```

Therefore:

```text
L_FA × B × S × H_KV × D × E_state
=
bytes
```

The factor of `2` accounts for the two state families:

```text
2 = K + V
```

Full dimensional form:

```text
2
× [layers]
× [batch]
× [tokens]
× [KV heads/layer]
× [elements/head]
× [bytes/element]
=
[bytes]
```

Dimensional validation:

```text
PASS
```

---

## 7. Conditional BF16 Case

The checkpoint is BF16, but the runtime KV-cache dtype is UNKNOWN.

Therefore the following is a **CONDITIONAL MODEL**, not a runtime observation:

```text
E_state = 2 bytes
```

Substitution into the byte model:

```text
M_KV_bytes(BF16 conditional)
=
32768 × B × S × 2
```

Therefore:

```text
M_KV_bytes(BF16 conditional)
=
65536 × B × S bytes
```

For one batch element:

```text
B = 1
```

then:

```text
M_KV_bytes
=
65536 × S bytes
```

Thus the conditional BF16 KV-state scaling is:

```text
64 KiB per sequence token per batch element
```

because:

```text
65536 bytes = 64 KiB
```

Classification:

```text
CONDITIONAL MODEL
```

It must not be reported as the observed runtime cache footprint.

---

## 8. Maximum-Position Conditional Analysis

The accepted configuration declares:

```text
max_position_embeddings = 262144
```

This value defines a model/configuration positional limit.

It does NOT establish:

```text
runtime cache preallocation = 262144 tokens
```

because preallocation strategy remains UNKNOWN.

A purely conditional BF16, `B=1`, full-length calculation is:

```text
M_KV_bytes
=
65536 × 262144
```

Therefore:

```text
M_KV_bytes
=
17,179,869,184 bytes
```

which is:

```text
16 GiB
```

Classification:

```text
CONDITIONAL MODEL
```

Interpretation:

```text
If
B = 1
AND
S = 262144
AND
E_state = 2 bytes
AND
all 16 full-attention KV states are resident simultaneously,

then the structural KV-state requirement is 16 GiB.
```

This does NOT prove that the runtime allocates 16 GiB, because runtime cache
preallocation and dtype remain UNKNOWN.

---

## 9. Scaling Properties

The derived structural model is linear in batch size and sequence length:

```text
M_KV_bytes ∝ B
M_KV_bytes ∝ S
```

Therefore:

```text
2 × batch
→
2 × KV state
```

and:

```text
2 × sequence length
→
2 × KV state
```

For fixed `B`, `S`, and layer geometry, the state coefficient is also linear in
runtime element size:

```text
M_KV_bytes ∝ E_state
```

Examples as conditional models for `B=1`:

```text
E_state = 1 byte
→ 32768 × S bytes
→ 32 KiB/token

E_state = 2 bytes
→ 65536 × S bytes
→ 64 KiB/token

E_state = 4 bytes
→ 131072 × S bytes
→ 128 KiB/token
```

These are parameterized scenarios only. They do not establish which runtime dtype is
actually used.

---

## 10. Runtime Unknown Register

The following remain UNKNOWN after T4.4:

| Unknown | Status | Control / Technical Effect |
|---|---|---|
| Runtime KV-cache dtype | UNKNOWN | Prevents promotion of BF16 case to runtime fact |
| Runtime batch size `B` | UNKNOWN | Kept as a model variable |
| Runtime sequence length `S` | UNKNOWN | Kept as a model variable |
| Cache allocation strategy | UNKNOWN | Does not alter the structural equation; affects actual allocation footprint |
| K/V physical layout | UNKNOWN | Does not alter logical state count; may affect allocator overhead |
| Preallocation policy | UNKNOWN | Prevents equating `max_position_embeddings` with allocated state |
| KV quantization | UNKNOWN | Prevents assuming reduced state size |
| Allocator reuse behavior | UNKNOWN | Relevant to physical peak, not logical state requirement |
| Runtime effect of `use_cache=true` | UNKNOWN | Config field is not runtime observation |
| MTP runtime KV-state contribution | UNKNOWN | Kept outside the base 16-layer equation |

These UNKNOWNs do not invalidate the structural T4.4 model because the model is
explicitly parameterized by the unresolved runtime quantities.

---

## 11. MTP Boundary

The base T4.4 equation covers the 16 language-model full-attention layers.

MTP has a self-attention tensor set, but its active runtime execution path was not
established by the accepted evidence.

Therefore:

```text
BASE T4.4 KV STATE
=
16 language-model full-attention layers
```

and:

```text
MTP KV STATE
=
UNKNOWN / CONDITIONAL
```

No MTP term is silently added to the accepted base equation.

---

## 12. Hardware Constraint Reconciliation

The accepted SET2/SET4 evidence identifies a constrained WSL2 environment and the
project's runtime memory model separates weight, activation, state, workspace, and
other execution memory domains.

The T4.4 equation therefore establishes only the KV-state component:

```text
M_total_runtime
≠
M_KV
```

More explicitly:

```text
M_total_runtime
=
M_weights
+
M_activations
+
M_full_attention_state
+
M_linear_attention_state
+
M_workspace
+
M_temporary
+
other runtime state
```

T4.4 establishes the full-attention state component only.

### Conditional capacity interpretation

Under the conditional BF16 model with `B=1`:

```text
M_KV = 64 KiB × S
```

Therefore a hypothetical 12 GiB memory budget would correspond to:

```text
12 GiB / 64 KiB = 196608 tokens
```

of KV state **only**, assuming all of that budget were available exclusively to the
full-attention KV state.

This is NOT an overall inference-fit result because weights, activations, linear state,
workspace, temporary buffers, allocator overhead, and other memory consumers also require
memory.

Classification:

```text
CONDITIONAL MODEL
```

No workload placement, scheduling, optimization, or memory-constrained execution
strategy is introduced here.

---

## 13. What T4.4 Establishes

T4.4 technically establishes:

```text
1. Full-attention state mechanism = KV cache
2. Number of full-attention layers = 16
3. KV heads/layer = 4
4. Head dimension = 256
5. K+V logical state elements = 32768 × B × S
6. KV logical bytes = 32768 × B × S × E_state
7. BF16 case = 65536 × B × S bytes (conditional)
8. State scaling is linear in B, S, and E_state
9. max_position_embeddings is a limit, not proof of preallocation
10. Runtime implementation unknowns remain explicit
```

---

## 14. What T4.4 Does Not Establish

T4.4 does not establish:

```text
runtime KV-cache dtype
actual cache allocation strategy
actual allocator overhead
actual cache layout
actual cache preallocation
actual runtime sequence length
actual runtime batch size
runtime quantization
runtime paging implementation
physical runtime memory consumption
peak total inference memory
overall model fit
throughput
latency
workload placement
scheduling
memory-constrained execution strategy
```

Those questions belong to later technical domains or require runtime evidence that is
not available here.

---

## 15. Technical Acceptance

The SET4-T4.4 technical acceptance criteria were evaluated independently.

| Criterion | Status | Basis |
|---|---|---|
| Full-attention state mechanism established | PASS | KV-cache mechanism established |
| Structural layer count established | PASS | 16 full-attention layers |
| KV heads established | PASS | 4 KV heads |
| Head dimension established | PASS | 256 |
| Parameterized KV-state equation derived | PASS | `32768 × B × S` elements |
| Units verified | PASS | Dimensional analysis resolves to bytes |
| Conditional dtype model established | PASS | `E_state` parameter + BF16 case |
| Runtime UNKNOWNs preserved | PASS | No unsupported promotion |
| Maximum-position semantics bounded | PASS | Limit explicitly separated from preallocation |
| Total-runtime-memory distinction preserved | PASS | KV state kept separate from other domains |
| Unsupported runtime allocation claim absent | PASS | Allocation strategy remains UNKNOWN |
| Unsupported performance claim absent | PASS | No throughput/latency claim |

Technical verdict:

```text
SET4-T4.4 TECHNICAL ACCEPTANCE = PASS
```

This is a **technical acceptance** decision. It does not by itself advance the control
plane to T4.5.

---

## 16. Control-Plane Implication

T4.4 technical work is now complete enough to satisfy the technical model objective:

```text
T4.4 TECHNICAL STATE = PASS
```

However:

```text
T4.4 TECHNICAL PASS
≠
T4.5 AUTHORIZATION
```

The current control task remains T4.4 until a separate Orchestrator control action
reconciles/persists the new technical acceptance and independently determines the
legitimate successor.

Therefore this action does NOT:

```text
start T4.5
authorize T4.5
modify SET5 state
reopen SET2
```

---

## 17. Provenance Boundary

The raw Executor evidence remains preserved at:

```text
docs/set-4/04-full-attention-state-model.md
```

This document contains only Orchestrator-owned reasoning and derived technical results.

That separation preserves:

```text
RAW EVIDENCE
≠
DERIVATION
≠
TECHNICAL ACCEPTANCE
≠
CONTROL AUTHORIZATION
```

---

## 18. Final Technical Decision

```text
TECHNICAL MODEL:
PASS

DIMENSIONAL VALIDATION:
PASS

CONDITIONAL BF16 MODEL:
VALID

UNKNOWN HANDLING:
PASS

HARDWARE RECONCILIATION:
CONDITIONAL / NON-BLOCKING

T4.4 TECHNICAL ACCEPTANCE:
PASS

T4.4 CONTROL CLOSURE:
PENDING SEPARATE ORCHESTRATOR ACTION

T4.5:
NOT STARTED
```
