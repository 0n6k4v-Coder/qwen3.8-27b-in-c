# SET 4 — Workspace / Buffer Model — Orchestrator Technical Model

## Document Status

- **Document:** `docs/set-4/06-workspace-buffer-model-technical.md`
- **SET:** `SET 4 — Runtime Memory Model`
- **Source Task:** `SET4-T4.6`
- **Role:** ORCHESTRATOR technical reasoning / derivation / calculation / acceptance
- **Status:** **TECHNICAL PASS — CONTROL STATE UNCHANGED**
- **Dependency:** `SET4-T4.5 PASS / CONTROL CLOSED`
- **Executor Evidence:** `docs/set-4/06-workspace-buffer-model.md`
- **Model:** `Qwen/Qwen3.8-27B`

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

## 1. Purpose and Boundary

T4.6 establishes the workspace / temporary-buffer memory domain required by the SET4 contract. T4.7 remains a separate task that combines all memory domains into total peak runtime memory. T4.8 remains a separate hardware-reconciliation task.

The model therefore keeps:

```text
WEIGHTS                 → T4.2
ACTIVATIONS             → T4.3
FULL-ATTENTION STATE    → T4.4
LINEAR-ATTENTION STATE  → T4.5
WORKSPACE / TEMPORARY   → T4.6
TOTAL PEAK MEMORY       → T4.7
HARDWARE RECONCILIATION → T4.8
```

The persisted Executor evidence establishes 17 candidate workspace/transient objects, their provenance, conditional shapes, and UNKNOWN runtime behavior. The present technical model reclassifies objects where needed so T4.6 does not double-count T4.3 activation/output objects or persistent weights.

## 2. Inventory Validation and Classification Correction

### 2.1 T4.6-relevant objects

The technical workspace domain contains these classes:

```text
A. Full-attention transient/workspace candidates
   RM-016  RoPE buffers
   RM-031  QK product
   RM-032  softmax output
   RM-033  weighted sum
   RM-039  attention mask
   RM-041  RoPE frequency/cache table
   RM-045  causal-attention workspace

B. Linear-attention transient/workspace candidates
   RM-043  per-layer workspace
   RM-044  gated-delta intermediate buffers

C. Conditional/global infrastructure
   RM-046  mmap checkpoint view
   RM-047  streaming buffers

D. Boundary objects, not independently added to T4.6 workspace capacity
   RM-015  Q/K norm outputs → T4.3 activation domain
   RM-017  attention output gate → activation/runtime detail
   RM-037  input_ids → input domain
   RM-038  position_ids → input domain
   RM-040  output hidden states → activation/output domain
   RM-042  generation_config → runtime metadata
```

### 2.2 RM-015 correction

The SET4-01 downstream table labels RM-015 as “RMSNorm weights”, while the detailed evidence artifact correctly describes runtime `q_norm_out` / `k_norm_out` with shapes `[B,24,S,256]` and `[B,4,S,256]` and classifies them as `ACTIVATION / TRANSIENT`.

The checkpoint norm weights themselves are persistent parameter tensors and belong to T4.2, not T4.6.

Therefore RM-015 is **not independently added to the T4.6 workspace peak**. This prevents double-counting a T4.3 activation domain and a persistent weight domain.

## 3. Authoritative Structural Inputs

| Variable | Value | Classification |
|---|---:|---|
| `L_FA` | 16 | VERIFIED FACT |
| `L_LA` | 48 | VERIFIED FACT |
| `H_Q` | 24 | VERIFIED FACT |
| `H_KV` | 4 | VERIFIED FACT |
| `D_head` | 256 | VERIFIED FACT |
| `D_rot` | 64 | DERIVED FINDING: `256 × 0.25` |
| `H_V` | 48 | VERIFIED FACT |
| `D_K` | 128 | VERIFIED FACT |
| `D_V` | 128 | VERIFIED FACT |
| `K_conv` | 4 | VERIFIED FACT |
| `B` | UNKNOWN | runtime input |
| `S` | UNKNOWN | runtime input |
| `E_*` | UNKNOWN | runtime dtype / element size by object |
| kernel fusion | UNKNOWN | implementation behavior |
| allocator reuse | UNKNOWN | implementation behavior |

The accepted operator model establishes full-attention Q/K/V projection dimensions, GQA organization, the LA/FA topology, and the LA head dimensions. The project contains no executed local inference engine; therefore runtime B, S, allocator policy, fusion, and live dtype remain UNKNOWN.

## 4. Lifetime / Reuse Model

### 4.1 Layer sequencing

The accepted topology is:

```text
[LA → LA → LA → FA] × 16
```

Each decoder layer processes one token-mixing branch followed by the MLP. Therefore workspaces belonging solely to one layer do not structurally coexist with the workspace of another layer merely because multiple layers exist in the network.

**Derived finding:** a workspace pool can be logically reusable across the 48 LA layers and 16 FA layers when the execution is sequential by layer.

This is a **logical reuse property of the computation graph**, not proof that a particular runtime allocator actually reuses memory.

### 4.2 Full-attention lifetime classes

| Object | Lifetime | Structural reuse |
|---|---|---|
| RM-016 RoPE buffer | per sequence / per attention invocation; may be cached globally | CONDITIONAL |
| RM-031 QK product | within one causal-attention invocation | CONDITIONAL |
| RM-032 softmax output | within one causal-attention invocation | CONDITIONAL |
| RM-033 weighted sum | within one causal-attention invocation | CONDITIONAL |
| RM-039 mask | per attention invocation or implicit | CONDITIONAL |
| RM-041 RoPE table | global/static if precomputed; transient if generated on demand | CONDITIONAL |
| RM-045 attention workspace | per full-attention layer invocation | CONDITIONAL |

RM-031 and RM-032 represent different stages of the same attention computation. Whether they can share storage or must coexist is implementation-dependent. Therefore two valid conditional cases exist rather than an invented single allocator behavior.

### 4.3 Linear-attention lifetime classes

| Object | Lifetime | Structural reuse |
|---|---|---|
| RM-043 per-layer workspace | one LA-layer invocation | CONDITIONAL |
| RM-044 gated-delta intermediates | one LA-layer invocation | CONDITIONAL / algorithm-dependent |

The exact shape/count of RM-044 remains UNKNOWN because the precise implementation-level gated-delta algorithm is unresolved in local evidence.

### 4.4 Cross-layer reuse

Let:

```text
W_FA_layer = logical peak workspace for one FA layer
W_LA_layer = logical peak workspace for one LA layer
```

Then the **logical reusable per-layer capacity** is:

```text
W_layer_reusable = max(W_FA_layer, W_LA_layer)
```

not:

```text
16 × W_FA_layer + 48 × W_LA_layer
```

because the layer computations are sequential rather than concurrent.

This statement is a structural model, not a claim about allocator retention.

## 5. Full-Attention Workspace Model

### 5.1 Q/K normalization transient

Inputs:

```text
B
S
H_Q = 24
H_KV = 4
D_head = 256
E_qn = runtime bytes/element
```

Formula:

```text
M_QKNORM
= B × S × (24 + 4) × 256 × E_qn
= B × S × 7,168 × E_qn
```

This is the size of materialized Q/K-norm outputs if they are separate buffers. It is not added to T4.6 peak because the object is classified in the T4.3 activation domain.

### 5.2 QK product

Conditional materialized case:

```text
M_QK
= B × 24 × S² × E_QK
```

Classification: CONDITIONAL MODEL.

### 5.3 Softmax output

Conditional materialized case:

```text
M_SM
= B × 24 × S² × E_SM
```

Classification: CONDITIONAL MODEL.

### 5.4 Weighted sum

Conditional materialized case:

```text
M_WS
= B × 24 × S × 256 × E_WS
= B × S × 6,144 × E_WS
```

Classification: CONDITIONAL MODEL.

### 5.5 Attention mask

Conditional materialized case:

```text
M_MASK
= B × S² × E_MASK
```

If the causal mask is applied implicitly by the attention kernel:

```text
M_MASK = 0
```

is a **conditional implementation case**, not an observed runtime fact.

### 5.6 RoPE

Conditional per-sequence table:

```text
M_ROPE(S)
= S × 64 × E_ROPE
```

Conditional maximum-position BF16 table:

```text
262,144 × 64 × 2
= 33,554,432 bytes
≈ 32 MiB
```

The 32 MiB result is only a structural BF16 reference case. Actual runtime materialization, dtype, and MRoPE layout remain UNKNOWN.

## 6. Conditional Full-Attention Peak Cases

### Case FA-A — Separate materialized intermediates

When QK, softmax, weighted sum, and mask are all materialized and live simultaneously:

```text
W_FA_materialized
=
M_QK + M_SM + M_WS + M_MASK
+ M_ROPE
+ W_FA_extra
```

where `W_FA_extra` covers any additional implementation workspace not represented by the named objects.

### Case FA-B — QK/softmax storage reuse

If QK storage is reused by the softmax stage:

```text
W_FA_reuse_qk
=
max(M_QK, M_SM) + M_WS + M_MASK + M_ROPE + W_FA_extra
```

### Case FA-C — Fused/block attention

If QK and softmax are not materialized as full `[B,H,S,S]` buffers:

```text
W_FA_fused
=
M_WS_tile + M_MASK_impl + M_ROPE + W_FA_extra
```

where `M_WS_tile` and `M_MASK_impl` are implementation-dependent and therefore UNKNOWN.

These cases are mutually conditional. No one case is promoted to a runtime fact.

## 7. Linear-Attention Workspace Model

The accepted structure establishes a sequential LA layer computation but does not resolve the exact gated-delta intermediate algorithm.

Define:

```text
W_LA_layer
=
W_LA_structural
+
W_LA_gated_delta
```

where:

```text
W_LA_structural = conditional workspace for verified projections/convolution/output flow
W_LA_gated_delta = UNKNOWN / algorithm-dependent
```

RM-044 is therefore deliberately retained as an UNKNOWN-shaped conditional buffer rather than assigned an invented tensor shape.

Because 48 LA layers are sequential:

```text
logical LA workspace accumulation
≠
48 × physical peak workspace
```

The logical reusable capacity is one LA-layer peak, subject to allocator behavior.

## 8. Global / Static Workspace

Potential global workspace includes RM-041 if a RoPE frequency/cos/sin table is precomputed and retained.

Define:

```text
W_GLOBAL
=
W_ROPE_GLOBAL + W_OTHER_GLOBAL
```

with:

```text
W_ROPE_GLOBAL ∈ {0, S×64×E_ROPE, 262144×64×E_ROPE, other implementation-specific form}
```

The exact runtime choice remains UNKNOWN.

RM-046 mmap and RM-047 streaming buffers are not assigned a numeric T4.6 peak because their runtime policy is UNKNOWN and their physical residency is part of the separate weight-residency/runtime implementation boundary.

## 9. Workspace Composition Model

The T4.6 workspace domain is therefore represented by:

```text
W_WORKSPACE
=
W_GLOBAL
+
W_LAYER_REUSABLE
+
W_CONDITIONAL_INFRA
```

with:

```text
W_LAYER_REUSABLE
=
max(W_FA_layer, W_LA_layer)
```

and:

```text
W_FA_layer ∈ {W_FA_materialized,
              W_FA_reuse_qk,
              W_FA_fused}
```

and:

```text
W_LA_layer
=
W_LA_structural + W_LA_gated_delta
```

`W_CONDITIONAL_INFRA` remains UNKNOWN unless an explicit runtime policy is established for mmap/streaming buffers.

### Critical boundary

```text
W_WORKSPACE
≠
TOTAL PEAK RUNTIME MEMORY
```

T4.7 will combine this workspace model with:

```text
weights
activations
full-attention state
linear-attention state
workspace
other temporary execution buffers
```

## 10. Numerical Reference Cases

These are conditional reference points only.

### Case 1 — QK + softmax, BF16, B=1, S=2048

```text
M_QK
= 1 × 24 × 2048² × 2
= 201,326,592 bytes
≈ 192 MiB

M_SM
= 201,326,592 bytes
≈ 192 MiB

M_QK + M_SM
= 402,653,184 bytes
≈ 384 MiB
```

### Case 2 — Weighted sum, BF16, B=1, S=2048

```text
M_WS
= 1 × 24 × 2048 × 256 × 2
= 25,165,824 bytes
≈ 24 MiB
```

### Case 3 — causal mask, BF16, B=1, S=2048

```text
M_MASK
= 1 × 2048² × 2
= 8,388,608 bytes
≈ 8 MiB
```

### Case 4 — maximum-position RoPE table, BF16

```text
M_ROPE_MAX
= 262144 × 64 × 2
= 33,554,432 bytes
≈ 32 MiB
```

These values are not runtime measurements and do not establish that the corresponding buffers coexist.

## 11. Unknown Register

The following remain explicitly UNKNOWN:

```text
UK-003  kernel fusion / materialization / allocator reuse
UK-004  runtime dtype
UK-006  exact attention output-gate tensor/runtime footprint
UK-008  MTP runtime execution
UK-009  runtime B and S
UK-013  weight streaming / paging strategy and buffer residency
UQ-006  exact MRoPE section layout
UK-001  exact gated-delta implementation / intermediate state shape
UK-010  softmax scaling implementation detail
```

None is silently converted into a runtime fact.

## 12. Acceptance Criteria Matrix

| Criterion | Status | Rationale |
|---|---|---|
| Workspace domain complete enough for T4.6 | PASS | material workspace/transient candidates are inventoried and boundaries are explicit |
| Persistent/stateful memory separated | PASS | T4.2/T4.4/T4.5 domains are excluded from T4.6 peak |
| Provenance-backed material objects | PASS | every included object traces to SET3/SET0/SET4 evidence |
| Lifetime relationships modeled where derivable | PASS | per-layer/per-invocation/global lifetime classes established |
| Reuse relationships modeled where derivable | PASS | sequential cross-layer logical reuse and conditional intra-stage reuse are explicit |
| Peak workspace parameterized/bounded where derivable | PASS | reusable per-layer maximum plus conditional FA cases and global workspace model are explicit |
| Unsupported runtime behavior avoided | PASS | runtime allocator, fusion, dtype, B/S remain UNKNOWN |
| Conditional cases remain conditional | PASS | FA-A/FA-B/FA-C and RoPE alternatives are explicitly conditional |
| T4.6 technical acceptance decision explicit | PASS | Orchestrator technical verdict below |
| T4.7 remains separate | PASS | no total-memory composition performed |
| T4.8 remains separate | PASS | no hardware reconciliation performed |
| SET5 untouched | PASS | no SET5 implementation or runtime work performed |

## 13. Technical Acceptance

```text
T4.6 TECHNICAL ACCEPTANCE = PASS
```

### Rationale

T4.6 requires a workspace/buffer memory model with temporary/execution-buffer requirements, and specifically asks for lifetime, reuse, and peak requirements **where derivable**. The available evidence supports a parameterized model for those quantities without requiring unsupported runtime claims.

The model establishes:

```text
1. workspace-domain boundaries
2. object provenance
3. per-stage lifetime classes
4. structural sequential reuse across layers
5. conditional intra-attention reuse/fusion cases
6. a parameterized workspace composition
7. explicit conditional/UNKNOWN implementation boundaries
```

The absence of a live runtime allocator does **not** by itself prevent technical acceptance because the contract explicitly permits conditional and parameterized memory models where runtime behavior is unresolved.

### Acceptance limitation

`PASS` means the **technical model required by T4.6 is complete as a conditional/parameterized model**. It does not mean that the actual runtime allocator, runtime peak, or physical resident memory has been observed.

## 14. Control Impact

This document provides technical acceptance only.

It does **not** change:

```text
ROADMAP
T4.6 control closure
T4.7 authorization
T4.8 authorization
SET4 closure
SET5 authorization
```

The current persisted ROADMAP therefore remains the active control state until a separate Orchestrator control-closure action is performed.

## 15. Final Technical Conclusion

```text
T4.6 WORKSPACE / BUFFER MODEL
= TECHNICALLY ACCEPTED

MODEL TYPE:
PARAMETERIZED + CONDITIONAL

RUNTIME OBSERVATION:
NONE

UNKNOWNs:
PRESERVED

T4.7:
DOWNSTREAM / SEPARATE

T4.8:
DOWNSTREAM / SEPARATE

SET5:
UNTOUCHED
```
