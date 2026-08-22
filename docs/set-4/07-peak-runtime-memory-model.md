# SET 4 — Peak Runtime Memory Model — Orchestrator Technical Model

## Document Status

- **Document:** `docs/set-4/07-peak-runtime-memory-model.md`
- **SET:** `SET 4 — Runtime Memory Model`
- **Source Task:** `SET4-T4.7`
- **Role:** ORCHESTRATOR technical reasoning / derivation / calculation / acceptance
- **Status:** **TECHNICAL PASS — CONTROL STATE UNCHANGED**
- **Dependency:** `SET4-T4.6 PASS / CONTROL CLOSED`
- **Model:** `Qwen/Qwen3.8-27B`

```text
EVIDENCE
≠
DERIVATION
≠
CONDITIONAL RUNTIME MODEL
≠
TECHNICAL ACCEPTANCE
≠
CONTROL AUTHORIZATION
```

## 1. Purpose and Boundary

T4.7 combines the accepted SET4 memory domains into a parameterized peak-runtime-memory model.

The SET4 contract requires the model to distinguish at minimum:

```text
WEIGHT MEMORY
ACTIVATION MEMORY
FULL-ATTENTION STATE
LINEAR-ATTENTION STATE
WORKSPACE MEMORY
TEMPORARY / EXECUTION BUFFERS
PEAK RUNTIME MEMORY
UNKNOWN / CONDITIONAL MEMORY
```

T4.8 remains separate and performs hardware-constraint reconciliation. T4.7 does not establish physical hardware fit.

## 2. Accepted Upstream Inputs

### 2.1 Weight residency domain — T4.2

The authoritative checkpoint logical-weight total is:

```text
W_logical = 55,562,855,904 bytes
```

This is checkpoint logical tensor bytes, not runtime-resident bytes.

T4.2 models runtime residency as conditional on unresolved streaming/paging/loading policy. For each logical weight category `c`:

```text
W_resident(c) = L_c × rho_c
```

where `rho_c` is the unresolved runtime-resident fraction. Runtime transformation/duplication may add an additional conditional term.

The authoritative aggregate subsystem totals are:

```text
Language model core      = 48,706,403,328 bytes
Vision                   =    921,460,192 bytes
Language embedding       =  2,542,796,800 bytes
LM head                  =  2,542,796,800 bytes
MTP                      =    849,398,784 bytes
----------------------------------------------
Global                   = 55,562,855,904 bytes
```

T4.2 also records that the per-family decomposition must defer to the SET1 subsystem reconciliation for aggregate totals. T4.7 therefore uses the SET1-authoritative aggregate totals and does not recompute the inconsistent per-family subtotal.

Classification: `VERIFIED FACT` for logical totals; `CONDITIONAL MODEL` for runtime residency.

### 2.2 Activation domain — T4.3

T4.3 establishes parameterized activation shapes and phase/lifetime semantics for `B` and `S`, with runtime computation dtype unresolved.

The important T4.3 structural upper bound is:

```text
A_peak_language = max(
    A_fa_layer_max,
    A_la_layer_max,
    A_embed_final_max
)
```

under a no-fusion, sequential-layer structural assumption.

However, T4.3 includes QK, softmax, weighted-sum, causal-mask, RoPE and other transient objects in its activation-side upper bounds. Those same objects are explicitly in the T4.6 workspace/transient domain. Therefore T4.7 MUST NOT add the raw T4.3 upper bound directly to T4.6 workspace.

This creates an **exclusive-domain decomposition** for T4.7.

Classification: `DERIVED FINDING`.

### 2.3 Full-attention state — T4.4

Accepted structural model:

```text
M_KV_elements = 32,768 × B × S
```

and therefore:

```text
M_KV = 32,768 × B × S × E_KV
```

where `E_KV` is the unresolved runtime bytes/element.

Conditional BF16 reference case:

```text
E_KV = 2
M_KV = 65,536 × B × S bytes
```

Classification: `CONDITIONAL MODEL`.

### 2.4 Linear-attention state — T4.5

Accepted structural model:

```text
M_recurrent = 37,748,736 × B × E_r
M_conv      =  1,474,560 × B × E_c
```

Therefore:

```text
M_LA_state
=
B × (37,748,736 × E_r + 1,474,560 × E_c)
```

where `E_r` and `E_c` are unresolved runtime bytes/element.

Classification: `CONDITIONAL MODEL`.

### 2.5 Workspace domain — T4.6

Accepted T4.6 model:

```text
W_workspace
=
W_global
+
W_layer_reusable
+
W_conditional_infra
```

with:

```text
W_layer_reusable = max(W_FA_layer, W_LA_layer)
```

and conditional full-attention cases:

```text
W_FA_materialized = M_QK + M_SM + M_WS + M_MASK + M_ROPE + W_FA_extra
W_FA_reuse_qk     = max(M_QK, M_SM) + M_WS + M_MASK + M_ROPE + W_FA_extra
W_FA_fused        = M_WS_tile + M_MASK_impl + M_ROPE + W_FA_extra
```

Linear-attention workspace:

```text
W_LA_layer = W_LA_structural + W_LA_gated_delta
```

where `W_LA_gated_delta` remains UNKNOWN / algorithm-dependent.

Classification: `CONDITIONAL MODEL`.

## 3. Exclusive Memory-Domain Partition

To prevent double-counting, T4.7 assigns each material object to exactly one additive domain.

### Domain A — Runtime-resident weights

```text
W_resident
```

Includes runtime-resident model-weight bytes and any explicitly demonstrated runtime representation conversion overhead.

Does NOT automatically include mmap/streaming infrastructure unless that infrastructure is shown to be an additional live allocation.

### Domain B — Persistent attention state

```text
M_KV + M_LA_state
```

Includes:
- full-attention KV state from T4.4
- recurrent + convolution state from T4.5

### Domain C — Exclusive activation memory

This domain excludes objects assigned to T4.6 workspace/transient accounting.

The language-layer structural activation components retained in C are:

```text
Full-attention core activation:
A_FA_core
=
B×S×30,208×E
+
A_MLP(B,S,E)

Linear-attention core activation:
A_LA_core
=
B×S×26,624×E
+
A_gdelta(B,S,E)
+
A_MLP(B,S,E)
```

where:

```text
A_MLP(B,S,E)
=
B×S×72,432×E
```

for the no-fusion structural upper-bound form, with the SiLU element size parameterized when runtime dtype is unresolved.

Thus, under a BF16 reference case and no fusion:

```text
A_FA_core_BF16
=
B×S×205,072 bytes
```

before unresolved gated/other implementation-specific additions.

More generally the authoritative T4.7 expression keeps `E`, `E_silu`, `E_qk`, `E_sm`, and other runtime widths symbolic rather than replacing them with observed values.

### Domain D — Workspace / execution buffers

```text
W_workspace
```

This includes QK, softmax, weighted-sum, causal-mask, RoPE, LA workspace, gated-delta buffers, and the other T4.6 workspace candidates when materialized.

### Domain E — Request I/O / conditional subsystem memory

```text
M_IO
+
M_vision_cond
+
M_MTP_cond
```

These domains are conditional and only contribute when their corresponding execution path is active or materialized.

## 4. Lifetime and Coexistence Rule

The key peak-memory property is:

```text
persistent domains
+
maximum simultaneously-live execution phase
```

not the cumulative sum of every object across all 64 layers.

Under sequential layer execution:

```text
T4.2 weights
+
T4.4 KV state
+
T4.5 LA state
+
phase-local activations
+
phase-local workspace
```

may coexist.

But workspaces belonging exclusively to different decoder layers do not structurally require simultaneous allocation:

```text
W_layer_reusable = max(W_FA_layer, W_LA_layer)
```

This is a structural reuse finding, not a claim about physical allocator reuse.

## 5. General Peak Runtime Memory Formula

Define:

```text
M_persistent
=
W_resident
+
M_KV
+
M_LA_state
```

and define phase-local execution memory:

```text
P_FA
=
A_FA_core_exclusive
+
W_FA_layer
+
M_IO_FA

P_LA
=
A_LA_core_exclusive
+
W_LA_layer
+
M_IO_LA

P_FINAL
=
A_final_exclusive
+
W_final
+
M_IO_final

P_VISION
=
A_vision_cond
+
W_vision_cond

P_MTP
=
A_MTP_cond
+
W_MTP_cond
```

Then the parameterized peak model is:

```text
M_peak
=
M_persistent
+
W_global
+
max(
    P_FA,
    P_LA,
    P_FINAL,
    P_VISION,
    P_MTP
)
+
M_conditional_infrastructure
```

with the critical requirement that no object appears in more than one additive domain.

An equivalent expanded form is:

```text
M_peak
=
W_resident
+
M_KV
+
M_LA_state
+
W_global
+
max(P_FA, P_LA, P_FINAL, P_VISION, P_MTP)
+
M_conditional_infrastructure
```

Classification: `CONDITIONAL MODEL`.

## 6. Full-Attention Peak Phase

The full-attention phase must preserve the distinction between activation-core memory and T4.6 workspace.

### Case FA-A — all named attention workspace buffers materialized

```text
P_FA_A
=
A_FA_core_exclusive
+
M_QK
+
M_SM
+
M_WS
+
M_MASK
+
M_ROPE
+
W_FA_extra
+
M_IO_FA
```

where:

```text
M_QK   = B×24×S²×E_QK
M_SM   = B×24×S²×E_SM
M_WS   = B×24×S×256×E_WS
M_MASK = B×S²×E_MASK
```

### Case FA-B — QK/softmax storage reuse

```text
P_FA_B
=
A_FA_core_exclusive
+
max(M_QK, M_SM)
+
M_WS
+
M_MASK
+
M_ROPE
+
W_FA_extra
+
M_IO_FA
```

### Case FA-C — fused/block attention

```text
P_FA_C
=
A_FA_core_exclusive
+
M_WS_tile
+
M_MASK_impl
+
M_ROPE
+
W_FA_extra
+
M_IO_FA
```

where `M_WS_tile` and `M_MASK_impl` remain implementation-dependent UNKNOWNs.

No case is a runtime observation.

## 7. Linear-Attention Peak Phase

```text
P_LA
=
A_LA_core_exclusive
+
W_LA_structural
+
W_LA_gated_delta
+
M_IO_LA
```

where:

```text
W_LA_gated_delta = UNKNOWN / algorithm-dependent
```

The 48 LA layers are not multiplied into a cumulative transient peak because they are sequential in the accepted topology:

```text
[LA → LA → LA → FA] × 16
```

Persistent T4.5 recurrent/conv state remains separately additive as `M_LA_state`.

## 8. Conditional Vision and MTP Phases

### Vision

Vision activation/residency is conditional on whether the vision path is invoked. Exact image resolution, token count, and activation shapes are unresolved.

```text
P_VISION = A_vision_cond + W_vision_cond
```

Classification: `CONDITIONAL MODEL`.

### MTP

MTP execution is UNKNOWN.

```text
P_MTP = A_MTP_cond + W_MTP_cond
```

If MTP is inactive:

```text
P_MTP = 0
```

If active, the contribution remains conditional on unresolved runtime shapes and scheduling.

Classification: `CONDITIONAL MODEL`.

## 9. Persistent Weight Residency and Streaming Boundary

T4.2 does not establish an always-resident weight policy. Therefore:

```text
W_resident
∈
[0, W_logical + transformation_overhead]
```

at the abstract model level, subject to the actual execution requirements of each operator and the unresolved residency policy.

The structural constraint:

```text
W_logical = 55,562,855,904 bytes ≈ 51.7 GiB
```

exceeds the accepted ~12 GB WSL2 guest-memory cap. Therefore the present environment cannot be assumed to keep the full logical checkpoint resident simultaneously. The actual mechanism remains UNKNOWN (streaming/paging/phased loading/device placement).

This is a structural constraint, not a T4.8 hardware-fit conclusion.

## 10. Dimensional Validation

Every additive term in `M_peak` has units of bytes.

Examples:

```text
B × S × hidden_size × bytes/element = bytes

B × H_Q × S² × bytes/element = bytes

L_FA × B × S × H_KV × D × bytes/element = bytes

L_LA × B × H_V × D_K × D_V × bytes/element = bytes
```

The outer `max()` selects among simultaneously possible phase-local byte totals without changing dimensions.

Dimensional audit:

```text
PASS
```

## 11. Double-Counting Audit

The following exclusions are mandatory:

```text
T4.3 AC-08 QK product
→ T4.6 workspace, not additive activation-core

T4.3 AC-09 softmax output
→ T4.6 workspace, not additive activation-core

T4.3 AC-10 weighted sum
→ T4.6 workspace, not additive activation-core

T4.3 AC-30 RoPE tables
→ T4.6 workspace/global workspace when materialized

T4.3 AC-33 causal mask
→ T4.6 workspace when materialized

T4.3 AC-35 gated-delta intermediates
→ T4.6 LA workspace when materialized

T4.3 AC-07 KV boundary
→ T4.4 state domain

T4.3 AC-36 convolution state boundary
→ T4.5 state domain

T4.2 persistent weights
→ W_resident only

T4.6 RM-015 Q/K norm outputs
→ T4.3 activation domain; not T4.6 workspace
```

This partition is required to make the T4.7 sum additive without double-counting.

## 12. Numerical Reference Cases

These are conditional reference cases only, not runtime observations.

### Case N1 — Full-attention KV, BF16, B=1, S=2048

```text
M_KV
=
65,536 × 2048
=
134,217,728 bytes
=
128 MiB
```

### Case N2 — Full-attention QK + softmax, BF16, B=1, S=2048

```text
M_QK = 201,326,592 bytes ≈ 192 MiB
M_SM = 201,326,592 bytes ≈ 192 MiB

M_QK + M_SM ≈ 384 MiB
```

### Case N3 — Linear-attention state, B=1, both state families conditionally FP32

```text
M_recurrent
=
37,748,736 × 4
=
150,994,944 bytes

M_conv
=
1,474,560 × 4
=
5,898,240 bytes

M_LA_state
=
156,893,184 bytes
≈
149.6 MiB
```

These values illustrate parameterized scaling only.

## 13. UNKNOWN / CONDITIONAL Register

The following remain UNKNOWN and are not silently resolved:

```text
UK-001  exact linear-attention algorithm / gated-delta intermediates
UK-002  runtime computation dtype for specific LA/intermediate operations
UK-003  kernel fusion / materialization / allocator reuse
UK-004  general runtime computation dtype
UK-005  exact RMSNorm placement
UK-006  exact residual topology / output-gate formulation
UK-007  vision invocation / multimodal fusion behavior
UK-008  MTP active runtime execution
UK-009  runtime B, S and related request dimensions
UK-010  attention scaling implementation detail
UK-011  full-attention KV allocation/paging strategy
UK-012  convolution-state runtime allocation details
UK-013  weight streaming/paging/residency strategy
UQ-006  MRoPE execution-path semantics and layout
```

Additional T4.7-level unknowns:

```text
UK-T47-001  exact co-residency of transformed weight buffers with W_resident
UK-T47-002  exact allocator bookkeeping/metadata overhead
UK-T47-003  exact overlap between request I/O buffers and phase-local activation buffers
UK-T47-004  exact subsystem concurrency if vision/MTP are active
```

These are explicitly represented as UNKNOWN/CONDITIONAL rather than promoted to runtime facts.

## 14. Technical Conclusion

The accepted SET4 evidence is sufficient to construct a parameterized peak-runtime-memory composition without asserting unobserved runtime behavior.

The technically authoritative form is:

```text
M_peak
=
W_resident
+
M_KV
+
M_LA_state
+
W_global
+
max(P_FA, P_LA, P_FINAL, P_VISION, P_MTP)
+
M_conditional_infrastructure
```

subject to the exclusive-domain partition and double-counting rules above.

This is a **parameterized / conditional technical model**, not a runtime peak measurement and not a hardware-fit result.

## 15. Acceptance Criteria Matrix

| Criterion | Status | Rationale |
|---|---|---|
| 1. All required SET4 memory domains identified | PASS | T4.2, T4.3, T4.4, T4.5 and T4.6 provide the required domains |
| 2. Memory domains not double-counted | PASS | Exclusive-domain partition explicitly resolves T4.3/T4.6 overlap |
| 3. Persistent/stateful vs transient separated | PASS | Weight, KV, LA state and execution phases remain distinct |
| 4. Simultaneous-live vs cumulative allocation distinguished | PASS | Phase-local max plus persistent domains; no cumulative layer sum |
| 5. Peak runtime memory parameterized/bounded where derivable | PASS | General formula plus FA-A/B/C conditional cases |
| 6. Implementation-dependent behavior remains conditional | PASS | Fusion, allocator, dtype, residency, vision/MTP remain conditional |
| 7. Runtime UNKNOWNs preserved | PASS | UNKNOWN register explicitly carried forward |
| 8. Independent Orchestrator derivation performed | PASS | Formulas and composition independently derived from accepted inputs |
| 9. Technical acceptance explicitly declared | PASS | T4.7 technical verdict below |
| 10. T4.8 separate and unexecuted | PASS | Hardware reconciliation intentionally excluded |
| 11. SET5 untouched/not authorized | PASS | No SET5 execution or control transition performed |
| 12. Historical SET2/SET3 records preserved | PASS | No historical mutation performed |
| 13. No unsupported runtime conclusion | PASS | Model remains parameterized/conditional |

## 16. Technical Acceptance

```text
T4.7 TECHNICAL ACCEPTANCE = PASS
```

Meaning:

```text
Peak runtime memory model = technically accepted
```

Not implied:

```text
runtime peak observed = PASS
hardware fit = PASS
allocator behavior verified = PASS
T4.8 = PASS
SET4 = CLOSED
SET5 = AUTHORIZED
```

The remaining UNKNOWNs are acceptable for T4.7 because their effects are preserved as explicit parameters, bounded/conditional cases, or UNKNOWNs without unsupported runtime assumptions.

## 17. Artifact Boundary

This artifact establishes T4.7 technical reasoning and technical acceptance.

It does not:

- modify ROADMAP control state;
- activate T4.8;
- perform T4.8 hardware reconciliation;
- start SET5;
- alter T4.6 evidence;
- claim runtime measurement.

## 18. Final Technical State

```text
CURRENT SET:
SET4

CURRENT TECHNICAL TASK:
SET4-T4.7 — Peak Runtime Memory Model

T4.7 TECHNICAL ACCEPTANCE:
PASS

T4.8:
NOT EXECUTED

SET5:
NOT STARTED / NOT AUTHORIZED

CONTROL PLANE:
UNCHANGED BY THIS TECHNICAL ACTION
```
