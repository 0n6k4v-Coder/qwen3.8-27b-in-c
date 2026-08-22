# SET 4 — Linear-Attention State Model — Evidence Acquisition

## Document Status

- Document: `docs/set-4/05-linear-attention-state-model.md`
- SET: `SET 4 — Runtime Memory Model`
- Source Task: `SET4-T4.5`
- Status: **EVIDENCE ACQUIRED (Executor — NON-AUTHORITATIVE — ORCHESTRATOR REVIEW REQUIRED)**
- Responsibility: Execution Support (Executor — evidence acquisition only)
- Control State: `SET4-T4.4 = PASS` (Dependency satisfied)
- Control Task Owner: EXECUTOR (evidence acquisition)
- Technical Design Owner: ORCHESTRATOR (model construction, formula construction, interpretation, acceptance)

This document is an EXECUTOR evidence artifact. It records raw observations, provenance,
classification, and the UNKNOWN register for the linear-attention / recurrent / convolution
state domain only.

RAW OBSERVATION ≠ INTERPRETATION ≠ TECHNICAL DESIGN ≠ TECHNICAL ACCEPTANCE ≠ CONTROL AUTHORIZATION

---

## 1. Source and Provenance

### 1.1 Authoritative Model

- Model: `Qwen/Qwen3.8-27B`
- Official repository: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`
- Commit HEAD (local): `71c7adc9c53cbd4c15862f735665cdc24e5034f8`

### 1.2 Primary Evidence Sources

All observations below are traced to the following authoritative project artifacts:

```text
SET 4 (direct input):
  docs/set-4/01-runtime-memory-inventory.md        — T4.1 runtime memory inventory
    §3.4 Linear-Attention State Memory (RM-019 recurrent state, RM-020 conv state)
    §3.10.3 RM-044 linear-attention gated-delta intermediates
    §4 SET3 UNKNOWN carry-forward register
    §5.2 Runtime memory truth domain taxonomy (STATEFUL: RM-019, RM-020)
    §6.1 Downstream dependency mapping for T4.5
  docs/set-4/02-weight-residency-model.md          — T4.2 weight residency (accepted input)
    §3.4 Linear-Attention Weights (RM-003): 9 tensors × 48 layers = 432 tensors
  docs/set-4/03-activation-lifetime-model.md       — T4.3 activation lifetime model
    §4 AC-14 linear-attention QKV projection output
    §4 AC-15 linear-attention Z projection output
    §4 AC-16 linear-attention output projection output
    §11.1 AC-35 linear-attention gated-delta intermediates (boundary)
    §11.2 AC-36 linear-attention convolution state boundary (boundary)
    §14.3 persistence category: AC-36 is ACTIVATION BOUNDARY
    §19.3 T4.5 consumption: AC-35, AC-36

SET 3 (operator / computation model — accepted input):
  docs/set-3/01-operator-computation-model.md     — OC-4, OC-10, dataflow §6.3, §9 unknowns
    §3.4 OC-4 Qwen3_5GatedDeltaNet: exact tensor set, shapes, coverage
    §3.4 state model distinction: recurrent state + convolution state
    §2.3 layer topology: 48 linear-attention layers
    §3.4 algorithm identity: UNKNOWN (UK-001)
    §6.3 linear-attention dataflow: full stage ordering
    §6.3 classification: tensor shapes VERIFIED, algorithm UNKNOWN
    §8 assumption #4: mamba_ssm_dtype metadata not tensor dtype
    §9 Unknowns (UK-001 through UK-015)
      UK-001: Exact linear-attention algorithm
      UK-002: Whether mamba_ssm_dtype=float32 implies runtime state in float32
      UK-012: Runtime linear-attention state allocation

SET 0 (structural truth):
  docs/set-0/03-core-architecture.md               — §4 linear-attention config fields
  docs/set-0/04-attention-architecture.md          — §6 Linear Attention config, §7 Implementation,
                                                     §8 Convolution, §9 State, §10 Gated Delta Rule,
                                                     §11 Comparison table, §13 topology diagram,
                                                     §15.1 two state models, §15.3 heterogeneous decode
  docs/set-0/06-vision-and-mtp.md                   — §4 MTP config (not part of LA state)
  docs/set-0/07-layer-topology.md                   — §2 topology, §4 FA indices, §5 LA indices, §11
  docs/set-0/08-tensor-shape-mapping.md             — §2 Linear Attention tensor shapes (48/48 coverage)

SET 1 (checkpoint storage truth):
  model/official/raw-checkpoint-metadata/config.json — raw config.json
  model/official/raw-checkpoint-metadata/manifest.json — acquisition metadata
  model/official/SOURCE.md                             — artifact source provenance

ROADMAP.md — SET4-T4.5 task contract (§2012–2064), SET4 agenda §2059–2064,
             output contract §2218, deliverables §2218, hard boundary §2229–2247
```

### 1.3 Classification Schema

Every material assertion in this document is classified as exactly one:

- VERIFIED FACT — directly supported by raw SET1 tensor metadata, SET0 configuration fields, or established SET3 operator/dataflow evidence.
- DOCUMENTED CAPABILITY — sourced from authoritative external documentation; not promoted to VERIFIED FACT.
- DERIVED FINDING — arithmetic or logical combination of verified evidence. Explicitly labeled.
- CONDITIONAL MODEL — a memory property expressed as an explicit dependency on an unresolved runtime behavior. Bounded, parameterized, not an assumption.
- UNKNOWN — runtime behavior or structural detail not established by available evidence. Treated as a boundary.

### 1.4 Critical Distinctions

```text
CONFIG-DERIVED STATE SHAPE ≠ RUNTIME STATE ALLOCATION STRATEGY ≠ RUNTIME STATE LAYOUT
STRUCTURAL TRUTH ≠ RUNTIME IMPLEMENTATION TRUTH
LINEAR-ATTENTION RECURRENT STATE + CONVOLUTION STATE ≠ FULL-ATTENTION KV CACHE
LINEAR-ATTENTION RECURRENT STATE + CONVOLUTION STATE ≠ WEIGHT RESIDENCY
LINEAR-ATTENTION RECURRENT STATE + CONVOLUTION STATE ≠ ACTIVATION MEMORY
```

This document establishes the evidence for LINEAR-ATTENTION state only, kept strictly
separate from:
- Full-attention KV cache state (T4.4 — accepted input, not modeled here)
- Weight residency (T4.2 — accepted input)
- Activation memory (T4.3 — accepted input, AC-36 is boundary only)
- Workspace memory (T4.6 — downstream)

### 1.5 Document Conventions (Following T4.4 Pattern)

The structure follows the established SET4 evidence pattern (T4.4 evidence artifact):
- Document Status → Source and Provenance → Classification Schema → Critical Distinctions
- Environment Inspection → Task Contract → Raw Structural Evidence → Raw Evidence → UNKNOWN Register → Evidence Summary → Executor Result → Unknown/Missing

### 1.6 T4.5 Task Contract (from ROADMAP.md §2012–2064)

```text
SET4-T4.5 — Linear-Attention State Model:
  Establish verified, bounded, or conditional linear-attention state memory.
  Determine which portions are exactly derivable from accepted evidence.
  Where the exact linear-attention algorithm or state representation remains UNKNOWN,
    produce bounded or conditional memory models rather than inventing an algorithm.
  Preserve UK-001 and UK-012 as UNKNOWN unless independently resolved by authoritative evidence.
  Conditional models MAY be used where memory behavior can be expressed safely
    as an explicit dependency on an unresolved algorithm/state representation.
```

### 1.7 Executor Responsibility Boundary (from ROADMAP.md §2120–§2132)

```text
🧠 LUNA owns: Technical design, mathematical derivation, interpretation,
  capability modeling, constraint synthesis, sequencing, acceptance decisions.

🛠 EXECUTOR owns: Local environment access, file operations, terminal execution,
  evidence acquisition, provenance capture, evidence persistence,
  measurements explicitly assigned.
```

This document is the Executor's evidence-acquisition deliverable. The ORCHESTRATOR will
independently review, design the formula, derive the equation, interpret the evidence,
and accept the technical design.

### 1.8 Do-Not-Run Compliance (This Task)

The Executor did NOT perform:
- T4.5 technical design
- T4.5 mathematical derivation
- T4.5 formula construction
- T4.5 technical acceptance
- T4.5 memory-model interpretation
- T4.6 / T4.7 / SET5 execution
- SET2 remediation
- Modification of T4.4 documents or historical SET2/SET3 records

---

## 2. Evidence Acquisition Protocol and Environment

### 2.1 Environment Inspection Performed

The assigned environment was inspected for the presence of a runtime, inference engine,
or live model instance capable of exposing linear-attention runtime state behavior.

```text
Repository code artifacts:
  - No runtime inference engine source code (.c, .cpp, .cc, .go, .js, .ts) found in repository.
  - No model loading, compilation, or execution code found on disk.
  - Repository contains only: ROADMAP.md, docs/, model/ (metadata only).

Python environment:
  - python3: /usr/bin/python3 (3.12.3)
  - torch: 2.11.0+cu130 (installed, not used for model loading — no model weights loaded)
  - transformers: 5.3.0 (installed, not used — no model instantiated)
  - safetensors: 0.7.0 (installed, used for metadata acquisition only)
  - jax: 0.10.2, jaxlib: 0.10.2 (installed, not used)
  - numpy: installed (not used)
  - huggingface-hub: NOT used (no model download performed)

Hardware (consistent with SET4-04 §2.1 environment):
  - CPU: Intel Core Ultra 7 155H (Meteor Lake), 16C/22T host, 4C/8T guest (WSL2 cgroup)
  - GPU: Intel Arc 7D55 — VERIFIED ABSENT from WSL2 guest (torch.cuda.is_available() = False)
  - NPU: Intel AI Boost 7D1D — VERIFIED ABSENT from WSL2 guest
  - RAM: 16 GB host, 12 GB WSL2 cgroup cap, dual-channel, no ECC

Runtime observation conclusion:
  - No model was loaded, instantiated, or executed during this evidence-acquisition task.
  - No linear-attention recurrent state buffer, convolution state buffer, or state
    allocator was observed running.
  - No inference engine, no allocator, no kernel implementation, no live process
    was observed for linear-attention state management.
```

**Classification:** VERIFIED FACT (environment state: no runtime present). UNKNOWN (all
runtime linear-attention state behaviors — no live execution occurred).

### 2.2 Runtime-Visible Configuration (from config.json only)

The only runtime-relevant configuration fields available without executing a runtime:

| Field | Value | Source | Classification |
|---|---|---|---|
| `text_config.use_cache` | `true` | config.json | VERIFIED FACT (config field) |
| `text_config.mamba_ssm_dtype` | `"float32"` | config.json | VERIFIED FACT (metadata config field only — NOT tensor dtype) |
| `text_config.dtype` | `"bfloat16"` | config.json | VERIFIED FACT (checkpoint tensor dtype) |
| `text_config.num_hidden_layers` | `64` | config.json | VERIFIED FACT |
| `text_config.layer_types` | 64-entry array | config.json | VERIFIED FACT |
| `text_config.linear_num_key_heads` | `16` | config.json | VERIFIED FACT |
| `text_config.linear_num_value_heads` | `48` | config.json | VERIFIED FACT |
| `text_config.linear_key_head_dim` | `128` | config.json | VERIFIED FACT |
| `text_config.linear_value_head_dim` | `128` | config.json | VERIFIED FACT |
| `text_config.linear_conv_kernel_dim` | `4` | config.json | VERIFIED FACT |
| `text_config.full_attention_interval` | `4` | config.json | VERIFIED FACT |

All other runtime behaviors (whether `use_cache = true` causes KV/linear-attention state
caching to occur, whether the cache is paged, whether recurrent state is pre-allocated
or grown per-token) are UNKNOWN — the flags are not exercised at runtime in this environment.

---

## 3. Linear-Attention Architecture

### 3.1 Linear-Attention Layer Count

| Field | Value | Source | Source Type | Classification | Provenance |
|---|---|---|---|---|---|
| Linear-attention layer count | 48 | config.json: `text_config.layer_types` (48 × `"linear_attention"`) | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §4 (§242: `48 linear-attention layers`); SET0-04 §2 (§59); SET0-07 §2 (§80), §4 (§223: `48`); SET3 §2.3 (§122: `48`); ROADMAP §2038 |
| Linear-attention layer indices | 0, 1, 2, 4, 5, 6, 8, 9, 10, 12, 13, 14, 16, 17, 18, 20, 21, 22, 24, 25, 26, 28, 29, 30, 32, 33, 34, 36, 37, 38, 40, 41, 42, 44, 45, 46, 48, 49, 50, 52, 53, 54, 56, 57, 58, 60, 61, 62 | config.json: `text_config.layer_types` array (explicit 64-entry sequence) | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-07 §5 (§203–219: explicit index list, count 48) |
| Full-attention layer count (for boundary) | 16 | config.json: `text_config.layer_types` (16 × `"full_attention"`) | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §4 (§243); SET0-04 §2 (§59); SET0-07 §2, §4; SET3 §2.3 (§120: count = 16); SET4-04 §3.1 (§164) |

### 3.2 Layer Pattern / Cadence

| Field | Value | Source | Source Type | Classification | Provenance |
|---|---|---|---|---|---|
| Layer pattern | `[LA → LA → LA → FA] × 16` | config.json: `layer_types` array + `full_attention_interval = 4` | Checkpoint config artifact | VERIFIED FACT (pattern); DERIVED FINDING (×16 shorthand) | SET0-03 §4 (§216: `[LA → LA → LA → FA] × 16`); SET0-07 §2 (§86), §7 (§268); SET0-04 §2 (§63–67) |
| Full-attention interval | 4 | config.json: `text_config.full_attention_interval = 4` | Checkpoint config artifact | VERIFIED FACT; CONSISTENCY VERIFIED | SET0-03 §4 (§216–§244: 64/4=16, interval == array spacing); SET0-04 §8 (§412: interval consistent with pattern); SET3 §2.3 (§124) |
| Linear-attention / full-attention ratio | 48 / 16 = 3 | Arithmetic on config fields | DERIVED FINDING | SET0-07 §12 (§441: `48 / 64 = 75%`); SET0-04 §2 (§59) |

### 3.3 State Architecture Distinction

| Field | Value | Source | Classification | Provenance |
|---|---|---|---|---|
| Full-attention state model | KV Cache | SET0-04 §3.2 (§208), §3.3, §15.1 (§619–629) | VERIFIED FACT | SET0-04 §15.1 (§641: "LINEAR ATTENTION STATE: RECURRENT STATE + CONVOLUTION STATE"); SET3-OC-3 §3.3 §6.2 |
| Linear-attention state model | Recurrent State + Convolution State | SET0-04 §9 (§405–429), §10 (§433–456), §15.1 (§619–629) | VERIFIED FACT (state model identity) | SET0-04 §15.1 (§619–629: `48 Gated DeltaNet layers → recurrent state + convolution state`); SET3-OC-4 §3.4 (§329–336: "Full Attention → KV Cache; Linear Attention → Recurrent State + Convolution State") |
| Exact linear-attention algorithm | UNKNOWN (Mamba / Mamba-2 / GatedDeltaNet / DeltaNet) | SET0-03 §7 (§369–393); SET0-04 §7 (§378–382); SET3-OC-4 §3.4 (§338–344) | UNKNOWN (UK-001) | SET0-03 §7 (§385–386: "UNKNOWN: exact internal linear-attention algorithm"); SET0-04 §7 (§378–382); SET3 §9 UK-001 (§932); SET4-01 §4 (§1094) |

**Classification note:** The state-model distinction (KV cache ≠ recurrent+conv state) is
VERIFIED FACT. The exact algorithm identity (which specific linear-attention algorithm is
used) is UNKNOWN (UK-001). The Executor does NOT resolve UK-001.

---

## 4. Linear-Attention Head Structure

### 4.1 Linear-Attention Head Configuration (from config.json)

All values read directly from `config.json` (`text_config` section).

| Field | Value | Source | Source Type | Classification | Provenance |
|---|---|---|---|---|---|
| `linear_num_key_heads` | 16 | config.json: `text_config.linear_num_key_heads = 16` | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §7 (§332: `linear_num_key_heads: 16`); SET0-04 §6.1 (§287: `linear_num_key_heads: 16`); SET0-04 §13 (§543: `16 K heads`); SET3-OC-4 §3.4 (§291) |
| `linear_num_value_heads` | 48 | config.json: `text_config.linear_num_value_heads = 48` | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §7 (§335: `linear_num_value_heads: 48`); SET0-04 §6.1 (§290: `linear_num_value_heads: 48`); SET0-04 §13 (§544: `48 V heads`); SET3-OC-4 §3.4 (§293) |
| `linear_key_head_dim` | 128 | config.json: `text_config.linear_key_head_dim = 128` | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §7 (§326: `linear_key_head_dim: 128`); SET0-04 §6.1 (§293: `linear_key_head_dim: 128`); SET0-04 §13 (§545: `128-d heads`); SET3-OC-4 §3.4 (§294) |
| `linear_value_head_dim` | 128 | config.json: `text_config.linear_value_head_dim = 128` | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §7 (§329: `linear_value_head_dim: 128`); SET0-04 §6.1 (§296: `linear_value_head_dim: 128`); SET0-04 §13 (§545: `128-d heads`); SET3-OC-4 §3.4 (§295) |
| `linear_conv_kernel_dim` | 4 | config.json: `text_config.linear_conv_kernel_dim = 4` | Checkpoint config artifact (raw) | VERIFIED FACT | SET0-03 §7 (§339: `linear_conv_kernel_dim: 4`); SET0-04 §6.1 (§299: `linear_conv_kernel_dim: 4`); SET0-04 §8 (§391: `linear_conv_kernel_dim: 4`); SET0-04 §13 (§546: `kernel dimension = 4`); SET3-OC-4 §3.4 (§296); SET3-OC-10 §3.10 (§454) |

### 4.2 Derived Head Relationships

| Field | Value | Source | Source Type | Classification | Provenance |
|---|---|---|---|---|---|
| Key representation dimension | 16 × 128 = 2048 | Arithmetic on `linear_num_key_heads × linear_key_head_dim` | DERIVED FINDING | 2048 = `16 × 128` | SET0-04 §6.2 (§313: `16 × 128 = 2048`); SET0-07 §12 (§454: `48 / 16 = 3`) |
| Value representation dimension | 48 × 128 = 6144 | Arithmetic on `linear_num_value_heads × linear_value_head_dim` | DERIVED FINDING | 6144 = `48 × 128` | SET0-04 §6.2 (§319: `48 × 128 = 6144`); SET0-07 §12 (§454: `48 / 16 = 3`); SET3-OC-4 §3.4 (§319: `in_proj_z output: 6144 = 48 × 128`) |
| KV head expansion ratio | 48 / 16 = 3 | Arithmetic on config fields | DERIVED FINDING | 3 | SET0-04 §6.2 (§336: `48 / 16 = 3`); SET3-OC-4 §3.4 (§317: `10240 = 16×128 + 16×128 + 48×128`) |
| QKV projection output dimension | 10240 | `(16×128) + (16×128) + (48×128) = 2048 + 2048 + 6144 = 10240` | DERIVED FINDING (arithmetic; matches checkpoint tensor `in_proj_qkv.weight [10240, 5120]`) | VERIFIED FACT (checkpoint tensor shape matches) | SET3-OC-4 §3.4 (§317–318: `10240 = 16×128 + 16×128 + 48×128 = 2048 + 2048 + 6144 = 10240`); SET0-08 §2 (§65: `linear_attn.in_proj_qkv.weight [10240, 5120] 48/48`); SET0-03 §7 (§347–358: `16 × 128 = 2048`, `48 × 128 = 6144`) |
| Z projection output dimension | 6144 | `(48 × 128) = 6144` | VERIFIED FACT (matches checkpoint tensor `in_proj_z.weight [6144, 5120]`) | VERIFIED FACT | SET3-OC-4 §3.4 (§319: `in_proj_z output: 6144 = 48 × 128`); SET0-08 §2 (§66: `linear_attn.in_proj_z.weight [6144, 5120] 48/48`); SET4-02 §3.4 (§279) |
| in_proj_b output dimension | 48 | `linear_num_value_heads = 48` | VERIFIED FACT (matches checkpoint tensor `in_proj_b.weight [48, 5120]`) | VERIFIED FACT | SET3-OC-4 §3.4 (§321: `in_proj_b: 48 = linear_num_value_heads`); SET0-08 §2 (§67: `linear_attn.in_proj_b.weight [48, 5120] 48/48`); SET4-02 §3.4 (§282) |
| in_proj_a output dimension | 48 | `linear_num_value_heads = 48` | VERIFIED FACT (matches checkpoint tensor `in_proj_a.weight [48, 5120]`) | VERIFIED FACT | SET3-OC-4 §3.4 (§322: `in_proj_a: 48 = linear_num_value_heads`); SET0-08 §2 (§68: `linear_attn.in_proj_a.weight [48, 5120] 48/48`); SET4-02 §3.4 (§283) |
| out_proj input dimension | 6144 | `48 × 128 = 6144` | VERIFIED FACT (matches checkpoint tensor `out_proj.weight [5120, 6144]`) | VERIFIED FACT | SET3-OC-4 §3.4 (§323: `out_proj: 6144 → 5120, i.e., 48×128 → 5120`); SET0-08 §2 (§69: `linear_attn.out_proj.weight [5120, 6144] 48/48`); SET4-02 §3.4 (§285) |
| conv1d kernel shape | [10240, 1, 4] | Raw checkpoint tensor metadata | VERIFIED FACT | VERIFIED FACT | SET0-08 §2 (§70: `linear_attn.conv1d.weight [10240, 1, 4] 48/48`); SET3-OC-10 §3.10 (§455); SET3-OC-4 §3.4 (§324: `conv1d: [10240, 1, 4] — kernel size 4 along sequence dimension`); SET4-02 §3.4 (§287) |
| A_log parameter shape | [48] | Raw checkpoint tensor metadata | VERIFIED FACT | VERIFIED FACT | SET0-08 §2 (§71: `linear_attn.A_log [48] 48/48`); SET3-OC-4 §3.4 (§325: `A_log: [48] — state dimension parameter`); SET4-02 §3.4 (§289) |
| dt_bias parameter shape | [48] | Raw checkpoint tensor metadata | VERIFIED FACT | VERIFIED FACT | SET0-08 §2 (§72: `linear_attn.dt_bias [48] 48/48`); SET3-OC-4 §3.4 (§326: `dt_bias: [48] — timestep bias parameter`); SET4-02 §3.4 (§290) |
| norm parameter shape | [128] | Raw checkpoint tensor metadata | VERIFIED FACT | VERIFIED FACT | SET0-08 §2 (§73: `linear_attn.norm.weight [128] 48/48`); SET3-OC-4 §3.4 (§327: `norm: [128] — per-head normalization`) |

### 4.3 Q/K/V Head Organization

| Property | Value | Source | Classification | Provenance |
|---|---|---|---|---|
| Q heads (linear-attention) | 16 (linear_num_key_heads) | config.json | VERIFIED FACT | SET0-04 §13 (§543: `16 K heads`); SET3-OC-4 §3.4 (§291) |
| K heads (linear-attention) | 16 (linear_num_key_heads) | config.json | VERIFIED FACT | SET0-04 §6.1 (§287: `linear_num_key_heads: 16`); SET3-OC-4 §3.4 (§291) |
| V heads (linear-attention) | 48 (linear_num_value_heads) | config.json | VERIFIED FACT | SET0-04 §6.1 (§290: `linear_num_value_heads: 48`); SET3-OC-4 §3.4 (§293) |
| Head dimension (linear-attention) | 128 | config.json `linear_key_head_dim` = `linear_value_head_dim` = 128 | VERIFIED FACT | SET0-04 §6.1 (§293–298); SET0-04 §13 (§545: `128-d heads`); SET3-OC-4 §3.4 (§294–295) |
| K/Q head expansion | 48 / 16 = 3 (V heads / K heads) | Arithmetic | DERIVED FINDING | SET0-04 §6.2 (§336: `48 / 16 = 3`); SET0-03 §7 (§369–370, §397) |

**Important boundary:** The implementation expands the key/query head count to match
the value head count (48). The ratio is `48 / 16 = 3`. SET0-04 §6.2 states:
"The implementation expands the key/query head count to match the value head count.
The ratio is: 48 / 16 = 3. Thus the 16 key/query heads are repeated across 48 processing heads."
This is a DERIVED FINDING (head expansion ratio). It must NOT be confused with the GQA
structure of full attention (which uses 24/4 = 6).

---

## 5. Linear-Attention Projection Evidence

### 5.1 Verified Linear-Attention Projection Tensors

Per-layer tensor set (48/48 coverage VERIFIED):

| Tensor Name | Shape | Coverage | Source | Classification | Provenance |
|---|---|---|---|---|---|
| `linear_attn.in_proj_qkv.weight` | `[10240, 5120]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§65); SET3-OC-4 §3.4 (§303); T4.1 RM-003 (§175), T4.2 §3.4 (§244) |
| `linear_attn.in_proj_z.weight` | `[6144, 5120]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§66); SET3-OC-4 §3.4 (§304); T4.1 RM-003 (§175), T4.2 §3.4 (§245) |
| `linear_attn.in_proj_b.weight` | `[48, 5120]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§67); SET3-OC-4 §3.4 (§305); T4.1 RM-003 (§175), T4.2 §3.4 (§246) |
| `linear_attn.in_proj_a.weight` | `[48, 5120]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§68); SET3-OC-4 §3.4 (§306); T4.1 RM-003 (§175), T4.2 §3.4 (§247) |
| `linear_attn.out_proj.weight` | `[5120, 6144]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§69); SET3-OC-4 §3.4 (§307); T4.1 RM-003 (§175), T4.2 §3.4 (§248) |
| `linear_attn.conv1d.weight` | `[10240, 1, 4]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§70); SET3-OC-4 §3.4 (§308); T4.2 §3.4 (§287); SET3-OC-10 §3.10 (§455) |
| `linear_attn.A_log` | `[48]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§71); SET3-OC-4 §3.4 (§309); T4.2 §3.4 (§289) |
| `linear_attn.dt_bias` | `[48]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§72); SET3-OC-4 §3.4 (§310); T4.2 §3.4 (§290) |
| `linear_attn.norm.weight` | `[128]` | 48/48 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§73); SET3-OC-4 §3.4 (§311) |

**Tensor count:** 9 tensors × 48 layers = 432 tensors total (VERIFIED FACT).
Source: SET3 §5.2 (§583: "Linear-attention layers: 432 tensors (9 per layer × 48)"),
SET0-08 §2, T4.1 RM-003 (§177: "9 tensors × 48 layers = 432 tensors (48/48 coverage").

### 5.2 Projection Output Shapes (Derived from Checkpoint Tensor Shapes)

| Projection | Output Dimension (from weight matrix rows) | Shape (parameterized by B, S) | Source | Classification |
|---|---|---|---|---|
| QKV projection (in_proj_qkv) | 10240 = 16×128 + 16×128 + 48×128 | `[B, S, 10240]` | `in_proj_qkv.weight [10240, 5120]` | VERIFIED FACT (shape from raw tensor) |
| Z projection (in_proj_z) | 6144 = 48×128 | `[B, S, 6144]` | `in_proj_z.weight [6144, 5120]` | VERIFIED FACT (shape from raw tensor) |
| B parameter (in_proj_b) | 48 = linear_num_value_heads | `[48]` | `in_proj_b.weight [48, 5120]` | VERIFIED FACT |
| A parameter (in_proj_a) | 48 = linear_num_value_heads | `[48]` | `in_proj_a.weight [48, 5120]` | VERIFIED FACT |
| Out projection (out_proj) | 6144 → 5120 | `6144 → [B, S, 5120]` | `out_proj.weight [5120, 6144]` | VERIFIED FACT |

### 5.3 State-Relevant Parameters

The following checkpoint parameters parameterize the linear-attention recurrence but
are NOT the runtime state buffers themselves:

| Tensor | Shape | Role | Source | Classification | Provenance |
|---|---|---|---|---|---|
| `linear_attn.A_log` | `[48]` | Parameterizes the recurrence matrix (A in SSM/DeltaNet) | Raw checkpoint metadata | VERIFIED FACT (parameter exists) | SET0-08 §2 (§71); SET3-OC-4 §3.4 (§325: "state dimension parameter"); SET4-01 RM-003 (§179: "Stateful: No for weights; YES for A_log and dt_bias as runtime recurrence parameters") |
| `linear_attn.dt_bias` | `[48]` | Timestep bias for the recurrence | Raw checkpoint metadata | VERIFIED FACT (parameter exists) | SET0-08 §2 (§72); SET3-OC-4 §3.4 (§326: "timestep bias parameter"); SET4-01 RM-003 (§179) |

**Critical distinction (VERIFIED FACT):** `A_log [48]` and `dt_bias [48]` are checkpoint
parameter tensors. Whether they define the runtime recurrent state buffer shape is
CONDITIONAL — the runtime state buffer itself is UNKNOWN (UK-012, UK-001).
SET4-01 RM-019 states: "A_log and dt_bias are checkpoint parameter tensors, but their
exact runtime role in the linear-attention state update is UNKNOWN (UK-001)."

---

## 6. Linear-Attention Dataflow / Operator Stages

### 6.1 Linear-Attention Execution Stages (From SET3-OC-4 §3.4 §6.3)

The linear-attention operator (Qwen3_5GatedDeltaNet, OC-4) is documented as executing
through these stages:

```text
Hidden States
    ↓
RMSNorm (input_layernorm)
    ↓
Q / K / V-related Projections:  in_proj_qkv [10240], in_proj_z [6144]
    ↓
Causal Convolution: conv1d [10240, 1, 4], kernel_dim=4
    ↓
B/A Parameters: in_proj_b [48], in_proj_a [48]
    ↓
Gated Delta Rule (recurrent)
    ↓
Recurrent State Update (A_log [48], dt_bias [48])
    ↓
Output Gating / Normalization: norm.weight [128]
    ↓
Output Projection: out_proj [5120, 6144]
    ↓
[batch, seq, 5120]
```

**Source:** SET3-OC-4 §3.4 §6.3 (§700–734). Classification: DERIVED FINDING (stage
ordering is described by the official implementation, not by checkpoint metadata alone).

### 6.2 State-Relevant Stage Boundaries

| Stage | State Type | Checkpoint Tensor Evidence | Classification |
|---|---|---|---|
| Causal Convolution (conv1d) | Convolution State | `conv1d.weight [10240, 1, 4]` — kernel params exist | VERIFIED FACT (convolution stage exists, kernel_dim=4) | VERIFIED; runtime conv state buffer = UNKNOWN (UK-012) |
| B/A Parameters | Gating/Delta | `in_proj_b [48, 5120]`, `in_proj_a [48, 5120]` | VERIFIED FACT | VERIFIED FACT |
| Recurrent State Update | Recurrent State | `A_log [48]`, `dt_bias [48]` — state params exist | VERIFIED FACT (parameters exist); runtime state buffer = UNKNOWN (UK-012) | SET3-OC-4 §3.4 (§329–336) |
| Output Gating/Normalization | Activation | `norm.weight [128]` | VERIFIED FACT | VERIFIED FACT |

**State model distinction (VERIFIED FACT):**
```text
Full Attention  →  KV Cache
Linear Attention →  Recurrent State + Convolution State
```
Source: SET3-OC-4 §3.4 (§329–336); SET0-04 §9 (§405–429), §15.1 (§619–629).

---

## 7. State-Related Configuration Evidence

### 7.1 State-Related Config Fields (from config.json)

| Field | Value | Path | Classification | Provenance |
|---|---|---|---|---|
| `linear_num_key_heads` | 16 | `text_config.linear_num_key_heads` | VERIFIED FACT | SET0-03 §7, SET0-04 §6.1, SET3-OC-4 §3.4 |
| `linear_num_value_heads` | 48 | `text_config.linear_num_value_heads` | VERIFIED FACT | SET0-03 §7, SET0-04 §6.1, SET3-OC-4 §3.4 |
| `linear_key_head_dim` | 128 | `text_config.linear_key_head_dim` | VERIFIED FACT | SET0-03 §7, SET0-04 §6.1, SET3-OC-4 §3.4 |
| `linear_value_head_dim` | 128 | `text_config.linear_value_head_dim` | VERIFIED FACT | SET0-03 §7, SET0-04 §6.1, SET3-OC-4 §3.4 |
| `linear_conv_kernel_dim` | 4 | `text_config.linear_conv_kernel_dim` | VERIFIED FACT | SET0-03 §7, SET0-04 §6.1/§8, SET3-OC-4 §3.4, SET3-OC-10 §3.10 |
| `mamba_ssm_dtype` | `"float32"` | `text_config.mamba_ssm_dtype` | VERIFIED FACT (metadata field only) | SET0-03 §7 (§388); SET3 §4.3 (§519–524: "metadata field, not tensor dtype"); SET3 §8 ASSUMPTION #4 (§921–924) |
| `mamba_ssm_dtype` implies runtime state in float32 | UNKNOWN | — | UNKNOWN (UK-002) | SET3 §9 UK-002 (§933); SET4-01 §4 (§1095); SET4-03 §2 (§110) |
| `use_cache` | `true` | `text_config.use_cache` | VERIFIED FACT (config field) | config.json line 116; SET4-01 RM-012 (§384), RM-013 (§403) |
| `max_position_embeddings` | 262144 | `text_config.max_position_embeddings` | VERIFIED FACT | SET0-03 §4 (§154); SET3 §2.2 (§97) |
| `dtype` | `"bfloat16"` | `text_config.dtype` | VERIFIED FACT (checkpoint tensor dtype) | config.json line 13; SET0-03 §4 (§149); SET3 §2.2 (§96) |

### 7.2 State-Related Dtype Metadata

| Property | Value | Source | Classification |
|---|---|---|---|
| Checkpoint tensor dtype (all 1,199 tensors) | BF16 | config.json `text_config.dtype` + SET1 raw tensor metadata | VERIFIED FACT |
| Bytes per BF16 element | 2 | Derived from BF16 | DERIVED FINDING |
| `mamba_ssm_dtype` | `"float32"` | config.json `text_config.mamba_ssm_dtype` | VERIFIED FACT (metadata config field, NOT tensor storage dtype) |
| Whether `mamba_ssm_dtype=float32` implies runtime state in float32 | UNKNOWN | — | UNKNOWN (UK-002) |
| Runtime computation dtype | UNKNOWN | No runtime observation | UNKNOWN (UK-002, UK-004) |
| Runtime linear-attention state dtype | UNKNOWN | No runtime observation | UNKNOWN (UK-002) |
| Whether checkpoint BF16 implies runtime state in BF16 | UNKNOWN | No runtime observation | UNKNOWN (UK-002, UK-004) |

### 7.3 State-Related Configuration Flags

| Flag / Field | Value | Classification |
|---|---|---|
| `layer_types` (per-layer dispatch) | 64-entry array: 48× `"linear_attention"`, 16× `"full_attention"` | VERIFIED FACT |
| `full_attention_interval` | 4 | VERIFIED FACT (consistency check vs layer_types) |
| `use_cache` | true | VERIFIED FACT (config field only; runtime behavior = UNKNOWN) |

---

## 8. Linear-Attention Tensor Inventory (Per-Layer)

### 8.1 Complete Per-Layer Tensor Set (48/48 Coverage VERIFIED)

All 9 linear-attention tensors per layer are present with 48/48 coverage:

| # | Tensor Name | Shape | Parameters (at shape) | Logical Bytes | Operator |
|---|---|---|---|---|---|
| 1 | `linear_attn.in_proj_qkv.weight` | `[10240, 5120]` | 52,428,800 | 104,857,600 | OC-4 |
| 2 | `linear_attn.in_proj_z.weight` | `[6144, 5120]` | 31,457,280 | 62,914,560 | OC-4 |
| 3 | `linear_attn.in_proj_b.weight` | `[48, 5120]` | 245,760 | 491,520 | OC-4 |
| 4 | `linear_attn.in_proj_a.weight` | `[48, 5120]` | 245,760 | 491,520 | OC-4 |
| 5 | `linear_attn.out_proj.weight` | `[5120, 6144]` | 31,457,280 | 62,914,560 | OC-4 |
| 6 | `linear_attn.conv1d.weight` | `[10240, 1, 4]` | 40,960 | 81,920 | OC-10 |
| 7 | `linear_attn.A_log` | `[48]` | 48 | 96 | OC-4 (state param) |
| 8 | `linear_attn.dt_bias` | `[48]` | 48 | 96 | OC-4 (state param) |
| 9 | `linear_attn.norm.weight` | `[128]` | 128 | 256 | OC-4 |
| | **Per-layer subtotal** | | **186,676,768** | **373,353,536** | |
| | **Aggregate (48 layers)** | | **8,960,484,864** | **17,920,969,728** | |

Note: `A_log` and `dt_bias` are checkpoint parameter tensors (VERIFIED FACT), NOT
runtime state buffers. Their role as recurrence state parameters is documented but
the runtime state buffer shape is UNKNOWN (UK-012). These are distinct from the
runtime recurrent state and convolution state (RM-019, RM-020 in SET4-01).

Per-parameter arithmetic: DERIVED FINDING from raw tensor shapes × BF16 (2 bytes/element).
Sources: SET0-08 §2 (§65–73); SET3-OC-4 §3.4 (§303–311); SET4-02 §3.4 (§244–290);
SET3 §5.2 (§583: 432 tensors for linear-attention).

### 8.2 State-Parameter vs State-Buffer Distinction

The checkpoint contains **parameter** tensors for the linear-attention recurrence:
- `A_log [48]` — recurrence matrix log-parameter
- `dt_bias [48]` — timestep bias
- `in_proj_b [48]` — gating parameter (B)
- `in_proj_a [48]` — delta rule parameter (A)
- `conv1d.weight [10240, 1, 4]` — causal convolution kernel

These are all VERIFIED FACT (checkpoint tensors exist). However:

| Runtime state object | Checkpoint parameter presence | Runtime state buffer | Classification |
|---|---|---|---|
| Recurrent state buffer | `A_log [48]`, `dt_bias [48]` parameterize the recurrence | UNKNOWN (no runtime observation) | CHECKPOINT PARAM = VERIFIED FACT; RUNTIME STATE BUFFER = UNKNOWN (UK-012) |
| Convolution state buffer | `conv1d.weight [10240, 1, 4]` defines kernel | UNKNOWN (no runtime observation) | CHECKPOINT PARAM = VERIFIED FACT; RUNTIME STATE BUFFER = UNKNOWN (UK-012) |

Source: SET4-01 RM-019 (§528: "checkpoint contains A_log [48] and dt_bias [48] as state
parameters, but the runtime recurrent state buffer shape, dtype, and allocation strategy
are UNKNOWN"); SET4-01 RM-020 (§547: "The runtime state buffer that the convolution
populates is UNKNOWN").

---

## 9. State-Related Dataflow Evidence

### 9.1 Linear-Attention Dataflow (SET3-OC-4 §3.4 §6.3)

```text
hidden_states [batch, seq, 5120]
    ↓
RMSNorm (input_layernorm)
    ↓
Q/K/V Projections: in_proj_qkv [10240, 5120], in_proj_z [6144, 5120]
    ↓
Causal Convolution: conv1d [10240, 1, 4], kernel_dim=4
    ↓
B/A Parameters: in_proj_b [48], in_proj_a [48]
    ↓
Gated Delta Rule (recurrent)
    ↓
Recurrent State Update (A_log [48], dt_bias [48])
    ↓
Output Gating / Normalization: norm.weight [128]
    ↓
Output Projection: out_proj [5120, 6144]
    ↓
[batch, seq, 5120]
```

Classification: Tensor shapes = VERIFIED FACT. Stage ordering = DERIVED FINDING
(described by official implementation). Exact algorithm (Mamba/Mamba-2/GatedDeltaNet) =
UNKNOWN (UK-001).

### 9.2 State Input/Output Boundaries (Activation Side)

From T4.3 §11 (AC-35, AC-36) and §19.3:

| AC ID | Boundary Object | Activations Consumed | State Produced | Classification |
|---|---|---|---|---|
| AC-35 | `gated_delta_intermediates[L]` | Q/K/V projections, B/A params | Feeds recurrent state update | CONDITIONAL MODEL (existence implied; exact shape = UNKNOWN, UK-001) |
| AC-36 | `la_conv_state_boundary[L]` | QKV projection output `[B, S, 10240]` (AC-14) | Feeds convolution state | ACTIVATION BOUNDARY (state: T4.5 UNKNOWN, UK-012) |

**AC-35 detail (T4.3 §11.1):**
- Identity: `gated_delta_intermediates[L]` — intermediate tensors for gated-delta-rule computation
- Shape: UNKNOWN — depends on the exact linear-attention algorithm (UK-001)
- Dtype: UNKNOWN (UK-002)
- Classification: CONDITIONAL MODEL (existence implied by SET3 dataflow; exact shape = UNKNOWN, depends on UK-001)
- Provenance: SET3-OC-4 §3.4 §6.3 (Gated Delta Rule stage); SET0-04 §7; T4.1 §3.10.3 RM-044

**AC-36 detail (T4.3 §11.2):**
- Identity: `la_conv_state_boundary[L]` — activation-side observation that QKV projection output feeds a downstream convolution state transition
- Activation-side boundary shape: QKV projection output `[B, S, 10240]` (AC-14) — VERIFIED FACT
- Runtime state buffer shape: UNKNOWN (UK-012, T4.5 domain)
- Dtype: Activation-side BF16 (conditional); convolution state dtype = UNKNOWN (UK-002)
- Classification: ACTIVATION BOUNDARY (the convolution state buffer itself is STATEFUL and belongs to T4.5)
- Provenance: SET3-OC-4 §3.4 (state model); SET3-OC-10 §3.10 (CausalConv1D); SET0-04 §8; T4.1 §3.4 RM-020

**T4.3 §19.3 T4.5 Consumption:**
- AC-35 (`gated_delta_intermediates`): Activation boundary — exact shapes UNKNOWN (UK-001, T4.5 domain)
- AC-36 (`la_conv_state_boundary`): Activation boundary — exact state shape UNKNOWN (UK-012, T4.5 domain)

---

## 10. Runtime Evidence (Linear-Attention State)

### 10.1 Environment Inspection Results

| Observation | Result | Source / Method |
|---|---|---|
| Was a model loaded into a runtime (torch/transformers/JAX)? | NO — no model was instantiated or loaded | Repository inspection: no inference engine code in repository; Python packages installed but no model weights loaded |
| Was a linear-attention recurrent state buffer observed? | NO — no state buffer was running | No runtime process observed; no state allocator code in repository |
| Was a linear-attention convolution state buffer observed? | NO — no conv state buffer was running | No runtime process observed |
| Was a live inference engine running? | NO | `ps` process inspection; repository file scan |
| Was batch size (B) observed at runtime? | UNKNOWN | No runtime execution occurred |
| Was sequence length (S) observed at runtime? | UNKNOWN | No runtime execution occurred |
| Was linear-attention state memory allocation observed? | UNKNOWN | No runtime execution occurred; no allocator code present |
| Was recurrent state growth strategy observed? | UNKNOWN | No runtime execution occurred |
| Was state persistence strategy observed? | UNKNOWN | No runtime execution occurred |
| Was state dtype at runtime observed? | UNKNOWN | No runtime execution occurred |
| Was state reuse (circular/paged) observed? | UNKNOWN | No runtime execution occurred |
| Was state quantization observed? | UNKNOWN | No runtime execution occurred |
| Was state layout (head-major vs seq-major) observed? | UNKNOWN | No runtime execution occurred |
| Was batch dependency observed? | UNKNOWN | No runtime execution occurred |
| Was sequence dependency observed? | UNKNOWN | No runtime execution occurred |
| Was a state allocator observed? | NO | No runtime process; no allocator code in repository |
| Was a loaded model observed? | NO | No model weights loaded in any runtime |
| Was an inference engine observed? | NO | No inference engine code in repository; no process running |
| Was a kernel implementation observed? | NO | No kernel source code in repository; no runtime process |
| Was an allocator observed? | NO | No allocator code in repository; no runtime process |
| Was GPU/accelerator observed? | VERIFIED ABSENT from WSL2 guest | `torch.cuda.is_available() = False`; SET2-T2.8 §8.1 |

**Classification:** VERIFIED FACT (environment state: no runtime present). UNKNOWN (all
runtime linear-attention state behaviors — no live execution occurred).

### 10.2 Runtime-Visible Configuration (from config.json only)

The only runtime-relevant configuration fields available without executing a runtime:

| Field | Value | Source | Classification |
|---|---|---|---|
| `text_config.use_cache` | `true` | config.json | VERIFIED FACT (config field only; runtime behavior = UNKNOWN) |

All other runtime behaviors (whether `use_cache = true` causes actual linear-attention
state caching to occur, whether the recurrent state is pre-allocated or grown per-token,
whether the convolution state is circular/paged, etc.) are UNKNOWN — the flag is not
exercised at runtime in this environment.

---

## 11. Runtime UNKNOWN Register (Linear-Attention State)

For every runtime property, classified as UNKNOWN when actual evidence does not establish it.

### 11.1 Linear-Attention State UNKNOWN Register

| # | Runtime Property | Status | Reason |
|---|---|---|---|
| 1 | Runtime recurrent state dtype | UNKNOWN (UK-002) | No runtime engine executed; config only declares parameter dtype (BF16) and `mamba_ssm_dtype = float32` (metadata field, not tensor dtype) |
| 2 | Runtime convolution state dtype | UNKNOWN (UK-002) | No runtime engine executed; no evidence source documents conv state dtype |
| 3 | Runtime recurrent state allocation strategy (circular / paged / contiguous) | UNKNOWN (UK-012) | No runtime engine executed; no state allocator observed |
| 4 | Runtime convolution state allocation strategy | UNKNOWN (UK-012) | No runtime engine executed; no state allocator observed |
| 5 | Whether recurrent state is pre-allocated to max_position_embeddings (262144) or grows per-token | UNKNOWN (UK-012) | No runtime engine executed; implementation detail, no evidence source |
| 6 | Whether convolution state is pre-allocated (kernel_size=4) or grows per-token | UNKNOWN (UK-012) | No runtime engine executed; implementation detail, no evidence source |
| 7 | Recurrent state element layout (head-major vs seq-major vs blocked) | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| 8 | Whether recurrent state is contiguous in memory or strided | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| 9 | Whether recurrent state uses separate allocation per layer or a combined buffer | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| 10 | Whether convolution state uses separate allocation per layer or a combined buffer | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| 11 | Whether recurrent state is quantized (e.g., FP8 state) | UNKNOWN | Config dtype = BF16; `mamba_ssm_dtype = float32` is metadata only; no quantization config field observed for state |
| 12 | Whether convolution state is quantized | UNKNOWN | No evidence source documents state quantization; config does not specify |
| 13 | Whether recurrent state and convolution state share an allocator | UNKNOWN | No runtime observation; different state types per SET0-04 §15.1 |
| 14 | Runtime batch size (B) | UNKNOWN (UK-009) | No runtime execution occurred |
| 15 | Runtime sequence length (S) | UNKNOWN (UK-009) | No runtime execution occurred |
| 16 | Runtime batch × layer count for recurrent state | UNKNOWN | Both B and potentially dynamic layer count are runtime-dependent |
| 17 | Runtime recurrent state buffer shape | UNKNOWN (UK-001, UK-012) | Depends on exact linear-attention algorithm (UK-001) and runtime allocation (UK-012) |
| 18 | Runtime convolution state buffer shape | UNKNOWN (UK-001, UK-012) | Depends on exact linear-attention algorithm (UK-001) and runtime allocation (UK-012) |
| 19 | Runtime recurrent state initialization | UNKNOWN | No runtime engine executed; no evidence source establishes initialization |
| 20 | Runtime convolution state initialization | UNKNOWN | No runtime engine executed; no evidence source establishes initialization |
| 21 | Runtime recurrent state growth (per-token update pattern) | UNKNOWN | No runtime engine executed; no evidence source |
| 22 | Runtime recurrent state persistence (across what time horizon) | UNKNOWN | No runtime engine executed; no evidence source |
| 23 | Whether `A_log [48]` / `dt_bias [48]` directly define runtime state buffer dimensions | UNKNOWN (UK-001) | Checkpoint params exist (VERIFIED); runtime state buffer shape is UNKNOWN; whether 48 is the state dimension or maps to `[48 × 128 = 6144]` is algorithm-dependent (UK-001) |
| 24 | Whether `mamba_ssm_dtype = float32` implies runtime recurrent state in float32 | UNKNOWN (UK-002) | Metadata field only; no runtime observation |
| 25 | Whether `use_cache = true` triggers actual linear-attention state caching at inference time | UNKNOWN | Flag is config only; not exercised at runtime; exact runtime effect on linear-attention state = UNKNOWN |
| 26 | Whether recurrent state is evicted/paged to disk under memory pressure | UNKNOWN | No runtime engine executed; no streaming/paging policy observed |
| 27 | Whether recurrent state is shared across beam-search candidates | UNKNOWN | No runtime observation; no beam config field inspected for cache semantics |
| 28 | Whether tensor parallelism (TP) / device distribution affects linear-attention state | UNKNOWN | No runtime engine executed; config does not declare TP degree |
| 29 | Whether in-place operations reduce recurrent state footprint | UNKNOWN | No runtime kernel implementation observed |

### 11.2 SET3 UNKNOWNs Carried Forward (Relevant to T4.5)

| UK ID | Description | Evidence Domain | T4.5 Relevance |
|---|---|---|---|
| UK-001 | Exact linear-attention algorithm (Mamba/Mamba-2/GatedDeltaNet/DeltaNet) | Config declares presence, not identity | Directly relevant to state buffer shape, allocation, and dtype |
| UK-002 | Whether `mamba_ssm_dtype=float32` implies any runtime state in float32 | Metadata field only | Directly relevant to runtime recurrent/conv state dtype |
| UK-012 | Runtime linear-attention state allocation | Depends on runtime, not checkpoint | Directly relevant to state buffer allocation, layout, reuse |
| UK-004 | Exact MLP formulation beyond canonical gated-SiLU structure | Not established | Indirect (affects intermediate dtype during LA computation) |
| UK-005 | Exact normalization placement | Tensor names suggest placement but not verified | Indirect (affects coexistence of activations with state) |
| UK-006 | Exact residual connection structure | Not established by raw metadata | Indirect (affects activation coexistence) |
| UK-008 | Exact MTP computation path and integration | Checkpoint present, runtime UNKNOWN | Indirect (if MTP is active, it may interact with LA state) |
| UK-009 | Runtime batch/sequence tensor memory layout | No runtime execution | Directly relevant (B, S parameterize state element counts) |
| UK-011 | Runtime KV cache allocation strategy | Depends on runtime, not checkpoint | Indirect (full-attention state domain — kept separate) |
| UK-013 | Runtime streaming/paging strategy | Not established | Indirect (affects weight residency, not state allocation directly) |

---

## 12. MTP / Special-Path Boundary

### 12.1 MTP — Not Part of Linear-Attention State

The model declares `mtp_num_hidden_layers = 1` (VERIFIED FACT — config.json). The
checkpoint contains 15 MTP tensors including `mtp.layers.0.self_attn.*` tensors which
replicate the full-attention projection structure (`q_proj [12288, 5120]`, `k_proj
[1024, 5120]`, `v_proj [1024, 5120]`, `o_proj [5120, 6144]`, `q_norm [256]`,
`k_norm [256]`).

However:

- MTP active runtime execution is **UNKNOWN** (UK-008). Source: SET0-06 §6 (§200–201:
  "MTP active runtime execution = UNKNOWN"); SET0-08 §4 (§133: "MTP active runtime
  execution = UNKNOWN"); SET0-06 §6 (§220: do not infer checkpoint presence = runtime
  execution).
- Whether the MTP self-attention layer maintains KV cache state at runtime is **UNKNOWN**.
- The T4.4 evidence (SET4-04 §6.4) explicitly excludes MTP from the full-attention state
  model: "The base T4.4 model intentionally excludes the MTP self-attention contribution
  because MTP runtime execution remains UNKNOWN in the accepted evidence."

**Classification:** The linear-attention state model (T4.5) concerns only the 48
`linear_attention` layers. MTP is a separate conditional path (UK-008) and is NOT
included in the linear-attention state domain. Whether MTP maintains any state at
runtime is UNKNOWN.

### 12.2 Two Distinct State Models (Boundary Preserved)

This task concerns the linear-attention / recurrent / conv state domain ONLY. The
Executor does NOT merge:

```text
linear-attention state (recurrent state + convolution state)
≠
full-attention KV cache
≠
activation memory
≠
weight residency
```

The two state models are structurally distinct and VERIFIED FACT:

| State Model | Layer Type | Layer Count | State Mechanism |
|---|---|---|---|
| Full-attention state | `full_attention` → OC-3 (Qwen3_5Attention) | 16 | KV Cache |
| Linear-attention state | `linear_attention` → OC-4 (Qwen3_5GatedDeltaNet) | 48 | Recurrent State + Convolution State |

Source: SET0-04 §9 (§405–429), §15.1 (§619–629); SET3-OC-4 §3.4 (§329–336).

### 12.3 MTP Self-Attention State (Separate, UNKNOWN)

If MTP is active at runtime, its `self_attn` layer would use KV cache (matching full-attention
mechanism). This is a separate state domain from linear-attention state.

| Property | Status |
|---|---|
| MTP checkpoint tensors (self_attn.*) | VERIFIED FACT (exist in checkpoint, SET0-08 §3) |
| MTP runtime execution | UNKNOWN (UK-008) |
| MTP self-attention KV cache at runtime | UNKNOWN (UK-008) |
| MTP state as part of linear-attention state | NOT APPLICABLE — separate domain |

---

## 13. Structural Linear-Attention Evidence Summary

### 13.1 VERIFIED FACT (Linear-Attention Structure)

| # | Fact | Classification | Provenance |
|---|---|---|---|
| 1 | 48 linear-attention layers | VERIFIED FACT | config.json layer_types; SET0-03 §4, SET0-04 §2, SET0-07 §2, SET0-07 §4, SET3 §2.3 |
| 2 | Linear-attention layer indices: 0,1,2,4,5,6,...,60,61,62 | VERIFIED FACT | SET0-07 §5 (§203–219) |
| 3 | Linear-attention state model: recurrent state + convolution state | VERIFIED FACT | SET0-04 §9, §15.1; SET3-OC-4 §3.4 |
| 4 | 16 key heads | VERIFIED FACT | config.json linear_num_key_heads=16 |
| 5 | 48 value heads | VERIFIED FACT | config.json linear_num_value_heads=48 |
| 6 | 128-dimensional heads (linear_key_head_dim = linear_value_head_dim = 128) | VERIFIED FACT | config.json; SET0-03 §7, SET0-04 §6.1 |
| 7 | Convolution kernel dimension = 4 (linear_conv_kernel_dim = 4) | VERIFIED FACT | config.json; SET0-04 §8, SET3-OC-10 §3.10 |
| 8 | 9 per-layer tensors, 48/48 coverage, 432 total tensors | VERIFIED FACT | SET0-08 §2; SET3 §5.2 (§583); SET4-02 §3.4 |
| 9 | `in_proj_qkv.weight [10240, 5120]` | VERIFIED FACT | config.json; SET0-08 §2 §65 |
| 10 | `in_proj_z.weight [6144, 5120]` | VERIFIED FACT | config.json; SET0-08 §2 §66 |
| 11 | `in_proj_b.weight [48, 5120]` | VERIFIED FACT | config.json; SET0-08 §2 §67 |
| 12 | `in_proj_a.weight [48, 5120]` | VERIFIED FACT | config.json; SET0-08 §2 §68 |
| 13 | `out_proj.weight [5120, 6144]` | VERIFIED FACT | config.json; SET0-08 §2 §69 |
| 14 | `conv1d.weight [10240, 1, 4]` | VERIFIED FACT | config.json; SET0-08 §2 §70 |
| 15 | `A_log [48]` (recurrence parameter) | VERIFIED FACT | config.json; SET0-08 §2 §71 |
| 16 | `dt_bias [48]` (timestep bias parameter) | VERIFIED FACT | config.json; SET0-08 §2 §72 |
| 17 | `norm.weight [128]` (per-head norm) | VERIFIED FACT | config.json; SET0-08 §2 §73 |
| 18 | All checkpoint tensors are BF16 (2 bytes/element) | VERIFIED FACT | SET1-T1.6 §3 §516–518; SET3 §4.3 |
| 19 | `mamba_ssm_dtype = float32` is metadata field, NOT tensor dtype | VERIFIED FACT | SET3 §4.3 §519–524; SET3 §8 §921–924 |
| 20 | `use_cache = true` (config field) | VERIFIED FACT | config.json line 116 |
| 21 | `max_position_embeddings = 262144` (config field, relevant to state) | VERIFIED FACT | config.json; SET0-03 §4, SET3 §2.2 |
| 22 | QKV projection output = 10240 = 16×128 + 16×128 + 48×128 | VERIFIED FACT (matches tensor) | SET3-OC-4 §3.4 (§317) |
| 23 | Z projection output = 6144 = 48×128 | VERIFIED FACT (matches tensor) | SET3-OC-4 §3.4 (§319) |
| 24 | V head expansion ratio = 48/16 = 3 | DERIVED FINDING | SET0-04 §6.2 (§336) |
| 25 | Linear-attention execution stages include convolution + recurrent state update | VERIFIED FACT | SET3-OC-4 §3.4 §6.2 (§700–734) |
| 26 | Linear-attention state is STATEFUL (recurrent + conv) | VERIFIED FACT | SET4-01 RM-019 §523, RM-020 §542 (Stateful = Yes) |

### 13.2 CONDITIONAL MODEL / UNKNOWN (Linear-Attention State Runtime)

| # | Property | Classification | Status |
|---|---|---|---|
| 1 | Runtime recurrent state buffer shape | UNKNOWN (UK-001, UK-012) | Not established by any evidence source |
| 2 | Runtime convolution state buffer shape | UNKNOWN (UK-001, UK-012) | Not established by any evidence source |
| 3 | Runtime recurrent state dtype | UNKNOWN (UK-002) | No runtime observation |
| 4 | Runtime convolution state dtype | UNKNOWN (UK-002) | No runtime observation |
| 5 | Runtime recurrent state allocation strategy | UNKNOWN (UK-012) | No runtime observation |
| 6 | Runtime convolution state allocation strategy | UNKNOWN (UK-012) | No runtime observation |
| 7 | Whether state is pre-allocated to max_position_embeddings or grows per-token | UNKNOWN (UK-012) | No runtime observation |
| 8 | Whether A_log [48] / dt_bias [48] / value-head config defines runtime state dims | CONDITIONAL | Config-derived dimension 6144 = 48×128 exists but is NOT a verified runtime state shape |
| 9 | Whether conv1d [10240, 1, 4] kernel_dim=4 defines runtime conv state dims | CONDITIONAL | Kernel exists, runtime state buffer shape UNKNOWN |

---

## 14. Evidence Summary

### 14.1 Structural Evidence Established

The Executor established the following structural evidence for the linear-attention
state model:

1. **Layer topology** — 48 linear-attention layers out of 64 total (VERIFIED FACT).
   Layer indices fully enumerated (VERIFIED FACT). Pattern: `[LA → LA → LA → FA] × 16`
   (VERIFIED FACT + DERIVED FINDING).

2. **Head configuration** — 16 key heads, 48 value heads, 128-dim heads
   (VERIFIED FACT from config.json). KV head expansion ratio = 3 (DERIVED FINDING).

3. **State architecture** — Linear-attention uses recurrent state + convolution state,
   DISTINCT from full-attention KV cache (VERIFIED FACT). Mapped to OC-4
   (Qwen3_5GatedDeltaNet).

4. **Per-layer tensors** — 9 tensors per linear-attention layer, 48/48 coverage,
   432 total tensors (VERIFIED FACT with exact shapes). Key state-parameter tensors:
   `A_log [48]`, `dt_bias [48]`, `conv1d.weight [10240, 1, 4]` all VERIFIED FACT.

5. **Config fields** — All linear-attention config fields VERIFIED FACT from config.json.

6. **Dtype metadata** — All checkpoint tensors BF16 (VERIFIED FACT). `mamba_ssm_dtype =
   float32` is metadata only, not tensor dtype (VERIFIED FACT). Runtime state dtype =
   UNKNOWN (UK-002).

7. **Dataflow** — Linear-attention execution stages established (VERIFIED FACT for
   tensor shapes, DERIVED FINDING for stage ordering). Includes convolution stage
   (OC-10) and recurrent state update stage.

8. **Activation boundary** — T4.3 establishes AC-35 (gated-delta intermediates) and
   AC-36 (conv state boundary) as activation-side observations feeding into T4.5
   state modeling. Exact state shapes at the state level = UNKNOWN.

### 14.2 What Is NOT Established (Runtime)

No runtime evidence for linear-attention state was available:
- No model was loaded or executed.
- No inference engine, allocator, or kernel was observed.
- No state buffer (recurrent or convolution) was observed at runtime.
- No state allocation, layout, initialization, growth, persistence, or reuse was observed.
- All runtime state properties are classified UNKNOWN.

### 14.3 Conditional Models (Bounded)

The following conditional/config-derived dimensions exist but are NOT verified
runtime state shapes:

- Config-derived recurrent state dimension: `48 × 128 = 6144` (from
  `linear_num_value_heads × linear_value_head_dim`)
- Config-derived convolution input channels: `10240` (from `in_proj_qkv` output, also
  = `linear_num_value_heads × linear_value_head_dim` + key + query heads)
- Config-derived convolution kernel size: `4` (from `linear_conv_kernel_dim`)

These are structural derivations from verified config, NOT runtime state buffer
observations. They are CONDITIONAL — the actual runtime state shape depends on the
unresolved algorithm (UK-001).

---

## 15. Executor Result

### 15.1 Raw Report (Section A)

Actions actually performed:

1. **Repository inspection** — confirmed repository contains only `ROADMAP.md`, `docs/`,
   `model/` (metadata only). No runtime inference engine source code (.c, .cpp, .cc,
   .go, .js, .ts) present.
2. **Config inspection** — read `model/official/raw-checkpoint-metadata/config.json`
   (raw checkpoint metadata, 139 lines, fully parsed).
3. **Manifest inspection** — read `model/official/raw-checkpoint-metadata/manifest.json`
   (acquisition metadata, 388 lines, fully parsed).
4. **Source inspection** — read `model/official/SOURCE.md` (artifact provenance).
5. **SET0 inspection** — read `docs/set-0/03-core-architecture.md`,
   `docs/set-0/04-attention-architecture.md` (full 851 lines),
   `docs/set-0/06-vision-and-mtp.md`, `docs/set-0/07-layer-topology.md` (full 655 lines),
   `docs/set-0/08-tensor-shape-mapping.md` (full 214 lines).
6. **SET3 inspection** — read `docs/set-3/01-operator-computation-model.md`
   (full 1142 lines, including OC-4 §3.4, dataflow §6.3, §8 assumptions, §9 unknowns).
7. **SET4-01 inspection** — read `docs/set-4/01-runtime-memory-inventory.md` (full 1547
   lines, including RM-019, RM-020, RM-044, §4 UNKNOWN carry-forward, §5.2 taxonomy,
   §6 downstream mapping).
8. **SET4-02 inspection** — read `docs/set-4/02-weight-residency-model.md` (partial,
   §1–§4.4, linear-attention weight section).
9. **SET4-03 inspection** — read `docs/set-4/03-activation-lifetime-model.md` (full 1568
   lines, including AC-14, AC-15, AC-16, AC-35, AC-36, §19.3 T4.5 consumption,
   §12 shape model table, §14.3 persistence categories).
10. **SET4-04 inspection** — read `docs/set-4/04-full-attention-state-model.md`
    (full 914 lines) and `docs/set-4/04-full-attention-state-model-technical.md`
    (partial, §1–§3 — Orchestrator technical model, for format conventions only).
11. **ROADMAP inspection** — read SET4 task plan §2012–§2064 (T4.5 contract),
    §2218 (deliverables file structure), §2229–2247 (hard boundary),
    §2760–2884 (SET4 control state) to confirm T4.5 assignment and scope.
12. **Environment inspection** — verified Python 3.12.3, torch 2.11.0 (no GPU:
    `torch.cuda.is_available() = False`), transformers 5.3.0, jax 0.10.2. No
    inference engine running (ps inspection). No model loaded.

### 15.2 Raw Evidence (Section B)

Consolidated raw observations with provenance. Full detail in sections 3–12 above.

#### Structural Evidence (VERIFIED FACT / DERIVED FINDING)

| # | Observation | Source | Classification |
|---|---|---|---|
| 1 | 48 linear-attention layers | config.json `layer_types` (48× `"linear_attention"`) | VERIFIED FACT |
| 2 | Linear-attention layer indices (full list) | SET0-07 §5 | VERIFIED FACT |
| 3 | `[LA → LA → LA → FA] × 16` pattern | config.json `layer_types` + `full_attention_interval = 4` | VERIFIED FACT / DERIVED |
| 4 | 16 key heads | config.json `linear_num_key_heads = 16` | VERIFIED FACT |
| 5 | 48 value heads | config.json `linear_num_value_heads = 48` | VERIFIED FACT |
| 6 | 128 head dimension | config.json `linear_key_head_dim = 128`, `linear_value_head_dim = 128` | VERIFIED FACT |
| 7 | Conv kernel dim = 4 | config.json `linear_conv_kernel_dim = 4` | VERIFIED FACT |
| 8 | 9 tensors/layer × 48 layers = 432 tensors | config.json + SET0-08 tensor metadata | VERIFIED FACT |
| 9 | in_proj_qkv `[10240, 5120]` | Raw safetensors header | VERIFIED FACT |
| 10 | in_proj_z `[6144, 5120]` | Raw safetensors header | VERIFIED FACT |
| 11 | in_proj_b `[48, 5120]` | Raw safetensors header | VERIFIED FACT |
| 12 | in_proj_a `[48, 5120]` | Raw safetensors header | VERIFIED FACT |
| 13 | out_proj `[5120, 6144]` | Raw safetensors header | VERIFIED FACT |
| 14 | conv1d `[10240, 1, 4]` | Raw safetensors header | VERIFIED FACT |
| 15 | A_log `[48]` (recurrence parameter) | Raw safetensors header | VERIFIED FACT (parameter; state buffer UNKNOWN) |
| 16 | dt_bias `[48]` (timestep bias) | Raw safetensors header | VERIFIED FACT (parameter; state buffer UNKNOWN) |
| 17 | norm `[128]` (per-head norm) | Raw safetensors header | VERIFIED FACT |
| 18 | 10240 = 16×128 + 16×128 + 48×128 (QKV decomposition) | Arithmetic on config | VERIFIED FACT (matches tensor) / DERIVED |
| 19 | 6144 = 48×128 (V/Z dimension) | Arithmetic on config | VERIFIED FACT (matches tensor) |
| 20 | 48/16 = 3 (head expansion ratio) | Arithmetic on config | DERIVED FINDING |
| 21 | Linear-attention state = recurrent + conv state (≠ KV cache) | SET0-04 §9, §15.1; SET3-OC-4 §3.4 | VERIFIED FACT |
| 22 | All checkpoint tensors BF16, 2 bytes/element | SET1-T1.6 §3 | VERIFIED FACT |
| 23 | `mamba_ssm_dtype = float32` is metadata, not tensor dtype | SET3 §4.3 §519–524 | VERIFIED FACT |
| 24 | `use_cache = true` (config field) | config.json | VERIFIED FACT |
| 25 | `max_position_embeddings = 262144` | config.json | VERIFIED FACT |
| 26 | Linear-attention execution stages (conv, gated delta, recurrent state) | SET3-OC-4 §3.4 §6.3 | DERIVED FINDING |
| 27 | AC-35 gated-delta intermediates (activation boundary → T4.5) | T4.3 §11.1 | VERIFIED FACT (boundary) |
| 28 | AC-36 conv state boundary (activation boundary → T4.5) | T4.3 §11.2 | VERIFIED FACT (boundary) |
| 29 | MTP self_attn tensors exist but MTP runtime = UNKNOWN (UK-008) | SET0-06 §6, SET0-08 §3 | VERIFIED FACT (checkpoint) / UNKNOWN (runtime) |

#### Runtime Evidence (UNKNOWN)

| # | Observation | Source | Classification |
|---|---|---|---|
| 1 | No model loaded, no runtime, no allocator | Environment inspection (ps, repo scan) | VERIFIED FACT (no runtime present) |
| 2 | Runtime recurrent state dtype | Not observed | UNKNOWN (UK-002) |
| 3 | Runtime conv state dtype | Not observed | UNKNOWN (UK-002) |
| 4 | Runtime recurrent state buffer shape | Not observed | UNKNOWN (UK-001, UK-012) |
| 5 | Runtime conv state buffer shape | Not observed | UNKNOWN (UK-001, UK-012) |
| 6 | Runtime recurrent state allocation strategy | Not observed | UNKNOWN (UK-012) |
| 7 | Runtime conv state allocation strategy | Not observed | UNKNOWN (UK-012) |
| 8 | Runtime state pre-allocation vs per-token growth | Not observed | UNKNOWN (UK-012) |
| 9 | Runtime state layout (head-major / seq-major) | Not observed | UNKNOWN |
| 10 | Runtime state reuse / circular / paged | Not observed | UNKNOWN (UK-012) |
| 11 | Runtime state quantization | Not observed | UNKNOWN |
| 12 | Runtime batch size (B) | Not observed | UNKNOWN (UK-009) |
| 13 | Runtime sequence length (S) | Not observed | UNKNOWN (UK-009) |
| 14 | Runtime state initialization | Not observed | UNKNOWN |
| 15 | Runtime state persistence | Not observed | UNKNOWN |
| 16 | GPU/NPU accessibility from WSL2 guest | VERIFIED ABSENT | VERIFIED FACT |

### 15.3 Result (Section C)

The Executor interpretation of the evidence:

1. **The linear-attention state domain is structurally bounded.** There are 48
   `linear_attention` layers, each mapped to the OC-4 (Qwen3_5GatedDeltaNet) operator.
   The state model is structurally distinct from full-attention KV cache: it is
   recurrent state + convolution state. This distinction is VERIFIED FACT across
   SET0-04 §9/§15.1, SET3-OC-4 §3.4, and SET4-01 RM-019/RM-020.

2. **The state-relevant checkpoint parameters are fully identified.** All 9 per-layer
   linear-attention tensors (432 total across 48 layers, 48/48 coverage) are VERIFIED
   FACT with exact shapes from raw safetensors headers. The state-parameterizing tensors
   (`A_log [48]`, `dt_bias [48]`, `conv1d.weight [10240, 1, 4]`) are VERIFIED FACT.
   However, these are checkpoint *parameters*, NOT runtime state buffers.

3. **The exact runtime state buffer shape is UNKNOWN.** While config-derived dimensions
   exist (e.g., `48 × 128 = 6144` from value heads, `conv1d` input channels = 10240,
   kernel_size = 4), these are CONDITIONAL — the actual runtime state buffer shape
   depends on the unresolved linear-attention algorithm (UK-001) and runtime allocation
   strategy (UK-012). No evidence source establishes the runtime state buffer shape as a
   VERIFIED FACT.

4. **No runtime evidence is available.** The environment contains no inference engine,
   no loaded model, no state allocator, and no GPU/NPU accessible from WSL2 guest.
   All runtime state behaviors (dtype, allocation, layout, initialization, growth,
   persistence, reuse, quantization, batch/sequence dependence) are classified UNKNOWN.
   This is consistent with the T4.4 evidence environment conclusion (SET4-04 §2.1).

5. **The linear-attention state domain is kept strictly separate** from full-attention
   KV cache (T4.4), weight residency (T4.2), activation memory (T4.3), and workspace
   (T4.6). The Executor did NOT merge these domains.

```text
RESULT
=
NON-AUTHORITATIVE
ORCHESTRATOR REVIEW REQUIRED
```

The Executor does NOT declare T4.5 PASS, COMPLETE, TECHNICAL MODEL ACCEPTED, MEMORY MODEL
VERIFIED, or STATE FORMULA ACCEPTED. Those are ORCHESTRATOR responsibilities only.

### 15.4 Executor Recommendation (Section D)

```text
NON-AUTHORITATIVE
NEVER PROJECT AUTHORITY
```

The following recommendations are evidence-driven observations for the Orchestrator's
analysis only. They do not constitute authorization, acceptance, or readiness:

1. **UK-001 (exact linear-attention algorithm) is the primary blocking unknown for
   state buffer shape derivation.** The config provides head dimensions (16 K heads,
   48 V heads, 128 dim) and the conv kernel dimension (4), but the runtime recurrent
   state buffer shape is conditional on the algorithm identity. The Orchestrator should
   analyze UK-001 to determine whether the recurrent state dimension is `48` (matching
   `A_log`/`dt_bias`), `48 × 128 = 6144` (matching value representation), or some other
   configuration, and whether the convolution state shape is `[B, 10240, 4]` or differs.

2. **UK-002 (`mamba_ssm_dtype = float32`) is the primary blocking unknown for state
   dtype.** The config field is VERIFIED FACT as metadata-only. Whether it implies
   runtime state in float32 (vs the checkpoint BF16) is UNKNOWN. The Orchestrator should
   resolve this to determine `E_state` for the linear-attention state memory model.

3. **UK-012 (runtime linear-attention state allocation) is the primary blocking unknown
   for allocation strategy.** Even if the state shape is derived from config, the
   allocation strategy (pre-allocated vs per-token growth, circular vs paged vs
   contiguous, per-layer vs combined buffer, head-major vs seq-major layout) is
   UNKNOWN. The Orchestrator should treat the state memory as a CONDITIONAL MODEL
   parameterized on these unresolved runtime behaviors.

4. **The activation boundary (AC-36) connects T4.3 to T4.5.** The QKV projection output
   `[B, S, 10240]` (AC-14) feeds the convolution state stage. This is the structurally
   verified input to the T4.5 domain. The Orchestrator may use this as the entry point
   for state modeling.

5. **The `A_log`/`dt_bias` parameters vs runtime state buffer distinction** should be
   preserved: these are checkpoint parameters (VERIFIED FACT) that parameterize the
   recurrence, but the runtime state buffer is a separate object (UNKNOWN). Do not
   conflate `A_log [48]` with the runtime recurrent state buffer shape.

The recommendation does NOT determine:
- Current SET
- Next task
- Authorization
- Acceptance
- Readiness
- Revision

---

## 16. Unknown / Missing

### 16.1 Structural Evidence Gaps (Not Established)

| # | Missing Evidence | Status |
|---|---|---|
| 1 | Exact runtime recurrent state buffer shape | UNKNOWN (UK-001, UK-012) |
| 2 | Exact runtime convolution state buffer shape | UNKNOWN (UK-001, UK-012) |
| 3 | Whether `A_log [48]` dimension = runtime recurrent state dimension | UNKNOWN (UK-001) |
| 4 | Whether `dt_bias [48]` dimension = runtime recurrent state dimension | UNKNOWN (UK-001) |
| 5 | Whether `conv1d.weight [10240, 1, 4]` kernel_size=4 = runtime conv state length | UNKNOWN (UK-001) |
| 6 | Whether `in_proj_qkv` output (10240) = runtime conv state input channels | UNKNOWN (UK-001) |
| 7 | Whether `in_proj_z` output (6144 = 48×128) = runtime recurrent state dimension | UNKNOWN (UK-001) |
| 8 | Whether K/Q heads are expanded to 48 V heads before state storage | UNKNOWN (UK-001) |
| 9 | Number of distinct state objects per linear-attention layer | UNKNOWN (UK-001) |
| 10 | Whether the recurrent state is a single tensor or partitioned (per-group) | UNKNOWN (UK-001) |

### 16.2 Runtime Evidence Gaps (Not Established)

| # | Missing Runtime Evidence | Status |
|---|---|---|
| 1 | Runtime recurrent state dtype | UNKNOWN (UK-002) |
| 2 | Runtime convolution state dtype | UNKNOWN (UK-002) |
| 3 | Runtime recurrent state allocation strategy | UNKNOWN (UK-012) |
| 4 | Runtime convolution state allocation strategy | UNKNOWN (UK-012) |
| 5 | Runtime pre-allocation vs per-token growth | UNKNOWN (UK-012) |
| 6 | Runtime state element layout (head-major / seq-major / blocked) | UNKNOWN |
| 7 | Runtime state contiguity / striding | UNKNOWN |
| 8 | Runtime per-layer vs combined buffer allocation | UNKNOWN |
| 9 | Runtime state reuse (circular / paged / contiguous) | UNKNOWN (UK-012) |
| 10 | Runtime state quantization | UNKNOWN |
| 11 | Runtime state initialization behavior | UNKNOWN |
| 12 | Runtime state growth / update pattern | UNKNOWN |
| 13 | Runtime state persistence horizon | UNKNOWN |
| 14 | Runtime batch dependency | UNKNOWN (UK-009) |
| 15 | Runtime sequence dependency | UNKNOWN (UK-009) |
| 16 | Runtime batch size (B) | UNKNOWN (UK-009) |
| 17 | Runtime sequence length (S) | UNKNOWN (UK-009) |
| 18 | Runtime state sharing across beam-search candidates | UNKNOWN |
| 19 | Runtime state eviction/paging to disk | UNKNOWN |
| 20 | Whether recurrent and conv state share an allocator | UNKNOWN |
| 21 | Runtime TP / device distribution effect on state | UNKNOWN |
| 22 | Whether `use_cache = true` triggers actual LA state caching | UNKNOWN |
| 23 | Whether `mamba_ssm_dtype = float32` implies runtime state in float32 | UNKNOWN (UK-002) |

### 16.3 Environment Evidence Gaps (Not Observed)

| # | Missing | Status |
|---|---|---|
| 1 | GPU / accelerator | VERIFIED ABSENT from WSL2 guest |
| 2 | Runtime libraries (live) | Python packages present but no model loaded |
| 3 | Model implementation (inference engine source) | NOT in repository |
| 4 | Inference engine | NOT running |
| 5 | Allocator | NOT running |
| 6 | Kernel implementation | NOT in repository |
| 7 | Runtime process | No model-loading process observed |
| 8 | Loaded model | No model loaded |

---

## 17. Commit / Push Verification

### 17.1 Git Status (Pre-Commit)

```text
Repository: /home/kawee/Code/project/qwen3.8-27b-in-c
Branch: main
HEAD: 71c7adc9c53cbd4c15862f735665cdc24e5034f8
```

### 17.2 Files Modified

Only `docs/set-4/05-linear-attention-state-model.md` created (new file).
No existing files modified. No T4.4 documents changed. No SET2/SET3 historical
records changed.

### 17.3 Remote Verification

See Executor Result section 15.1 for the sequence of verification commands and results.

---

## STOP

This is the conclusion of the SET4-T4.5 evidence-acquisition boundary.

The Executor has:
1. Acquired all structural evidence for linear-attention state architecture, layer topology,
   head dimensions, projection tensors, config fields, state-relevant dataflow, and
   activation boundaries from authoritative sources (config.json, SET0, SET1, SET3, SET4-01/02/03/04).
2. Inspected the runtime/environment and confirmed no model was loaded, no inference
   engine was running, and no GPU/NPU is accessible from the WSL2 guest.
3. Classified every observation as VERIFIED FACT, DERIVED FINDING, CONDITIONAL MODEL,
   or UNKNOWN.
4. Preserved all UK-001, UK-002, UK-012 unknowns as UNKNOWN — not promoted to facts.
5. Created and persisted the evidence artifact at `docs/set-4/05-linear-attention-state-model.md`.
6. Committed and pushed the artifact to remote.

The Executor did NOT design the T4.5 formula, derive the authoritative equation,
perform mathematical derivation, calculate final state memory, declare T4.5 PASS/COMPLETE,
authorize T4.6, start T4.6/T4.7/SET5, modify SET2 history, reopen SET2, create SET2
revisions, or redesign T4.4.

Control of T4.5 technical design, formula construction, interpretation, and acceptance
remains with the ORCHESTRATOR.
