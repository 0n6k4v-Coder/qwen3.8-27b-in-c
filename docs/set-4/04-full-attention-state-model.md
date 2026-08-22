# SET 4 — Full-Attention State Model — Evidence Acquisition

## Document Status

- Document: `docs/set-4/04-full-attention-state-model.md`
- SET: `SET 4 — Runtime Memory Model`
- Source Task: `SET4-T4.4`
- Status: EVIDENCE ACQUIRED (Executor — NON-AUTHORITATIVE — ORCHESTRATOR REVIEW REQUIRED)
- Responsibility: Execution Support (Executor — evidence acquisition only)
- Control State: `SET4-T4.3 = PASS` (Dependency satisfied)
- Control Task Owner: EXECUTOR (evidence acquisition)
- Technical Design Owner: ORCHESTRATOR (model construction, formula construction, interpretation, acceptance)

---

## 1. Source and Provenance

### 1.1 Authoritative Model

- Model: `Qwen/Qwen3.8-27B`
- Official repository: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

### 1.2 Primary Evidence Sources

All observations below are traced to the following authoritative project artifacts:

```text
SET 4 (direct input):
  docs/set-4/01-runtime-memory-inventory.md        — T4.1 runtime memory inventory (RM-012, RM-013, RM-014, RM-015, RM-016)
  docs/set-4/02-weight-residency-model.md          — T4.2 weight residency (dtype, config)
  docs/set-4/03-activation-lifetime-model.md      — T4.3 activation lifetime model (AC-05, AC-07 boundary; §19.1 T4.4 Consumption)

SET 3 (operator/computation model — accepted input):
  docs/set-3/01-operator-computation-model.md     — OC-3 operator, §3.3, §6.2, §4.3, §9 unknowns (UK-001..UK-015)
    Specifically: §3.3 full-attention config + verified tensor set;
                   §6.2 full-attention dataflow (KV Cache store stage);
                   §9 Unknowns (UK-001 through UK-015)

SET 0 (structural truth):
  docs/set-0/03-core-architecture.md              — §4 language config (hidden_size, num_hidden_layers, dtype, etc.)
  docs/set-0/04-attention-architecture.md         — §3 Full Attention config; §3.2 Q/K/V organization; §3.3 execution stages; §3 Full-Attention KV Cache; §11 attention comparison; §15.1 two state models
  docs/set-0/07-layer-topology.md                 — §2 topology summary; §4 full-attention layer indices (16); §5 linear-attention indices (48)
  docs/set-0/08-tensor-shape-mapping.md           — §2 Full Attention tensor shapes (q_proj, k_proj, v_proj, o_proj, q_norm, k_norm)

SET 1 (checkpoint storage truth):
  docs/set-1/02-parameter-reconstruction.md         — subsystem parameter totals
  docs/set-1/03-tensor-byte-accounting.md         — logical byte totals

Raw artifacts:
  model/official/raw-checkpoint-metadata/config.json    — raw config.json (source of VERIFIED FACT config fields)
  model/official/raw-checkpoint-metadata/manifest.json  — acquisition metadata
  model/official/SOURCE.md                               — artifact source provenance

ROADMAP.md — SET4 control state, atomic task plan (§2142 SET4-T4.4 task contract)
```

### 1.3 Classification Schema

Every material assertion in this document is classified as exactly one of:

- **VERIFIED FACT** — directly supported by raw SET1 tensor metadata, SET0 configuration fields, or established SET3 operator/dataflow evidence.
- **DOCUMENTED CAPABILITY** — sourced from authoritative external documentation; not promoted to VERIFIED FACT.
- **DERIVED FINDING** — arithmetic or logical combination of verified evidence. Explicitly labeled.
- **CONDITIONAL MODEL** — a memory property expressed as an explicit dependency on an unresolved runtime behavior. Bounded, parameterized, not an assumption.
- **UNKNOWN** — runtime behavior or structural detail not established by available evidence. Treated as a boundary.

### 1.4 Critical Distinctions

```text
CONFIG-DERIVED STATE SHAPE ≠ RUNTIME CACHE ALLOCATION STRATEGY ≠ RUNTIME CACHE LAYOUT
STRUCTURAL TRUTH ≠ RUNTIME IMPLEMENTATION TRUTH
```

This document establishes the **evidence** for full-attention state: the verified structural parameters that bound the KV-cache memory model, and the explicit boundaries where runtime implementation behavior is UNKNOWN and must be resolved by the ORCHESTRATOR.

This document does NOT:
- Design the final full-attention memory formula (ORCHESTRATOR)
- Derive the final KV-cache memory equation (ORCHESTRATOR)
- Calculate the final peak full-attention memory requirement (ORCHESTRATOR)
- Declare T4.4 technically PASS (ORCHESTRATOR)
- Declare T4.4 complete (ORCHESTRATOR)

---

## 2. Evidence Acquisition Protocol and Environment

### 2.1 Environment Inspection Performed

The assigned environment was inspected for the presence of a runtime, inference engine, or live model instance capable of exposing runtime KV-cache behavior.

```text
Repository code artifacts:
  - No runtime inference engine source code (.c, .cpp, .cc, .go, .js, .ts) found in repository.
  - No model loading, compilation, or execution code found on disk.
  - Repository contains only: ROADMAP.md, docs/, model/ (metadata only).

Python environment:
  - python3: /usr/bin/python3 (3.12.3)
  - torch: 2.11.0 (installed, not used for model loading — no model weights loaded)
  - transformers: 5.3.0 (installed, not used — no model instantiated)
  - safetensors: 0.7.0 (installed, used for metadata acquisition only)
  - jax: 0.10.2, jaxlib: 0.10.2 (installed, not used)
  - numpy: 2.4.3 (installed, not used)
  - huggingface-hub: NOT used (no model download performed)

Hardware (per SET2 evidence):
  - CPU: Intel Core Ultra 7 155H (Meteor Lake), 16C/22T host, 4C/8T guest (WSL2 cgroup)
  - GPU: Intel Arc 7D55 — VERIFIED ABSENT from WSL2 guest (SET2-T2.8 §8.1)
  - NPU: Intel AI Boost 7D1D — VERIFIED ABSENT from WSL2 guest (SET2-T2.8 §8.1)
  - RAM: 16 GB host, 12 GB WSL2 cgroup cap, dual-channel, no ECC

Runtime observation conclusion:
  - No model was loaded, instantiated, or executed during this evidence-acquisition task.
  - No KV-cache allocator, memory allocator, or inference engine was running.
  - The only runtime-relevant configuration field observable in the artifact is `use_cache = true`
    (config.json text_config field).
  - Therefore, ALL runtime-specific observations (KV cache allocation strategy, cache
    dtype at runtime, allocator reuse behavior, cache layout, page structure) are
    UNKNOWN and cannot be established from direct observation.
```

**Classification:** VERIFIED FACT (environment state: no runtime present). UNKNOWN (all runtime behaviors — no live execution occurred).

### 1.5 T4.4 Task Contract (from ROADMAP.md §2142)

The ROADMAP.md SET4 task contract for T4.4 states:

```text
SET4-T4.4 — Full-Attention State Model
  - Establish the verified and derived memory requirements of full-attention state,
    including K/V state where applicable.
  - Model dependency on batch size, sequence length, number of full-attention layers,
    KV heads, head dimension, dtype, and K/V multiplicity.
  - Produce parameterized formulas rather than a single unqualified memory number.
  - Clearly distinguish derived KV-memory requirements from observed runtime allocator behavior.
```

This evidence acquisition satisfies the first bullet (establish verified + derived parameters). Formula construction, dependency modeling, and final KV-memory derivation are explicitly ORCHESTRATOR responsibilities (§1.4, DO-NOT-RUN).

### 1.6 Executor Responsibility Boundary (from ROADMAP.md §2120–§2132, Hard Rule #5)

```text
🧠 LUNA owns: Technical design, mathematical derivation, interpretation,
  capability modeling, constraint synthesis, sequencing, acceptance decisions.
  Formula construction, memory-model construction, final technical conclusion.

🛠 EXECUTOR owns: Local environment access, file operations, terminal execution,
  evidence acquisition, provenance capture, evidence persistence,
  measurements explicitly assigned.
```

This document is the Executor's evidence-acquisition deliverable. The ORCHESTRATOR will independently review, design the formula, derive the equation, interpret the evidence, and accept the technical design.

---

## 3. Raw Structural Evidence — Full-Attention Layer Count

### 3.1 Full-Attention Layer Count

| Field | Value | Source | Source Type | Classification | Provenvenance |
|---|---|---|---|---|---|
| Total language layers | 64 | config.json: `text_config.num_hidden_layers = 64` | Checkpoint config artifact | VERIFIED FACT | SET0 §2.2 (§142 config fields); SET3 §2.2 (§98); SET0-07 §2 (§72) |
| Full-attention layers | 16 | config.json: `text_config.layer_types` array (64 entries, 16 × `"full_attention"`) | Checkpoint config artifact | VERIFIED FACT | SET0-04 §2 (§58: `16 × full_attention`); SET0-07 §2, §4 (§80: `48 × linear_attention`, `16 × full_attention`; §183 full-attention index set count = 16); SET3 §2.3 (§119: full-attention indices count = 16) |
| Linear-attention layers | 48 | config.json: `text_config.layer_types` array (48 × `"linear_attention"`) | Checkpoint config artifact | VERIFIED FACT (carried, not T4.4 focus) | SET0-04 §2 (§58); SET0-07 §2 (§80); SET3 §2.3 |
| Full-attention layer indices | 3, 7, 11, 15, 19, 23, 27, 31, 35, 39, 43, 47, 51, 55, 59, 63 | config.json: `text_config.layer_types` array (explicit 64-entry sequence) | Checkpoint config artifact | VERIFIED FACT | SET0-07 §4 (§183–§194); SET3 §2.3 (§119) |

### 3.2 Layer Pattern / Cadence

| Field | Value | Source | Source Type | Classification | Proven provenance |
|---|---|---|---|---|---|
| Layer pattern | `[LA → LA → LA → FA] × 16` | config.json: `text_config.layer_types` array + `full_attention_interval = 4` | Checkpoint config artifact | VERIFIED FACT (pattern); DERIVED FINDING (×16 shorthand) | SET0-03 §4 (§216: `[LA → LA → LA → FA] × 16`); SET0-07 §2 (§86), §7 (§268), §11 (§329 layer→operator mapping) |
| Full-attention interval | 4 | config.json: `text_config.full_attention_interval = 4` | Checkpoint config artifact | VERIFIED FACT; CONSISTENCY VERIFIED (interval == array spacing) | SET0-03 §4 (§216–§244: 64/4=16, 24/4=6, etc.); SET0-04 §8 (§412: `full_attention_interval = 4` consistent with `[LA → LA → LA → FA] × 16`); SET3 §2.3 (§124) |

**Classification note:** The layer pattern is VERIFIED FACT from the explicit 64-entry `layer_types` array. The `× 16` shorthand is DERIVED FINDING (64 ÷ 4 = 16). This evidence is authoritative input for T4.4 but is not T4.4-specific work.

---

## 4. Raw Structural Evidence — Full-Attention Head Structure

### 4.1 Full-Attention Head Configuration

All values below are read directly from `config.json` (`text_config` section).

| Field | Value | Source | Source Type | Classification | Proven provenance |
|---|---|---|---|---|---|
| `num_attention_heads` | 24 | config.json: `text_config.num_attention_heads = 24` | Checkpoint config artifact | VERIFIED FACT | SET0-04 §3.1 (§102: `num_attention_heads: 24`); SET0-03 §6 (§266–§267); SET3-OC-3 §3.3 (§228); T4.1 RM-002 (§160), RM-014 (§421) |
| `num_key_value_heads` | 4 | config.json: `text_config.num_key_value_heads = 4` | Checkpoint config artifact | VERIFIED FACT | SET0-04 §3.1 (§104: `num_key_value_heads: 4`); SET0-03 §6 (§269); SET3-OC-3 §3.3 (§229); T4.1 RM-002 (§160), RM-012 (§383), RM-013 (§402), RM-014 (§421) |
| `head_dim` | 256 | config.json: `text_config.head_dim = 256` | Checkpoint config artifact | VERIFIED FACT | SET0-04 §3.1 (§108: `head_dim: 256`); SET0-03 §6 (§272); SET3-OC-3 §3.3 (§230); T4.1 RM-002 (§160), RM-012 (§384), RM-013 (§403) |
| `attention_bias` | false | config.json: `text_config.attention_bias = false` | Checkpoint config artifact | VERIFIED FACT (config field; relevance to state = structural) | SET0-04 §3.1 (§111); SET3-OC-3 §3.3 (§231) |
| `attention_dropout` | 0.0 | config.json: `text_config.attention_dropout = 0.0` | Checkpoint config artifact | VERIFIED FACT (config field; relevance to state = structural) | SET0-04 §3.1 (§114); SET3-OC-3 §3.3 (§232) |

### 4.2 Derived Head Relationships

| Field | Value | Source | Source Type | Classification | Proven provenance |
|---|---|---|---|---|---|
| GQA grouping ratio | 24 / 4 = 6 | Arithmetic on config fields above | DERIVED FINDING | SET3-OC-3 §3.3 (§256–§258: `24 / 4 = 6`); SET0-04 §3.2 (§138: `24 / 4 = 6`) |
| Q projection output dim | 24 × 256 = 6144 (logical); checkpoint tensor = 12288 | config.json `num_attention_heads × head_dim` vs `self_attn.q_proj.weight [12288, 5120]` | DERIVED FACT (logical = 6144); VERIFIED FACT (checkpoint tensor shape = 12288) | SET3-OC-3 §3.3 (§257: `24 × 256 = 6144` → but q_proj output is 12288, includes QK or gate dimension); SET0-08 §2 (§54: `self_attn.q_proj.weight [12288, 5120]`); T4.1 RM-002 (§154, §160), RM-014 (§421) |
| K projection output dim | 4 × 256 = 1024 | config.json `num_key_value_heads × head_dim`; checkpoint `self_attn.k_proj.weight [1024, 5120]` = 1024 output rows | VERIFIED FACT (matches) | SET3-OC-3 §3.3 (§260: `4 × 256 = 1024 → k_proj output [1024, 5120]`); SET0-08 §2 (§55); T4.1 RM-002 (§154, §160), RM-014 (§421) |
| V projection output dim | 4 × 256 = 1024 | config.json `num_key_value_heads × head_dim`; checkpoint `self_attn.v_proj.weight [1024, 5120]` = 1024 output rows | VERIFIED FACT (matches) | SET3-OC-3 §3.3 (§261); SET0-08 §2 (§56); T4.1 RM-002 (§154, §160), RM-014 (§421) |
| O projection input dim | 24 × 256 = 6144 | config.json `num_attention_heads × head_dim`; checkpoint `self_attn.o_proj.weight [5120, 6144]` = 6144 input rows | VERIFIED FACT (matches) | SET3-OC-3 §3.3 (§262: `o_proj output [5120, 6144]`); SET0-08 §2 (§57); T4.1 RM-002 (§154, §160) |

**Important boundary (VERIFIED FACT, not to be silently resolved):** The Q projection output dimension is 12288 in the checkpoint tensor, but the logical `num_attention_heads × head_dim = 24 × 256 = 6144`. SET3-OC-3 §3.3 explicitly notes: "includes QK or gate dimension from GQA implementation" and the exact decomposition is UNKNOWN (UK-003 in SET3 §9). The checkpoint tensor shape `[12288, 5120]` is VERIFIED FACT; whether 12288 = 2 × 6144 (Q + K-like) or includes a gate dimension is UNKNOWN.

---

## 5. Raw Structural Evidence — K/V Projection Structure

### 5.1 Verified Full-Attention Projection Tensors

Per-layer tensor set (16/16 coverage VERIFIED:

| Tensor Name | Shape | Coverage | Source | Classification | Proven provenance |
|---|---|---|---|---|---|
| `self_attn.q_proj.weight` | `[12288, 5120]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§54); SET3-OC-3 §3.3 (§246–§249); T4.1 RM-002 (§154) |
| `self_attn.k_proj.weight` | `[1024, 5120]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§55); SET3-OC-3 §3.3 (§247); T4.1 RM-002 (§154) |
| `self_attn.v_proj.weight` | `[1024, 5120]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§56); SET3-OC-3 §3.3 (§248); T4.1 RM-002 (§154) |
| `self_attn.q_norm.weight` | `[256]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§58); SET3-OC-3 §3.3 (§250); T4.1 RM-002 (§154) |
| `self_attn.k_norm.weight` | `[256]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§59); SET3-OC-3 §3.3 (§251); T4.1 RM-002 (§154) |
| `self_attn.o_proj.weight` | `[5120, 6144]` | 16/16 | Raw checkpoint metadata (safetensors header) | VERIFIED FACT | SET0-08 §2 (§57); SET3-OC-3 §3.3 (§249); T4.1 RM-002 (§154) |

**Tensor count:** 6 tensors × 16 layers = 96 tensors total (VERIFIED FACT). Source: SET3 §5.2 (§582: "Full-attention layers: 96 tensors (6 per layer × 16)"), SET0-08 §2, T4.1 RM-002 (§155: "6 tensors × 16 layers = 96 tensors (16/16 coverage)").

### 5.2 Q/K/V Projection Output Shapes (Derived from Checkpoint Tensor Shapes)

| Projection | Output Dimension (from weight matrix rows) | Shape (parameterized by B, S) | Source | Classification |
|---|---|---|---|---|
| Q projection | 12288 | `[B, S, 12288]` | `q_proj.weight [12288, 5120]` | VERIFIED FACT (shape from raw tensor) |
| K projection | 1024 | `[B, S, 1024]` = `[B, S, 4 × 256]` | `k_proj.weight [1024, 5120]` | VERIFIED FACT (shape from raw tensor) |
| V projection | 1024 | `[B, S, 1024]` = `[B, S, 4 × 256]` | `v_proj.weight [1024, 5120]` | VERIFIED FACT (shape from raw tensor) |

Source: T4.1 RM-014 (§421: q_proj `[batch, seq, 12288]`; k_proj `[batch, 4, seq, 256]`; v_proj `[batch, 4, seq, 256]`); SET0-08 §2 (§54–57); SET3-OC-3 §3.3 (§260–262); T4.3 AC-05 (§205–213).

### 5.3 Q/K Normalization Tensors

| Tensor | Shape | Source | Classification | Proven provenance |
|---|---|---|---|---|
| `self_attn.q_norm.weight` | `[256]` | Raw checkpoint metadata | VERIFIED FACT | SET0-08 §2 (§58); SET3-OC-3 §3.3 (§250); T4.1 RM-002 (§154, §160) |
| `self_attn.k_norm.weight` | `[256]` | Raw checkpoint metadata | VERIFIED FACT | SET0-08 §2 (§59); SET3-OC-3 §3.3 (§251); T4.1 RM-002 (§154, §160) |
| `input_layernorm.weight` | `[5120]` | Raw checkpoint metadata | VERIFIED FACT | SET0-08 §2 (§50); SET3-OC-6 §3.6 (§403–404); T4.1 RM-005 (§221) |
| `post_attention_layernorm.weight` | `[5120]` | Raw checkpoint metadata | VERIFIED FACT | SET0-08 §2; SET3-OC-6 §3.6; T4.1 RM-005 (§221) |

### 5.4 RoPE Configuration (Relevant to State)

| Field | Value | Source | Classification | Proven provenance |
|---|---|---|---|---|
| `rope_theta` | 10000000 | config.json: `text_config.rope_parameters.rope_theta = 10000000` | VERIFIED FACT | SET0-04 §4 (§224: `rope_theta: 10000000`); SET3-OC-3 §3.3 (§236); T4.1 RM-016 (§457) |
| `partial_rotary_factor` | 0.25 | config.json: `text_config.partial_rotary_factor = 0.25` | VERIFIED FACT | SET0-04 §4 (§221); SET3-OC-3 §3.3 (§235); T4.1 RM-016 (§457) |
| `rope_type` | "default" | config.json: `text_config.rope_parameters.rope_type = "default"` | VERIFIED FACT | SET0-04 §4 (§227); SET3-OC-3 §3.3 (§237) |
| `mrope_interleaved` | true | config.json: `text_config.rope_parameters.mrope_interleaved = true` | VERIFIED FACT | SET0-04 §4 (§230); SET3-OC-3 §3.3 (§238) |
| `mrope_section` | [11, 11, 10] | config.json: `text_config.rope_parameters.mrope_section = [11, 11, 10]` | VERIFIED FACT | SET0-04 §4 (§233); SET3-OC-3 §3.3 (§239) |
| Rotary dimension | 256 × 0.25 = 64 | Arithmetic on `head_dim × partial_rotary_factor` | DERIVED FINDING | SET0-04 §4 (§247: `256 × 0.25 = 64`); SET3-OC-3 §3.3 (§275); T4.1 RM-016 (§458) |
| `max_position_embeddings` | 262144 | config.json: `text_config.max_position_embeddings = 262144` | VERIFIED FACT | SET0-03 §4 (§154); SET3 §2.2 (§97); T4.3 AC-30 (§697) |

### 5.5 Full-Attention Execution Stages (From SET0-04 §3.3)

The full-attention operator is documented as executing through these stages:

```text
Hidden States → Q/KV projection → Q/K normalization → RoPE → KV Cache → Causal Attention → Output Gating → Output Projection
```

Source: SET0-04 §3.3 (§182–§198), SET3-OC-3 §3.3 §6.2 (§264–268). Classification: DERIVED FINDING (stage ordering is described by the official implementation, not by checkpoint metadata alone — SET3-OC-3 §3.3 §6.2 says "Classification: DERIVED FINDING — the stage ordering is described by the official implementation").

This establishes that the KV cache store stage is a boundary point between full-attention activation memory (T4.3) and full-attention state memory (T4.4).

---

## 6. Raw Evidence — Full-Attention KV Cache State

### 6.1 KV Cache Presence and Configuration

| Field | Value | Source | Source Type | Classification | Proven provenance |
|---|---|---|---|---|---|
| Full-attention uses KV cache | VERIFIED (yes) | SET0-04 §3.3: "full-attention layers use a conventional attention history represented by a KV cache" (§201); SET0-04 §3.2 execution characteristics include "KV cache" (§208); SET3-OC-3 §6.2 full-attention dataflow shows "KV Cache" stage | Document evidence (SET0, SET3) | VERIFIED FACT | SET0-04 §3.2 (§177–212: verified implementation-level findings include "KV cache"); SET0-04 §3.3 (§200–201: "conventional attention history represented by a KV cache"); SET3-OC-3 §3.3 §6.2 (§264–268) |
| `use_cache` config field | true | config.json: `text_config.use_cache = true` | Checkpoint config artifact | VERIFIED FACT (config field only; runtime behavior of this flag is UNKNOWN) | T4.1 RM-012 (§384), RM-013 (§403); SET3-OC-3 §3.3 |

### 6.2 KV Cache Tensor Shapes (Config-Derived vs Runtime)

| Property | Value (Config-Derived) | Value (Runtime) | Source | Classification |
|---|---|---|---|---|
| KV cache K shape (config-derived) | `[batch, num_key_value_heads, seq_len, head_dim]` = `[B, 4, S, 256]` | UNKNOWN — not observed | T4.1 RM-012 §3.75 (§383); SET0-04 §3.2 (§170–174: not present); SET3-OC-3 §3.3 | CONDITIONAL MODEL (config-derived); UNKNOWN (runtime observation) |
| KV cache V shape (config-derived) | `[batch, num_key_value_heads, seq_len, head_dim]` = `[B, 4, S, 256]` | UNKNOWN — not observed | T4.1 RM-013 §3.95 (§402); SET0-04 §3.2; SET3-OC-3 §3.3 | CONDITIONAL MODEL (config-derived); UNKNOWN (runtime observation) |
| KV heads per layer | 4 | UNKNOWN at runtime | config.json `num_key_value_heads = 4` | VERIFIED FACT (config); UNKNOWN (runtime cache behavior) |
| KV heads across all full-attention layers | 4 × 16 = 64 | UNKNOWN at runtime | Arithmetic on config fields | DERIVED FINDING (arithmetic) |
| Head dimension in cache | 256 | UNKNOWN at runtime | config.json `head_dim = 256` | VERIFIED FACT (config); UNKNOWN (runtime cache behavior) |
| Query heads per KV head (GQA group) | 6 (24/4) | N/A (queries not cached) | Arithmetic on config fields | DERIVED FINDING (GQA ratio) |

**Critical distinction (VERIFIED FACT):** The config-derived KV cache shape `[B, 4, S, 256]` is a CONDITIONAL MODEL — it represents the shape inferred from the verified config dimensions. Whether the runtime cache actually uses this shape, whether K and V are stored separately or interleaved, whether the cache is padded to full `max_position_embeddings = 262144` or allocated incrementally, and the exact runtime dtype — ALL of these are UNKNOWN (UK-004, UK-011). The shape is NOT a VERIFIED runtime observation.

Source: T4.1 RM-012 §3.77 (§390: "CONDITIONAL MODEL (shape derived from verified config; runtime allocation strategy = UNKNOWN)"), RM-013 §3.9 (§409: same classification), T4.3 AC-07 §3.7 (§267: "ACTIVATION BOUNDARY ... KV-cache state residency/allocation = UNKNOWN (UK-011, T4.4 domain)").

### 6.3 KV Cache State Properties — UNKNOWN Register

| Property | Status | Source / Reason |
|---|---|---|
| Runtime KV cache dtype | UNKNOWN (UK-004) | T4.1 RM-012 §3.79 (§385: "UNKNOWN at runtime (UK-004). Config declares BF16; mamba_ssm_dtype metadata does not confirm KV cache dtype"); T4.1 RM-013 §3.9 (§404: same); SET3 §9 UK-004 (§933: "Unknown: runtime state dtype behavior"); T4.3 §17 (§1281: UK-011 KV cache allocation = T4.4 domain) |
| KV cache allocation strategy (circular / paged / contiguous) | UNKNOWN (UK-011) | T4.1 RM-012 §3.79 (§391: "Exact KV cache allocation strategy = UNKNOWN (UK-011)"); T4.3 §19.1 (§1384: "T4.4-specific resolution needed: UK-011"); SET3 §9 UK-011 (§942) |
| Whether K and V are stored separately or interleaved in cache | UNKNOWN (UK-003) | T4.1 RM-012 §3.79 (§391: "Whether K and V stored separately or interleaved = UNKNOWN (UK-003)"); T4.3 AC-07 (§268: same) |
| Whether cache is circular buffer or paged allocation | UNKNOWN (UK-011) | T4.1 RM-012 §3.79 (§388: "Conditional — circular buffer or paged allocation MAY be used (UK-011)") |
| Whether KV cache is pre-allocated to max_position_embeddings = 262144 or grows per-token | UNKNOWN (UK-011) | SET0-03 §4 (§154: `max_position_embeddings = 262144`); T4.3 AC-30 §3.97 notes precomputation strategy is UNKNOWN; no runtime observation available |
| KV cache element layout (head-major vs seq-major vs blocked) | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| Whether KV cache is contiguous in memory or strided | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| Whether KV cache uses separate allocation per layer or a combined buffer | UNKNOWN | No runtime allocator observed; no evidence source establishes this |
| Whether KV cache is quantized (e.g., FP8 KV cache) | UNKNOWN | Config dtype = BF16; `use_cache = true` does not specify precision; no quantization config field observed; SET3-OC-3 §3.3 does not document cache quantization. The `mamba_ssm_dtype = float32` field is metadata only (VERIFIED to be non-tensor-dtype per SET3 §4.3 §521–524). |

### 6.4 KV Cache Per-Layer Scope

| Property | Value | Source | Classification |
|---|---|---|---|
| Full-attention layers that maintain KV cache | 16 (layers 3, 7, 11, ..., 63) | SET0-07 §4 (§183–§194: full-attention index set, count = 16); SET3 §2.3 (§119) | VERIFIED FACT |
| Per-layer KV cache state count | K cache + V cache = 2 state objects per layer | SET0-04 §3.3 (§201: "KV cache" representing "attention history"); T4.1 RM-012 (§375: "kv_cache.k"), RM-013 (§394: "kv_cache.v") | VERIFIED FACT (conceptual: K + V) |
| Total KV state objects (all full-attention layers) | 16 × 2 = 32 K+V cache regions | Arithmetic on verified counts | DERIVED FINDING (arithmetic) |
| Whether MTP layer (1 layer with self_attn) also maintains KV cache | UNKNOWN (UK-008 — MTP runtime execution) | SET0-08 §3 (§101: `mtp.layers.0.self_attn.q_proj.weight [12288, 5120]`, k_proj, v_proj exist as checkpoint tensors); SET3-OC-11 §2.6 (§171: `mtp_num_hidden_layers = 1`); SET0-08 §4 (§133: "MTP active runtime execution = UNKNOWN") | VERIFIED FACT (checkpoint tensors exist); UNKNOWN (MTP active runtime execution, UK-008) |

---

## 7. Raw Evidence — Dtype Declarations

### 7.1 Model Parameter Dtype

| Property | Value | Source | Classification |
|---|---|---|---|
| `text_config.dtype` | `"bfloat16"` | config.json: `text_config.dtype = "bfloat16"` | VERIFIED FACT | SET0-03 §4 (§149: `dtype: bfloat16`); SET3 §2.2 (§96); SET1-T1.6 §1 (`dtype = bfloat16`); SET1-T1.6 §3 (§516: "All 1,199 tensors: BF16") |
| Bytes per element (BF16) | 2 | Derived from BF16 | DERIVED FINDING | SET1-T1.6 §1 (§518: "Bytes per element: 2"); SET3-OC-3 §3.3 |
| `mamba_ssm_dtype` | `"float32"` | config.json: `text_config.mamba_ssm_dtype = "float32"` | VERIFIED FACT (metadata field only) | SET0-03 §4 (§388: `mamba_ssm_dtype = float32`); SET3 §4.3 (§519: "mamba_ssm_dtype: float32 (metadata field, not tensor dtype)"); SET3 §8 ASSUMPTION #4 (§921–924) |
| Runtime computation dtype | UNKNOWN (UK-004) | No runtime observation performed | UNKNOWN | T4.1 RM-012 §3.79 (§385: "UNKNOWN at runtime (UK-004)"); T4.3 §2 (§110: "Runtime computation dtype = UNKNOWN (UK-002, UK-004)") |
| Runtime state dtype (KV cache) | UNKNOWN (UK-004) | No runtime observation performed | UNKNOWN | T4.1 RM-012 §3.79 (§385); SET3 §9 UK-004 (§933: "Unknown: runtime state dtype behavior") |
| Whether `mamba_ssm_dtype = float32` implies runtime state in float32 | UNKNOWN (UK-002) | No runtime observation; metadata field only | UNKNOWN | SET3 §9 UK-002 (§933); T4.3 §2 (§110: "Runtime computation dtype = UNKNOWN (UK-002, UK-004)"); SET3 §4.3 (§523–524: "metadata configuration field, not a tensor storage dtype") |

### 7.2 Dtype Boundary — Checkpoint vs Runtime

```text
CHECKPOINT STORAGE TRUTH ≠ RUNTIME MEMORY TRUTH
```

- All 1,199 checkpoint tensors are BF16 (2 bytes/element) — VERIFIED FACT (SET1-T1.6 §3 §516–524).
- Whether the runtime KV cache is stored in BF16, FP32, FP8, or any other dtype is UNKNOWN (UK-004) — no runtime inference engine was executed or observed.
- The `mamba_ssm_dtype = float32` config field is a metadata configuration field, NOT a tensor storage dtype — VERIFIED FACT (SET3 §4.3 §519–524, SET3 §8 ASSUMPTION #4 §921–924). Whether it implies any runtime state behavior is UNKNOWN (UK-002).

---

## 8. Runtime-Specific Observations (Section C of Executor Contract)

### 8.1 Runtime Environment Inspection Results

| Observation | Result | Source / Method |
|---|---|---|
| Was a model loaded into a runtime (torch/transformers/JAX)? | NO — no model was instantiated or loaded | Repository inspection: no inference engine code exists in repository (terminal scan of .c/.cpp/.go/.js/.ts files); Python packages torch/transformers/safetensors are installed but no model weights were loaded |
| Was a KV-cache allocator observed? | NO — no allocator was running | No runtime process observed; no memory allocator code in repository |
| Was a live inference engine running? | NO | `ps`/process inspection; repository file scan |
| Was batch size (B) observed at runtime? | UNKNOWN | No runtime execution occurred |
| Was sequence length (S) observed at runtime? | UNKNOWN | No runtime execution occurred |
| Was KV cache memory allocation observed? | UNKNOWN | No runtime execution occurred; no allocator code present |
| Was KV cache quantization observed? | UNKNOWN | No runtime execution occurred; no evidence source documents cache quantization |
| Was allocator reuse behavior (circular/paged/contiguous) observed? | UNKNOWN | No runtime execution occurred; no allocator code present |
| Was implementation-specific cache layout observed? | UNKNOWN | No runtime execution occurred; no allocator code present |
| Was runtime dtype observed? | UNKNOWN | No runtime execution occurred |
| Was tensor parallelism (TP) / device distribution observed? | UNKNOWN | No runtime execution occurred; config does not declare TP degree |
| Was the `use_cache` flag's runtime effect observed? | UNKNOWN | No runtime execution occurred; `use_cache = true` is a config field only |

### 8.2 Runtime-Visible Configuration (from config.json only)

The only runtime-relevant configuration field available without executing a runtime:

| Field | Value | Source | Classification |
|---|---|---|---|
| `text_config.use_cache` | `true` | config.json | VERIFIED FACT (config field) |

All other runtime behaviors (whether `use_cache = true` causes actual KV caching to occur, whether it causes pre-allocation, whether the cache is paged, etc.) are UNKNOWN — the flag is not exercised at runtime in this environment.

### 8.3 No Runtime Observation Performed

```text
No actual runtime environment was available to observe:
  - actual KV cache allocation strategy
  - actual cache dtype at runtime
  - allocator reuse behavior (circular vs paged vs contiguous)
  - implementation-specific cache layout
  - runtime batch configuration
  - runtime sequence configuration
  - actual cache quantization
  - actual tensor parallelism / sharding at runtime
```

This is a VERIFIED FACT about the evidence-acquisition environment: no model was loaded, no inference was performed, and no runtime allocator was observed. The Executor does not infer implementation behavior from the absence of evidence.

---

## 9. UNKNOWN Register (Section D of Executor Contract)

Explicitly preserved unresolved questions relevant to the full-attention state model. These are NOT silently converted to assumptions.

### 9.1 SET3 UNKNOWNs Carried Forward (Relevant to T4.4)

| UK ID | Description | Evidence Domain | T4.4 Relevance |
|---|---|---|---|
| UK-001 | Exact linear-attention algorithm (Mamba/Mamba-2/GatedDeltaNet/DeltaNet) | Config declares presence, not identity | Not directly relevant to full-attention state (T4.5 domain); carried for completeness |
| UK-002 | Whether `mamba_ssm_dtype=float32` implies any runtime state in float32 | Metadata field only | Relevant: whether KV cache dtype could be float32 if metadata field influences it — UNKNOWN |
| UK-003 | Exact full-attention operator kernel implementation | Implementation-level, not checkpoint-level | Relevant: kernel determines cache layout, fusion, buffer reuse — UNKNOWN |
| UK-004 | Runtime computation dtype (general) | Not established | Directly relevant: KV cache dtype, intermediate dtype unknown |
| UK-005 | Exact normalization placement (pre/post attention, final norm) | Tensor names suggest placement but not verified | Relevant: affects coexistence of activations with state, not state shape itself — UNKNOWN |
| UK-006 | Exact residual connection structure (which residuals exist) | Not established by raw metadata | Relevant: affects activation coexistence, not state shape — UNKNOWN |
| UK-011 | Runtime KV cache allocation and paging strategy | Depends on runtime, not checkpoint | Directly relevant to T4.4: allocation strategy = UNKNOWN |

### 9.2 T4.4-Specific Unknowns (Not Resolvable from Available Evidence)

| Description | Status | Reason |
|---|---|---|
| Runtime KV cache dtype | UNKNOWN (UK-004) | No runtime engine executed; config only declares parameter dtype (BF16), not cache dtype |
| Runtime KV cache allocation strategy (contiguous / circular / paged) | UNKNOWN (UK-011) | No allocator observed; no runtime engine present |
| Whether K and V cache buffers are allocated separately or interleaved | UNKNOWN (UK-003) | Implementation detail; no runtime observation |
| Whether KV cache is pre-allocated to max_position_embeddings (262144) or grown per-token | UNKNOWN (UK-011) | Implementation detail; no runtime observation |
| KV cache element layout (head-major / seq-major / blocked) | UNKNOWN | No allocator observed |
| Whether KV cache is contiguous or strided in memory | UNKNOWN | No allocator observed |
| Whether KV cache uses per-layer allocation or a combined multi-layer buffer | UNKNOWN | No allocator observed |
| Whether KV cache is quantized (e.g., FP8) | UNKNOWN | No quantization config field observed; no runtime observation |
| Runtime batch size (B) | UNKNOWN (UK-009) | No runtime execution occurred |
| Runtime sequence length (S) | UNKNOWN (UK-009) | No runtime execution occurred |
| Runtime batch × layer count for cache | UNKNOWN | Both B and potentially dynamic layer count are runtime-dependent |
| Whether `use_cache = true` triggers actual caching at inference time | UNKNOWN | Flag is config only; not exercised at runtime |
| Whether the cache is shared across beam-search candidates | UNKNOWN | No runtime observation; no beam config field inspected for cache semantics |
| Whether KV cache is evicted/paged to disk under memory pressure | UNKNOWN | No runtime observation; no streaming/paging policy observed |
| Whether the full-attention KV cache and linear-attention recurrent state share an allocator | UNKNOWN | No runtime observation; different state types per SET0-04 §15.1 |

### 9.3 What Is NOT Unknown (Verifiably Established)

| Fact | Classification | Source |
|---|---|---|
| 16 full-attention layers exist (indices 3,7,...,63) | VERIFIED FACT | config.json layer_types; SET0-07 §4, §6 |
| 4 key/value heads per full-attention layer | VERIFIED FACT | config.json num_key_value_heads=4; SET0-04 §3.1 |
| 24 query heads per full-attention layer | VERIFIED FACT | config.json num_attention_heads=24; SET0-04 §3.1 |
| 256-dimensional heads (head_dim) | VERIFIED FACT | config.json head_dim=256; SET0-04 §3.1 |
| GQA grouping: 6 query heads per KV head | DERIVED FINDING | 24/4 arithmetic; SET0-04 §3.2, SET3-OC-3 §3.3 |
| K projection output = 1024 (4 × 256) | VERIFIED FACT | checkpoint k_proj.weight [1024, 5120]; SET0-08 §2 |
| V projection output = 1024 (4 × 256) | VERIFIED FACT | checkpoint v_proj.weight [1024, 5120]; SET0-08 §2 |
| Q projection output = 12288 (checkpoint) | VERIFIED FACT | checkpoint q_proj.weight [12288, 5120]; SET0-08 §2 |
| KV cache is the state mechanism for full-attention layers | VERIFIED FACT | SET0-04 §3.2, §3.3, §15.1; SET3-OC-3 §6.2, §3.3 |
| Rotary dimension = 64 (256 × 0.25) | DERIVED FINDING | head_dim × partial_rotary_factor; SET0-04 §4 |
| max_position_embeddings = 262144 | VERIFIED FACT | config.json; SET0-03 §4, SET3 §2.2 |
| use_cache = true | VERIFIED FACT | config.json; T4.1 RM-012, RM-013 |
| All checkpoint tensors are BF16 (2 bytes/element) | VERIFIED FACT | SET1-T1.6 §3 §516–524 |
| mamba_ssm_dtype = float32 is metadata only (not tensor dtype) | VERIFIED FACT | SET3 §4.3 §519–524, §8 ASSUMPTION #4 |
| 16 full-attention layers × 2 (K+V) = 32 KV state objects | DERIVED FINDING | Arithmetic on verified counts |

---

## 10. Raw Evidence (Section B of Executor Contract)

Consolidated raw observations with provenance, no interpretation beyond the observation itself.

### Observation 1: Full-attention layer count
```text
CLAIM: 16 full-attention layers out of 64 total language layers
SOURCE: config.json text_config.layer_types (64 entries; 16 × "full_attention")
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-04 §2 (§58); SET0-07 §2 (§80), §4 (§183–194); SET3 §2.3 (§119)
UNKNOWN/KNOWN: KNOWN
```

### Observation 2: Full-attention layer indices
```text
CLAIM: Full-attention layer indices are 3, 7, 11, 15, 19, 23, 27, 31, 35, 39, 43, 47, 51, 55, 59, 63
SOURCE: config.json text_config.layer_types array (explicit 64-entry sequence)
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-07 §4 (§183–194: "The complete full-attention index set is: 3,7,11,15,19,23,27,31,35,39,43,47,51,55,59,63"; §191: "Count: 16")
UNKNOWN/KNOWN: KNOWN
```

### Observation 3: Query head count
```text
CLAIM: 24 query heads per full-attention layer
SOURCE: config.json text_config.num_attention_heads = 24
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-04 §3.1 (§102); SET0-03 §6 (§266); SET3-OC-3 §3.3 (§228); T4.1 RM-002 (§160)
UNKNOWN/KNOWN: KNOWN
```

### Observation 4: KV head count
```text
CLAIM: 4 key/value heads per full-attention layer
SOURCE: config.json text_config.num_key_value_heads = 4
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-04 §3.1 (§104); SET0-03 §6 (§269); SET3-OC-3 §3.3 (§229); T4.1 RM-002 (§160), RM-012 (§384), RM-013 (§403)
UNKNOWN/KNOWN: KNOWN
```

### Observation 5: Head dimension
```text
CLAIM: 256-dimensional attention heads (head_dim = 256)
SOURCE: config.json text_config.head_dim = 256
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-04 §3.1 (§108); SET0-03 §6 (§272); SET3-OC-3 §3.3 (§230); T4.1 RM-002 (§160), RM-012 (§384), RM-013 (§403)
UNKNOWN/KNOWN: KNOWN
```

### Observation 6: GQA ratio (query heads per KV head)
```text
CLAIM: Each KV head is shared by 6 query heads (24 / 4 = 6)
SOURCE: Arithmetic on config.json num_attention_heads (24) and num_key_value_heads (4)
SOURCE TYPE: Derived arithmetic on raw config fields
CLASSIFICATION: DERIVED FINDING
PROVENANCE: SET0-04 §3.2 (§138: "24 / 4 = 6"); SET0-03 §6 (§288: "24 / 4 = 6"); SET3-OC-3 §3.3 (§272)
UNKNOWN/KNOWN: KNOWN
```

### Observation 7: K projection weight tensor
```text
CLAIM: self_attn.k_proj.weight has shape [1024, 5120], present in 16/16 full-attention layers
SOURCE: Raw checkpoint tensor metadata (safetensors headers)
SOURCE TYPE: Checkpoint tensor metadata (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-08 §2 (§55: "self_attn.k_proj.weight | [1024, 5120] | 16/16 | VERIFIED FACT"); SET3-OC-3 §3.3 (§247); T4.1 RM-002 (§154)
UNKNOWN/KNOWN: KNOWN
```

### Observation 8: V projection weight tensor
```text
CLAIM: self_attn.v_proj.weight has shape [1024, 5120], present in 16/16 full-attention layers
SOURCE: Raw checkpoint tensor metadata (safetensors headers)
SOURCE TYPE: Checkpoint tensor metadata (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-08 §2 (§56: "self_attn.v_proj.weight | [1024, 5120] | 16/16 | VERIFIED FACT"); SET3-OC-3 §3.3 (§248); T4.1 RM-002 (§154)
UNKNOWN/KNOWN: KNOWN
```

### Observation 9: Q projection weight tensor
```text
CLAIM: self_attn.q_proj.weight has shape [12288, 5120], present in 16/16 full-attention layers. Logical Q dim = 24×256 = 6144, but checkpoint output = 12288 (includes QK or gate dimension from GQA implementation).
SOURCE: Raw checkpoint tensor metadata (safetensors headers) + config.json arithmetic
SOURCE TYPE: Checkpoint tensor metadata (raw) + config arithmetic
CLASSIFICATION: VERIFIED FACT (tensor shape); VERIFIED FACT (shape mismatch noted by SET3)
PROVENANCE: SET0-08 §2 (§54: "self_attn.q_proj.weight | [12288, 5120] | 16/16 | VERIFIED FACT"); SET3-OC-3 §3.3 (§256–259: "24 × 256 = 6144 → but q_proj output is 12288 (includes QK or gate dimension...)"); T4.1 RM-002 (§154), RM-014 (§421)
UNKNOWN/KNOWN: KNOWN (shape is known; exact decomposition of 12288 is UNKNOWN per UK-003)
```

### Observation 10: Q/K normalization weight tensors
```text
CLAIM: self_attn.q_norm.weight [256] and self_attn.k_norm.weight [256] present in 16/16 layers
SOURCE: Raw checkpoint tensor metadata (safetensors headers)
SOURCE TYPE: Checkpoint tensor metadata (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-08 §2 (§58–59); SET3-OC-3 §3.3 (§250–251); T4.1 RM-002 (§154), RM-015 (§438)
UNKNOWN/KNOWN: KNOWN
```

### Observation 11: O projection weight tensor
```text
CLAIM: self_attn.o_proj.weight has shape [5120, 6144], present in 16/16 full-attention layers
SOURCE: Raw checkpoint tensor metadata (saffets headers)
SOURCE TYPE: Checkpoint tensor metadata (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-08 §2 (§57: "self_attn.o_proj.weight | [5120, 6144] | 16/16 | VERIFIED FACT"); SET3-OC-3 §3.3 (§249); T4.1 RM-002 (§154), RM-012 (§494)
UNKNOWN/KNOWN: KNOWN
```

### Observation 12: Full-attention uses KV cache (state mechanism)
```text
CLAIM: Full-attention layers use a conventional attention history represented by a KV cache
SOURCE: SET0-04 §3.3 (§201: "full-attention layers use a conventional attention history represented by a KV cache") and §3.2 execution characteristics (§208: "KV cache")
SOURCE TYPE: Document evidence (SET0 structural analysis of official implementation)
CLASSIFICATION: VERIFIED FACT (state mechanism is KV cache, not recurrent state)
PROVENANCE: SET0-04 §3.2 (§177–212: "Verified implementation-level findings" include "KV cache" at §208); SET0-04 §3.3 (§200–201); SET0-04 §15.1 (§619–629: "16 Full-Attention layers → KV cache"); SET3-OC-3 §3.3 §6.2 (§264–268: dataflow includes "KV Cache" stage)
UNKNOWN/KNOWN: KNOWN (mechanism is KV cache; allocation/strategy is UNKNOWN)
```

### Observation 13: Full-attention execution stages
```text
CLAIM: Full-attention execution stages are: Hidden States → Q/KV projection → Q/K normalization → RoPE → KV Cache → Causal Attention → Output Gating → Output Projection
SOURCE: SET0-04 §3.3 (§182–198) and SET3-OC-3 §3.3 §6.2 (§264–268)
SOURCE TYPE: Document evidence (implementation-derived stage ordering)
CLASSIFICATION: DERIVED FINDING (official implementation describes stage ordering, not checkpoint metadata alone)
PROVENANCE: SET0-04 §3.3 (§182–198: "Hidden States → Q/KV projection → Q/K normalization → Rotary Position Encoding → KV Cache → Causal Attention → Output Gating → Output Projection"); SET3-OC-3 §3.3 §6.2 (§266–268: same ordering; §269: "Classification: DERIVED FINDING — the stage ordering is described by the official implementation")
UNKNOWN/KNOWN: KNOWN (stage ordering); exact kernel fusion = UNKNOWN (UK-003)
```

### Observation 14: use_cache configuration field
```text
CLAIM: text_config.use_cache = true (configuration field present)
SOURCE: config.json (raw checkpoint metadata)
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT (config field existence and value)
PROVENANCE: config.json text_config.use_cache = true (line 116); T4.1 RM-012 (§384), RM-013 (§403)
UNKNOWN/KNOWN: KNOWN (field exists and = true); runtime effect = UNKNOWN
```

### Observation 15: Model dtype declaration
```text
CLAIM: text_config.dtype = "bfloat16" (all checkpoint tensors BF16, 2 bytes/element)
SOURCE: config.json text_config.dtype (raw) + SET1 raw tensor metadata verification
SOURCE TYPE: Checkpoint config artifact (raw) + checkpoint tensor metadata (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: config.json (line 13: "dtype": "bfloat16"); SET0-03 §4 (§149); SET3 §2.2 (§96); SET1-T1.6 §3 (§516–518: "All 1,199 tensors: BF16; Bytes per element: 2")
UNKNOWN/KNOWN: KNOWN (checkpoint dtype = BF16); KNOWN (mamba_ssm_dtype is metadata only); UNKNOWN (runtime cache dtype)
```

### Observation 16: Rotary position embedding dimension
```text
CLAIM: Rotary dimension = 256 × 0.25 = 64
SOURCE: config.json head_dim (256) × partial_rotary_factor (0.25)
SOURCE TYPE: Arithmetic on raw config fields
CLASSIFICATION: DERIVED FINDING
PROVENANCE: SET0-04 §4 (§247: "256 × 0.25 = 64"); SET3-OC-3 §3.3 (§275); T4.1 RM-016 (§458)
UNKNOWN/KNOWN: KNOWN
```

### Observation 17: max_position_embeddings
```text
CLAIM: max_position_embeddings = 262144
SOURCE: config.json text_config.max_position_embeddings = 262144
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-03 §4 (§154); SET3 §2.2 (§97); T4.3 AC-30 (§697)
UNKNOWN/KNOWN: KNOWN
```

### Observation 18: Full-attention tensor count
```text
CLAIM: 6 tensors per full-attention layer × 16 layers = 96 full-attention tensors
SOURCE: Raw checkpoint tensor metadata; config.json layer_types count
SOURCE TYPE: Checkpoint tensor metadata (raw) + config
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-08 §2 (§54–59: 6 tensors, 16/16 coverage); SET3 §5.2 (§582: "Full-attention layers: 96 tensors"); T4.1 RM-002 (§155); T4.2 §3.2 (§198: "6 tensors × 16 layers = 96 tensors (16/16 coverage)")
UNKNOWN/KNOWN: KNOWN
```

### Observation 19: Linear-attention state model is distinct from KV cache
```text
CLAIM: Full-attention KV cache and linear-attention recurrent/convolution state are distinct state models
SOURCE: SET0-04 §15.1 (§619–629: "16 Full-Attention layers → KV cache; 48 Gated DeltaNet layers → recurrent state + convolution state"); SET0-04 §9 (§405–429); SET0-04 §11 (§459–477 comparison table)
SOURCE TYPE: Document evidence (SET0 structural analysis)
CLASSIFICATION: VERIFIED FACT
PROVENANCE: SET0-04 §15.1 (§619: "The language model does not have one universal attention-state mechanism"); §15.3 (§653–668: "The attention-state memory model must distinguish: KV Cache from: Recurrent State + Convolution State"); SET3-OC-3 §3.4 (§329–336)
UNKNOWN/KNOWN: KNOWN (models are distinct; exact linear-attention algorithm = UNKNOWN UK-001)
```

### Observation 20: Attention output gating
```text
CLAIM: attn_output_gate = true; output_gate_type = swish (config fields)
SOURCE: config.json text_config.attn_output_gate = true; text_config.output_gate_type = "swish"
SOURCE TYPE: Checkpoint config artifact (raw)
CLASSIFICATION: VERIFIED FACT (config fields)
PROVENANCE: SET0-04 §5 (§261–276); SET3-OC-3 §3.3 (§233); T4.1 RM-002 (§163), RM-017 (§476)
UNKNOWN/KNOWN: KNOWN (config fields); UNKNOWN (exact gate tensor existence/location — UK-006)
```

### Observation 21: Environment — no runtime present
```text
CLAIM: No inference engine, loaded model, or running KV-cache allocator exists in the assigned environment
SOURCE: Repository file scan (.c/.cpp/.cc/.go/.js/.ts = none); Python environment inspection (torch/transformers installed but no model loaded); process inspection
SOURCE TYPE: Direct environment observation
CLASSIFICATION: VERIFIED FACT
PROVENANCE: This evidence-acquisition task; terminal inspection commands
UNKNOWN/KNOWN: KNOWN (no runtime present); UNKNOWN (all runtime KV-cache behaviors)
```

---

## 11. T4.3 Downstream Dependency Mapping (Consumed)

T4.3 §19.1 (§1377–1385) explicitly defines what T4.4 consumes:

| T4.3 Item | T4.3 Description | T4.3 Classification |
|---|---|---|
| AC-07 (`kv_store_boundary[L]`) | Activation-side boundary observation: K/V projection outputs (AC-05) feed a downstream KV-cache state transition. Shape `[B, 4, S, 256]` is NOT established in T4.3. | ACTIVATION BOUNDARY (state: T4.4 UNKNOWN) |
| Provenance | T4.1 RM-012 (K cache), RM-013 (V cache); SET0-04 §3.2; SET3-OC-3 §3.3 | VERIFIED FACT (provenance) |
| Parameterization inherited | `B` (UK-009), `S` (UK-009), state dtype `E_state` (UK-004) | UNKNOWN |
| T4.4-specific resolution needed | UK-011 (KV cache allocation and paging strategy) | UNKNOWN → T4.4 domain |

Source: T4.3 §19.1 (§1377–1385). Classification: VERIFIED FACT (T4.3 boundary mapping).

---

## 12. Raw Evidence Summary Table

### Consolidated Structural Facts (VERIFIED / DERIVED)

| # | Claim | Classification | Provenance |
|---|---|---|---|
| 1 | 64 total language layers | VERIFIED FACT | config.json; SET0-03 §4, SET0-07 §2 |
| 2 | 16 full-attention layers | VERIFIED FACT | config.json layer_types; SET0-04 §2, SET0-07 §2, SET0-07 §4 |
| 3 | 48 linear-attention layers | VERIFIED FACT (carried, not T4.4 focus) | config.json layer_types; SET0-04 §2, SET0-07 §2 |
| 4 | Full-attention indices: 3,7,11,...,63 | VERIFIED FACT | SET0-07 §4 |
| 5 | 24 query heads | VERIFIED FACT | config.json num_attention_heads=24; SET0-04 §3.1 |
| 6 | 4 KV heads | VERIFIED FACT | config.json num_key_value_heads=4; SET0-04 §3.1 |
| 7 | head_dim = 256 | VERIFIED FACT | config.json head_dim=256; SET0-04 §3.1 |
| 8 | GQA ratio = 6 (24/4) | DERIVED FINDING | Arithmetic; SET0-04 §3.2, SET3-OC-3 §3.3 |
| 9 | K proj output = 1024 (4×256) | VERIFIED FACT | checkpoint k_proj [1024,5120]; SET0-08 §2 |
| 10 | V proj output = 1024 (4×256) | VERIFIED FACT | checkpoint v_proj [1024,5120]; SET0-08 §2 |
| 11 | Q proj output = 12288 (checkpoint) | VERIFIED FACT | checkpoint q_proj [12288,5120]; SET0-08 §2 |
| 12 | Q logical = 6144, checkpoint = 12288 | VERIFIED FACT | SET3-OC-3 §3.3 (§256–259) |
| 13 | 6 tensors/layer × 16 layers = 96 FA tensors | VERIFIED FACT | SET0-08 §2; SET3 §5.2; T4.1 RM-002 |
| 14 | Full-attention uses KV cache (not recurrent state) | VERIFIED FACT | SET0-04 §3.2, §3.3, §15.1; SET3-OC-3 §3.3, §6.2 |
| 15 | Execution stages incl. "KV Cache" stage | DERIVED FINDING | SET0-04 §3.3 §182–198; SET3-OC-3 §3.3 §6.2 |
| 16 | use_cache = true | VERIFIED FACT | config.json; T4.1 RM-012, RM-013 |
| 17 | dtype = bfloat16 (all 1,199 tensors) | VERIFIED FACT | config.json; SET0-03 §4; SET1-T1.6 §3 |
| 18 | mamba_ssm_dtype = float32 is metadata only | VERIFIED FACT | SET3 §4.3 §519–524, §8 ASSUMPTION #4 |
| 19 | rotary_dim = 64 (256 × 0.25) | DERIVED FINDING | SET0-04 §4, SET3-OC-3 §3.3 |
| 20 | max_position_embeddings = 262144 | VERIFIED FACT | config.json; SET0-03 §4 |
| 21 | attn_output_gate = true, output_gate_type = swish | VERIFIED FACT | config.json; SET0-04 §5, SET3-OC-3 §3.3 |
| 22 | Full-attention state = KV cache (distinct from linear-attn state) | VERIFIED FACT | SET0-04 §15.1, §9, §11 |
| 23 | No runtime/inference engine present in environment | VERIFIED FACT | Environment inspection (this task) |

### Consolidated Unknown / Conditional Evidence

| # | Claim | Classification | Provenance |
|---|---|---|---|
| K1 | Runtime KV cache dtype | UNKNOWN (UK-004) | No runtime observed; config only declares parameter dtype |
| K2 | Runtime KV cache allocation strategy (contiguous/circular/paged) | UNKNOWN (UK-011) | No allocator observed |
| K3 | Whether K and V cache are separate or interleaved | UNKNOWN (UK-003) | Implementation detail |
| K4 | Whether cache pre-allocated to 262144 or grows per-token | UNKNOWN (UK-011) | Implementation detail |
| K5 | KV cache element layout (head-major/seq-major/blocked) | UNKNOWN | No allocator observed |
| K6 | Whether KV cache contiguous or strided | UNKNOWN | No allocator observed |
| K7 | Per-layer vs combined multi-layer cache buffer | UNKNOWN | No allocator observed |
| K8 | KV cache quantization (e.g., FP8) | UNKNOWN | No quantization config; no runtime observation |
| K9 | Runtime batch size (B) | UNKNOWN (UK-009) | No runtime execution |
| K10 | Runtime sequence length (S) | UNKNOWN (UK-009) | No runtime execution |
| K11 | use_cache flag runtime effect | UNKNOWN | Config field only; not exercised |
| K12 | Cache sharing across beam candidates | UNKNOWN | No runtime observation |
| K13 | KV cache eviction/paging to disk | UNKNOWN | No runtime observation |
| K14 | Full-attention cache × linear-attention state shared allocator | UNKNOWN | No runtime observation |
| K15 | Exact Q projection output dimension decomposition (12288) | UNKNOWN (UK-003) | SET3 explicitly: "includes QK or gate dimension" |
| K16 | Exact full-attention kernel implementation | UNKNOWN (UK-003) | Implementation-level, not checkpoint-level |
| K17 | Exact attention softmax scaling factor | UNKNOWN (UK-010) | Not in config or checkpoint |
| K18 | Exact normalization placement (pre/post-norm) | UNKNOWN (UK-005) | Tensor names suggest, not verified |
| K19 | Exact residual connection structure | UNKNOWN (UK-006) | Not established by raw metadata |
| K20 | Whether gate tensor exists as separate checkpoint tensor | UNKNOWN (UK-006) | SET3-OC-9 §3.9: "UNKNOWN: exact tensor location and formulation of the gate" |

### Conditional / Parameterized Fact (Config-Derived State Shape)

| # | Claim | Classification | Provenance |
|---|---|---|---|
| C1 | Config-derived KV cache K shape = `[B, 4, S, 256]` | CONDITIONAL MODEL | T4.1 RM-012 §3.77 (§390); SET0-04 §3.2; SET3-OC-3 §3.3 |
| C2 | Config-derived KV cache V shape = `[B, 4, S, 256]` | CONDITIONAL MODEL | T4.1 RM-013 §3.9 (§409); SET0-04 §3.2; SET3-OC-3 §3.3 |
| C3 | Config-derived KV cache element dtype | UNKNOWN (parameterized) | T4.1 RM-012 §3.79 (§385: "UNKNOWN at runtime (UK-004)") |

---

## 13. Result (Section C of Executor Contract)

### What the evidence establishes

The evidence establishes the following **structural constants** (all VERIFIED FACT or DERIVED FINDING from the pinned checkpoint config.json and verified tensor metadata):

1. **16 full-attention layers** at indices 3, 7, 11, ..., 63 (VERIFIED FACT).
2. **4 KV heads** per full-attention layer (VERIFIED FACT — `num_key_value_heads = 4`).
3. **24 query heads** per full-attention layer (VERIFIED FACT — `num_attention_heads = 24`).
4. **Head dimension = 256** (VERIFIED FACT — `head_dim = 256`).
5. **GQA grouping ratio = 6** (24/4, DERIVED FINDING).
6. **K projection output dimension = 1024** (4 × 256, VERIFIED FACT — matches `k_proj.weight [1024, 5120]`).
7. **V projection output dimension = 1024** (4 × 256, VERIFIED FACT — matches `v_proj.weight [1024, 5120]`).
8. **Q projection output dimension = 12288** (VERIFIED FACT — `q_proj.weight [12288, 5120]`; logical dim would be 6144 = 24 × 256; the 2× discrepancy is explicitly noted as including a QK/gate dimension per SET3, exact decomposition UNKNOWN).
9. **6 full-attention weight tensors per layer**: q_proj, k_proj, v_proj, o_proj, q_norm, k_norm (VERIFIED FACT — 96 tensors total, 16/16 coverage).
10. **Full-attention layers use a KV cache** as their state mechanism — VERIFIED FACT (not recurrent/conv state; SET0-04 §15.1).
11. **Execution stages include a "KV Cache" store stage** (DERIVED FINDING — stage ordering from official implementation).
12. **dtype = BF16** for all checkpoint tensors (VERIFIED FACT — 2 bytes/element).
13. **`mamba_ssm_dtype = float32`** is metadata only, NOT tensor storage dtype (VERIFIED FACT).
14. **`use_cache = true`** config field is present (VERIFIED FACT — runtime effect UNKNOWN).
15. **`max_position_embeddings = 262144`** (VERIFIED FACT).
16. **Rotary dimension = 64** (256 × 0.25, DERIVED FINDING).
17. **State mechanism distinction**: full-attention → KV cache; linear-attention → recurrent + conv state (VERIFIED FACT).

### What the evidence does NOT establish (all UNKNOWN)

The evidence does NOT establish any of the following runtime properties, because no runtime engine was executed or observed in the assigned environment:

- Runtime KV cache dtype (UK-004)
- Runtime KV cache allocation strategy (UK-011) — contiguous, circular, or paged
- Whether K and V caches are stored separately or interleaved (UK-003)
- Whether the cache is pre-allocated to max_position_embeddings or grown per-token
- KV cache element layout (head-major, seq-major, blocked, etc.)
- Whether KV cache is contiguous or strided
- Per-layer vs combined multi-layer cache buffer allocation
- KV cache quantization (e.g., FP8)
- Runtime batch size (B) (UK-009)
- Runtime sequence length (S) (UK-009)
- Whether `use_cache = true` actually triggers caching at runtime
- Cache sharing across beam-search candidates
- KV cache eviction/paging to disk under memory pressure

### What this means (NON-AUTHORITATIVE — ORCHESTRATOR INTERPRETATION REQUIRED)

The evidence establishes the **structural constants** that bound the full-attention KV-cache memory model. These constants are sufficient for the ORCHESTRATOR to construct a parameterized formula:

```text
The config-derived (conditional) KV cache K/V shape per layer is:
  K cache: [B, 4, S, 256]
  V cache: [B, 4, S, 256]
```

However, the actual runtime memory of the full-attention KV-cache state depends on **multiple UNKNOWN parameters** that cannot be resolved from repository/config evidence alone:

```text
KV cache memory = f(
    num_full_attention_layers = 16           [VERIFIED FACT — establishes multiplier]
    num_kv_heads = 4                         [VERIFIED FACT — establishes per-layer factor]
    head_dim = 256                             [VERIFIED FACT — establishes per-element factor]
    B (batch)                                [UNKNOWN — UK-009]
    S (sequence length)                      [UNKNOWN — UK-009]
    E_state (state dtype bytes/element)      [UNKNOWN — UK-004]
    allocation_strategy                        [UNKNOWN — UK-011]
    cache_layout                              [UNKNOWN — UK-003]
    preallocation_behavior                      [UNKNOWN — UK-011]
    quantization                              [UNKNOWN]
)
```

The structural constants (layer count, KV head count, head dimension, GQA ratio) are KNOWN. All runtime execution variables are UNKNOWN. The Executor has NOT constructed, derived, or calculated any final KV-cache memory equation.

---

## 14. Do-Not-Run Compliance (Section B of Executor Contract)

The following actions were NOT performed during T4.4 evidence acquisition:

- ✗ No final full-attention memory formula designed
- ✗ No final KV-cache memory equation derived
- ✗ No peak full-attention memory requirement calculated
- ✗ No T4.4 technical PASS declared
- ✗ No T4.4 complete declared
- ✗ No T4.4 readiness for T4.5 declared
- ✗ No T4.5 authorized
- ✗ No T4.5 started
- ✗ No T4.6 started
- ✗ No T4.7 started
- ✗ No SET5 started
- ✗ No SET2 reopened
- ✗ No SET2-T2.7-R2 created
- ✗ No SET2 revision created
- ✗ No modification to T4.1, T4.2, or T4.3 technical evidence
- ✗ No reinterpretation of historical SET2 authorization record
- ✗ No task selection or authorization by Executor

---

## 15. Persistence

This evidence document is persisted at:
```text
docs/set-4/04-full-attention-state-model.md
```

Following the existing SET4 evidence file structure defined in ROADMAP.md §2203–2216:
```text
docs/set-4/
├── 01-runtime-memory-inventory.md
├── 02-weight-residency-model.md
├── 03-activation-lifetime-model.md
├── 04-full-attention-state-model.md   ← THIS FILE (T4.4 evidence acquisition)
├── 05-linear-attention-state-model.md   (T4.5 — not started)
├── 06-workspace-buffer-model.md          (T4.6 — not started)
├── 07-peak-runtime-memory-model.md       (T4.7 — not started)
├── 08-memory-constraint-reconciliation.md (T4.8 — not started)
└── 09-set4-boundary-completeness-audit.md (T4.9 — not started)
```

---

## 16. Evidence Classification Legend

Used throughout this document:

```text
VERIFIED FACT:
  Directly supported by raw checkpoint config artifact (config.json),
  raw checkpoint tensor metadata (safetensors headers / manifest.json),
  or established SET3 operator/dataflow evidence.

DOCUMENTED CAPABILITY:
  Sourced from authoritative external documentation; not promoted to VERIFIED FACT.
  (Not used in this evidence document — all evidence is raw observation or derived
  from verified sources.)

DERIVED FINDING:
  Arithmetic or logical combination of VERIFIED FACT evidence. Explicitly labeled.

CONDITIONAL MODEL:
  A memory property expressed as an explicit dependency on an unresolved runtime
  behavior. Bounded, parameterized, not an assumption.

UNKNOWN:
  Runtime behavior or structural detail not established by available evidence.
  Treated as a boundary, not a gap to fill.
```

---

## 17. Acceptance Criteria Mapping (Executor Evidence)

| Criterion | Status | Evidence Location |
|---|---|---|
| T4.4 task scope followed exactly | MET (Executor scope only) | §1.4, §14 |
| Full-attention structural evidence collected | MET | §3, §4, §5, §6, §12, §10 |
| Attention-layer count established or marked UNKNOWN | MET — 16 full-attention layers (VERIFIED FACT) | §3.1, Observation T1, §12 row 2 |
| KV-head count established or marked UNKNOWN | MET — 4 KV heads (VERIFIED FACT) | §4.1, Observation T4, §12 row 6 |
| Query-head count established or marked UNKNOWN | MET — 24 query heads (VERIFIED FACT) | §4.1, Observation T3, §12 row 5 |
| Head dimension established or marked UNKNOWN | MET — 256 (VERIFIED FACT) | §4.1, Observation T5, §12 row 7 |
| Relevant K/V projection evidence collected | MET | §5, Observations T7–T10 |
| Relevant dtype evidence collected | MET | §7 |
| Runtime-specific observations collected where actually available | MET — none available, documented | §8 |
| Provenance captured | MET | Every observation includes Source/Source Type/Classification/Provenance | §10 |
| Unknowns explicitly preserved | MET | §9, §12 (Unknown/Conditional table) |
| No unsupported technical conclusion promoted to fact | MET | §13 marks interpretation as NON-AUTHORITATIVE; §14 Do-Not-Run |
| No final T4.4 mathematical model independently invented | MET | §14; §13 explicitly states Executor has NOT constructed the formula |
| No T4.5 work started | MET | §14 |
| No SET5 work started | MET | §14 |
| No SET2 revision created | MET | §14 |
| Evidence persisted according to repository contract | MET | §15 (this document) |
| Remote persistence verified | PENDING — Executor to commit + push + verify | §16 (below) |
