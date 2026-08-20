# SET 4 — Activation Lifetime Model

## Document Status

- Document: `docs/set-4/03-activation-lifetime-model.md`
- SET: `SET 4 — Runtime Memory Model`
- Source Task: `SET4-T4.3`
- Status: COMPLETE (T4.3 PASS)
- Responsibility: 🧠 LUNA
- Date: 2026-08-20
- Control State: `SET4-READINESS-GATE = PASS`, `SET4-T4.1 = PASS`, `SET4-T4.2 = PASS`
- Dependency: `SET4-T4.2 PASS`

---

## 1. Source and Provenance

### 1.1 Authoritative Model

- Model: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

### 1.2 Primary Evidence Sources (Accepted Upstream Input)

All assertions in this document are traced to accepted upstream evidence:

```text
SET 4 (direct input):
  docs/set-4/01-runtime-memory-inventory.md        — T4.1 full activation inventory (RM-001..RM-047)
  docs/set-4/02-weight-residency-model.md         — T4.2 weight residency model (accepted PASS)

SET 3 (operator/computation model — accepted input):
  docs/set-3/01-operator-computation-model.md     — OC-1..OC-11, dataflow §§6.1–6.7, unknowns §9

SET 1 (tensor / byte truth):
  docs/set-1/02-parameter-reconstruction.md       — 27,781,427,952 parameters
  docs/set-1/03-tensor-byte-accounting.md         — 55,562,855,904 logical bytes

SET 0 (structural truth):
  docs/set-0/04-attention-architecture.md         — attention families, config, dataflow
  docs/set-0/05-mlp-architecture.md               — MLP structure, dimensions, operator model
  docs/set-0/06-vision-and-mtp.md                 — vision + MTP configuration
  docs/set-0/07-layer-topology.md                 — 64-layer topology, layer-by-layer
  docs/set-0/08-tensor-shape-mapping.md           — verified tensor shapes, 15 MTP tensors
  docs/set-0/03-core-architecture.md              — config fields (hidden_size, vocab_size, etc.)

ROADMAP.md — SET4 control state, atomic task plan
```

### 1.3 Classification Schema

Every material assertion in this document is classified as exactly one of:

- VERIFIED FACT — directly supported by raw SET1 tensor metadata, SET0 configuration fields, or established SET3 operator/dataflow evidence.
- DOCUMENTED CAPABILITY — sourced from authoritative external documentation; not promoted to VERIFIED FACT.
- DERIVED FINDING — arithmetic or logical combination of verified evidence. Explicitly labeled.
- CONDITIONAL MODEL — a memory property expressed as an explicit dependency on an unresolved runtime behavior. Bounded, parameterized, not an assumption.
- UNKNOWN — runtime behavior or structural detail not established by available evidence. Treated as a boundary.

### 1.4 Critical Distinctions

```text
CHECKPOINT STORAGE TRUTH ≠ LOGICAL WEIGHT BYTES ≠ RUNTIME-RESIDENT WEIGHT MEMORY
STRUCTURAL TRUTH ≠ RUNTIME IMPLEMENTATION TRUTH
ACTIVATION MEMORY ≠ WEIGHT MEMORY ≠ STATE MEMORY ≠ WORKSPACE MEMORY
```

This document establishes the ACTIVATION MEMORY model only. Activation memory is the per-token, per-layer runtime memory for forward-pass intermediate tensors. It is kept strictly separate from:

- Weight memory (T4.2 — accepted input, not modeled here)
- Full-attention KV-cache state (T4.4 — downstream, not modeled here beyond the activation boundary)
- Linear-attention recurrent/conv state (T4.5 — downstream, not modeled here beyond the activation boundary)
- Workspace/temporary execution buffers (T4.6 — downstream, not modeled here)
- Final peak runtime memory (T4.7 — downstream, not produced here)

---

## 2. Model Shape Constants (VERIFIED FACT)

These are the structural constants from which activation shapes are derived. Runtime batch size B and sequence length S remain UNKNOWN (UK-009).

```text
hidden_size = 5120                          (SET0 §4, SET3 §2.2)
intermediate_size = 17408                     (SET0 §4, SET3-OC-5 §3.5)
vocab_size = 248320                           (SET0 §4, SET3 §2.2)
num_hidden_layers = 64                        (SET0 §4, SET3 §2.2)
full_attention_layers = 16                    (SET0-07 §4, SET3 §2.3)
linear_attention_layers = 48                  (SET0-07 §5, SET3 §2.3)

Full-attention (GQA):
  num_attention_heads = 24                    (SET0 §4, SET3-OC-3 §3.3)
  num_key_value_heads = 4                     (SET0 §4, SET3-OC-3 §3.3)
  head_dim_full = 256                         (SET0 §4, SET3-OC-3 §3.3)
  q_proj_out = 12288 = 24 × 256               (SET0-08 §2, SET3 §3.3)

Linear-attention:
  linear_num_key_heads = 16                   (SET0 §4, SET3-OC-4 §3.4)
  linear_num_value_heads = 48                 (SET0 §4, SET3-OC-4 §3.4)
  linear_head_dim = 128                       (SET0 §4, SET3-OC-4 §3.4)
  in_proj_qkv_out = 10240 = 16×128 + 16×128 + 48×128  (SET0-08 §2, SET3 §3.4)
  in_proj_z_out = 6144 = 48 × 128             (SET0-08 §2, SET3 §3.4)

RoPE:
  partial_rotary_factor = 0.25                (SET0 §4, SET3-OC-3 §3.3)
  head_dim_full = 256
  rotary_dim = 64 = 256 × 0.25                (SET0-04 §4, SET3-OC-3 §3.3)

Dtype:
  bytes_per_element(BF16) = 2                 (SET1-T1.6 §1, SET0 §4)
  Runtime computation dtype = UNKNOWN         (UK-002, UK-004)

Runtime dimensions:
  B (batch) = UNKNOWN                         (UK-009)
  S (sequence length) = UNKNOWN               (UK-009)
```

### 2.1 Shape Parameterization Convention

All activation shapes are parameterized as functions of B (batch) and S (sequence length) where runtime values are UNKNOWN (UK-009). The formulas use:

- `H = hidden_size = 5120`
- `I = intermediate_size = 17408`
- `V = vocab_size = 248320`
- `E = bytes_per_element` (2 for BF16, parameterized)

---

## 3. Activation Inventory

Each activation object records: identity, originating operator, consuming operator, shape, dtype, element count, byte size, lifetime, reuse relationship, persistence category, provenance, and classification.

### 3.1 AC-01: Token Embedding Lookup Output

| Field | Value |
|---|---|
| Identity | `embeddings` (runtime output of OC-1) |
| Originating operator | OC-1 (LanguageEmbedding) |
| Consuming operator | Layer 0 token mixer (OC-4) |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From OC-1 output until consumed by first token mixer; then reusable |
| Reuse relationship | Potentially reusable as Layer 0 input buffer if fused |
| Persistence category | TRANSIENT (per forward pass) |
| Provenance | SET3-OC-1 §3.1; SET0-08 §2; SET3 §6.1; T4.1 §3.2 RM-010 |
| Classification | VERIFIED FACT (shape structure, operator relationship) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether materialized separately or fused = UNKNOWN |

### 3.2 AC-02: Decoder-Layer Input (Hidden States, Per-Layer)

| Field | Value |
|---|---|
| Identity | `layer_input[L]` — hidden states entering layer L |
| Originating operator | Previous layer's residual output (or embedding for L=0) |
| Consuming operator | RMSNorm (OC-6) at layer L |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From previous layer output through RMSNorm consumption; then reusable or released |
| Reuse relationship | May overwrite previous layer's output buffer if in-place residual is used |
| Persistence category | ACTIVATION (per-layer, per-forward-pass) |
| Provenance | SET3 §6.1 (decoder-layer dataflow); SET0-05 §13 (MLP placement); T4.1 §3.5 RM-023, §3.5 RM-024 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); exact residual topology = UNKNOWN (UK-006); pre-norm vs post-norm placement = UNKNOWN (UK-005) |

### 3.3 AC-03: RMSNorm Output (Pre-Token-Mixer)

| Field | Value |
|---|---|
| Identity | `rmsnorm_pre_out[L]` — normalized states fed to token mixer |
| Originating operator | OC-6 (RMSNorm), weight `input_layernorm.weight [5120]` |
| Consuming operator | Token mixer (OC-3 full-attention or OC-4 linear-attention at layer L) |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From RMSNorm output through token mixer consumption; then reusable |
| Reuse relationship | Potentially overwrites `layer_input[L]` buffer after RMSNorm |
| Persistence category | ACTIVATION (per-layer, per-forward-pass) |
| Provenance | SET3-OC-6 §3.6; SET0-05 §5 (placement diagram); T4.1 §3.5 RM-023 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | Whether RMSNorm is applied before or after residual = UNKNOWN (UK-005); whether output is materialized separately or fused = UNKNOWN |

### 3.4 AC-04: RMSNorm Output (Post-Token-Mixer, Pre-MLP)

| Field | Value |
|---|---|
| Identity | `rmsnorm_post_out[L]` — normalized states fed to MLP |
| Originating operator | OC-6 (RMSNorm), weight `post_attention_layernorm.weight [5120]` |
| Consuming operator | OC-5 (DenseGatedSiLUMLP) at layer L |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From RMSNorm output through MLP consumption; then reusable |
| Reuse relationship | Potentially overwrites token mixer output buffer |
| Persistence category | ACTIVATION (per-layer, per-forward-pass) |
| Provenance | SET3-OC-6 §3.6; SET0-05 §5 (placement diagram); T4.1 §3.5 RM-024 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | Same as AC-03 — placement (UK-005), materialization (UNKNOWN) |

### 3.5 AC-05: Full-Attention Q/K/V Projection Outputs

| Field | Value |
|---|---|
| Identity | `fa_qkv_out[L]` — Q, K, V projection outputs at full-attention layer L |
| Originating operator | OC-3 (Qwen3_5Attention): `q_proj`, `k_proj`, `v_proj` |
| Consuming operator | Q/K normalization (OC-3 internal) → RoPE → KV cache |
| Shape (Q) | `[B, S, 12288]` = `[B, S, 24 × 256]` |
| Shape (K) | `[B, S, 1024]` = `[B, S, 4 × 256]` |
| Shape (V) | `[B, S, 1024]` = `[B, S, 4 × 256]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count (Q) | `B × S × 12288` |
| Element count (K) | `B × S × 1024` |
| Element count (V) | `B × S × 1024` |
| Byte size (Q) | `B × S × 12288 × E` |
| Byte size (K) | `B × S × 1024 × E` |
| Byte size (V) | `B × S × 1024 × E` |
| Lifetime | From projection until K/V stored in KV cache and Q consumed by attention; Q may be released after attention score computation |
| Reuse relationship | K and V may be reshaped in-place into KV cache format; Q may be released after QK product |
| Persistence category | ACTIVATION (per-layer, per-forward-pass) |
| Provenance | SET3-OC-3 §3.3 §6.2 (Q/KV projection stage); SET0-04 §3.3; SET0-08 §2 (tensor shapes); T4.1 §3.3 RM-014 |
| Classification | VERIFIED FACT (shapes from raw tensor metadata: q_proj `[12288, 5120]`, k_proj `[1024, 5120]`, v_proj `[1024, 5120]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether Q/K/V projections are fused into one matmul = UNKNOWN (UK-003); Q projection dimension 12288 includes QK or gate dimension — exact decomposition = UNKNOWN (UK-003) |

### 3.6 AC-06: Full-Attention Q/K Normalization Outputs

| Field | Value |
|---|---|
| Identity | `fa_q_norm_out[L]`, `fa_k_norm_out[L]` — normalized Q and K vectors |
| Originating operator | OC-3: `self_attn.q_norm.weight [256]`, `self_attn.k_norm.weight [256]` |
| Consuming operator | RoPE application → QK attention scores |
| Shape (Q norm) | `[B, S, 256]` per query head or `[B, 24, S, 256]` reshaped |
| Shape (K norm) | `[B, S, 256]` per KV head or `[B, 4, S, 256]` reshaped |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count (Q norm) | `B × S × 256` (or `B × 24 × S × 256` reshaped) |
| Element count (K norm) | `B × S × 256` (or `B × 4 × S × 256` reshaped) |
| Byte size (Q norm) | `B × S × 256 × E` |
| Byte size (K norm) | `B × S × 256 × E` |
| Lifetime | From norm output through RoPE and QK product computation; K norm output may persist until stored in KV cache |
| Reuse relationship | Potentially fused with RoPE; K norm output may be written directly into KV cache format |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-3 §3.3 §6.2 (Q/K normalization stage); SET0-04 §3.3; T4.1 §3.3 RM-015 |
| Classification | VERIFIED FACT (norm weight tensors, config); CONDITIONAL MODEL (runtime output shapes depend on head-count layout) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether applied in-place or as separate buffers = UNKNOWN (UK-003); whether Q norm is over 256-dim head or 12288-dim projected Q = UNKNOWN (UK-003) |

### 3.7 AC-07: Full-Attention KV Cache Store (Activation Boundary Only)

| Field | Value |
|---|---|---|
| Identity | `kv_store_boundary[L]` — activation-side observation that K/V projection outputs feed a downstream KV-cache state transition |
| Originating operator | OC-3 (Qwen3_5Attention) KV cache store stage (boundary only) |
| Consuming operator | Downstream KV-cache state (T4.4 — NOT modeled here) |
| Shape (K store) | UNKNOWN at state level. Activation-side boundary: K projection output `[B, S, 1024]` (AC-05 K) |
| Shape (V store) | UNKNOWN at state level. Activation-side boundary: V projection output `[B, S, 1024]` (AC-05 V) |
| Dtype | Activation-side: BF16 (conditional, UK-004). KV-cache state dtype = UNKNOWN (UK-004) — NOT modeled here |
| Element count (K) | Activation-side: `B × S × 1024`. State-level element count = UNKNOWN (T4.4 domain) |
| Element count (V) | Activation-side: `B × S × 1024`. State-level element count = UNKNOWN (T4.4 domain) |
| Byte size (K) | Activation-side: `B × S × 1024 × E`. State-level byte size = UNKNOWN (T4.4 domain) |
| Byte size (V) | Activation-side: `B × S × 1024 × E`. State-level byte size = UNKNOWN (T4.4 domain) |
| Lifetime | T4.3 boundary observation only: K and V projection outputs are consumed by the KV-cache store stage. |
| Reuse relationship | Activation-side: K and V proj outputs consumed at store boundary; then transfer ownership to downstream state (T4.4 models cache residency, cycling, paging). |
| Persistence category | ACTIVATION BOUNDARY — the KV-cache state object itself is STATEFUL and belongs to T4.4 |
| Provenance | SET3-OC-3 §3.3 §6.2 (KV Cache store stage); SET0-04 §3.2; T4.1 §3.3 RM-012, RM-013 |
| Classification | ACTIVATION BOUNDARY (the boundary observation); KV-cache state residency/allocation = UNKNOWN (UK-011 — T4.4 domain) |
| Unknown/conditional | K/V cache allocation strategy = UNKNOWN (UK-011, T4.4) — NOT resolved in T4.3; runtime state dtype = UNKNOWN (UK-004, T4.4) — NOT resolved in T4.3; whether K/V stored separately or interleaved = UNKNOWN (UK-003, T4.4); whether the cache is circular/paged = UNKNOWN (UK-011, T4.4) |
| Downstream boundary | Projection activation = T4.3 (AC-05). KV-cache residency / allocation / state = T4.4 UNKNOWN |
### 3.8 AC-08: Full-Attention QK Matrix Product

| Field | Value |
|---|---|
| Identity | `qk_product[L]` — query-key attention score matrix at full-attention layer L |
| Originating operator | OC-3 (Qwen3_5Attention) causal attention stage |
| Consuming operator | Scaling → softmax (OC-3 internal) |
| Shape | `[B, 24, S, S]` (full Q-head attention scores) |
| Dtype | UNKNOWN (UK-002, UK-004 — may be FP32 for numerical stability during softmax) |
| Element count | `B × 24 × S × S` |
| Byte size | `B × 24 × S × S × E_qk` where E_qk = UNKNOWN |
| Lifetime | From QK computation through softmax; released after attention weights computed |
| Reuse relationship | Potentially fused with softmax (not materialized separately) |
| Persistence category | TRANSIENT / WORKSPACE |
| Provenance | SET3-OC-3 §3.3 §6.2 (Causal Attention stage); SET0-04 §3.3; T4.1 §3.6 RM-031 |
| Classification | CONDITIONAL MODEL |
| Unknown/conditional | Whether QK product is materialized separately or fused with softmax = UNKNOWN (UK-003); whether scaling factor is applied and its value = UNKNOWN (UK-010); whether QK uses full 24 Q-head count or grouped layout = UNKNOWN (UK-003); runtime B, S = UNKNOWN (UK-009) |

### 3.9 AC-09: Full-Attention Softmax Output (Attention Weights)

| Field | Value |
|---|---|
| Identity | `attention_weights[L]` — normalized attention weights after causal softmax |
| Originating operator | OC-3 (Qwen3_5Attention) softmax stage |
| Consuming operator | Weighted sum with values (OC-3 internal) |
| Shape | `[B, 24, S, S]` |
| Dtype | UNKNOWN (UK-004) |
| Element count | `B × 24 × S × S` |
| Byte size | `B × 24 × S × S × E_softmax` where E_softmax = UNKNOWN |
| Lifetime | From softmax output through weighted sum computation; released after attention output |
| Reuse relationship | Potentially fused with weighted sum |
| Persistence category | TRANSIENT / WORKSPACE |
| Provenance | SET3-OC-3 §3.3 §6.2 (Causal Attention stage); SET0-04 §3.3; T4.1 §3.6 RM-032 |
| Classification | CONDITIONAL MODEL |
| Unknown/conditional | Exact softmax implementation = UNKNOWN (UK-003, UK-010); whether materialized or fused = UNKNOWN; dtype = UNKNOWN (UK-004); B, S = UNKNOWN (UK-009) |

### 3.10 AC-10: Full-Attention Weighted Sum (Attention Output Before O-Proj)

| Field | Value |
|---|---|
| Identity | `attn_weighted_sum[L]` — result of attention_weights × V |
| Originating operator | OC-3 (Qwen3_5Attention) weighted-sum stage |
| Consuming operator | OC-3 output gate (OC-9) → o_proj (OC-3) |
| Shape | `[B, 24, S, 256]` (per Q head, per KV head group) — expanded from `[B, 4, S, 256]` via GQA broadcasting |
| Dtype | UNKNOWN (UK-004) |
| Element count | `B × 24 × S × 256` |
| Byte size | `B × 24 × S × 256 × E_attn` where E_attn = UNKNOWN |
| Lifetime | From weighted sum through output gating and output projection; released after o_proj |
| Reuse relationship | Potentially fused with o_proj |
| Persistence category | TRANSIENT / WORKSPACE |
| Provenance | SET3-OC-3 §3.3 §6.2 (Causal Attention stage); SET0-04 §3.3; T4.1 §3.6 RM-033 |
| Classification | CONDITIONAL MODEL |
| Unknown/conditional | Whether weighted sum and o_proj are fused = UNKNOWN (UK-003); GQA head expansion representation = UNKNOWN (UK-003); dtype = UNKNOWN (UK-004); B, S = UNKNOWN (UK-009) |

### 3.11 AC-11: Full-Attention Output Gate Buffer

| Field | Value |
|---|---|
| Identity | `attn_output_gate_out[L]` — gated modulation output |
| Originating operator | OC-9 (AttentionOutputGate), config `attn_output_gate = true`, `output_gate_type = swish` |
| Consuming operator | Output projection o_proj (OC-3) |
| Shape | UNKNOWN — gate tensor existence and exact formulation are UNKNOWN (UK-006) |
| Dtype | UNKNOWN |
| Element count | UNKNOWN |
| Byte size | UNKNOWN |
| Lifetime | TRANSIENT |
| Reuse relationship | Unknown |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-9 §3.9; SET0-04 §5; T4.1 §3.3 RM-017 |
| Classification | UNKNOWN (gate tensor existence and shape) |
| Unknown/conditional | Whether a dedicated gate weight tensor exists = UNKNOWN (UK-006); whether gate is folded into o_proj = UNKNOWN; exact shape and dtype = UNKNOWN |

### 3.12 AC-12: Full-Attention Output Projection Output

| Field | Value |
|---|---|
| Identity | `fa_output[L]` — output of o_proj at full-attention layer L |
| Originating operator | OC-3: `self_attn.o_proj.weight [5120, 6144]` |
| Consuming operator | Residual addition + next RMSNorm (OC-6) in residual path |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From o_proj output through residual addition; then reusable as residual buffer |
| Reuse relationship | May be written in-place into the residual accumulation buffer |
| Persistence category | ACTIVATION (per-layer) |
| Provenance | SET3-OC-3 §3.3 §6.2 (Output Projection stage); SET0-04 §3.3; SET0-08 §2 (o_proj shape); T4.1 §3.3 RM-018 |
| Classification | VERIFIED FACT (shape structure from verified tensor shape) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether fused with gate output = UNKNOWN (UK-003); residual addition topology = UNKNOWN (UK-006) |

### 3.13 AC-13: Residual Connection Buffers (Per-Layer)

| Field | Value |
|---|---|
| Identity | `residual[L]` — residual skip connection accumulator at layer L |
| Originating operator | Layer input (RMSNorm pre-norm path) + token mixer output + MLP output |
| Consuming operator | Next RMSNorm (OC-6) — residual is added to sublayer outputs |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | Persists through the layer's sublayer operations (token mixer + MLP); released at layer completion or carried forward to next layer input |
| Reuse relationship | May be accumulated in-place (pre-norm residual add); may overwrite previous layer's residual |
| Persistence category | ACTIVATION (per-layer) |
| Provenance | SET3 §6.1 (decoder-layer dataflow); SET0-05 §13 (MLP placement); SET0-04 §3.1; T4.1 §3.5 RM-030 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | **Exact residual connection structure = UNKNOWN (UK-006).** SET3 §9 UK-006: "Exact residual connection structure (which residuals exist) = UNKNOWN." Number of residual connections, their exact placement (pre-norm vs post-norm), and in-place accumulation behavior are all UNKNOWN. Whether residues are accumulated in-place or via separate buffers = UNKNOWN. |

---

## 4. Linear-Attention Activations

### 4.1 AC-14: Linear-Attention QKV Projection Output

| Field | Value |
|---|---|
| Identity | `la_qkv_out[L]` — QKV projection output at linear-attention layer L |
| Originating operator | OC-4 (Qwen3_5GatedDeltaNet): `in_proj_qkv.weight [10240, 5120]` |
| Consuming operator | Causal convolution (OC-10) → gated delta rule (OC-4 internal) |
| Shape | `[B, S, 10240]` = `[B, S, 16×128 + 16×128 + 48×128]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 10240` |
| Byte size | `B × S × 10240 × E` |
| Lifetime | From projection through convolution input and gated-delta computation; QKV may be released after being consumed by conv + delta rule |
| Reuse relationship | May be fused with convolution input; QKV output may be overwritten after consumption |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-4 §3.4 §6.3 (QKV projection stage); SET0-04 §7.1; SET0-08 §2; T4.1 §3.4 RM-021 |
| Classification | VERIFIED FACT (shape from raw tensor shape `[10240, 5120]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether in_proj_qkv and in_proj_z are fused into one matmul = UNKNOWN; exact algorithm = UNKNOWN (UK-001); runtime dtype = UNKNOWN (UK-002) |

### 4.2 AC-15: Linear-Attention Z Projection Output

| Field | Value |
|---|---|
| Identity | `la_z_out[L]` — Z projection output at linear-attention layer L |
| Originating operator | OC-4 (Qwen3_5GatedDeltaNet): `in_proj_z.weight [6144, 5120]` |
| Consuming operator | Gated delta rule (OC-4 internal) |
| Shape | `[B, S, 6144]` = `[B, S, 48 × 128]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 6144` |
| Byte size | `B × S × 6144 × E` |
| Lifetime | From projection through gated-delta rule computation; released after consumption |
| Reuse relationship | May be fused with delta rule computation |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-4 §3.4 §6.3 (Z projection stage); SET0-04 §7.1; SET0-08 §2; T4.1 §3.4 RM-021 |
| Classification | VERIFIED FACT (shape from raw tensor shape `[6144, 5120]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether fused with QKV projection matmul = UNKNOWN; exact algorithm = UNKNOWN (UK-001); runtime dtype = UNKNOWN (UK-002) |

### 4.3 AC-16: Linear-Attention Output Projection Output

| Field | Value |
|---|---|
| Identity | `la_output[L]` — output of out_proj at linear-attention layer L |
| Originating operator | OC-4 (Qwen3_5GatedDeltaNet): `out_proj.weight [5120, 6144]` |
| Consuming operator | Residual addition + next RMSNorm (OC-6) in residual path |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From out_proj output through residual addition; then reusable as residual buffer or next layer input |
| Reuse relationship | May be written in-place into residual accumulation buffer |
| Persistence category | ACTIVATION (per-layer) |
| Provenance | SET3-OC-4 §3.4 §6.3 (output projection stage); SET0-04 §7.1; SET0-08 §2; T4.1 §3.4 RM-022 |
| Classification | VERIFIED FACT (shape structure from verified tensor shape) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); exact linear-attention algorithm = UNKNOWN (UK-001); runtime dtype = UNKNOWN (UK-002); residual topology = UNKNOWN (UK-006) |

---

## 5. MLP Activations (Including Gated-SiLU Computation)

### 5.1 AC-17: MLP Gate Projection Output

| Field | Value |
|---|---|
| Identity | `mlp_gate_out[L]` — output of gate_proj at layer L |
| Originating operator | OC-5 (DenseGatedSiLUMLP): `mlp.gate_proj.weight [17408, 5120]` |
| Consuming operator | SiLU activation (OC-5 internal) → elementwise multiply |
| Shape | `[B, S, I]` = `[B, S, 17408]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 17408` |
| Byte size | `B × S × 17408 × E` |
| Lifetime | From gate projection through SiLU + elementwise multiply; gate output consumed by SiLU |
| Reuse relationship | May be fused with SiLU (gate_proj → SiLU in one kernel) |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-5 §3.5 §6.4 (gate projection stage); SET0-05 §8; SET0-08 §2; T4.1 §3.5 RM-025 |
| Classification | VERIFIED FACT (shape from raw tensor shape `[17408, 5120]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether gate and up projections are fused = UNKNOWN (UK-004); runtime dtype = UNKNOWN (UK-002, UK-004) |

### 5.2 AC-18: MLP Up Projection Output

| Field | Value |
|---|---|
| Identity | `mlp_up_out[L]` — output of up_proj at layer L |
| Originating operator | OC-5 (DenseGatedSiLUMLP): `mlp.up_proj.weight [17408, 5120]` |
| Consuming operator | Elementwise multiply with SiLU(gate) (OC-5 internal) |
| Shape | `[B, S, I]` = `[B, S, 17408]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 17408` |
| Byte size | `B × S × 17408 × E` |
| Lifetime | From up projection through elementwise multiply with SiLU(gate) |
| Reuse relationship | May be fused with elementwise multiply or consumed in-place |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-5 §3.5 §6.4 (up projection stage); SET0-05 §8; SET0-08 §2; T4.1 §3.5 RM-026 |
| Classification | VERIFIED FACT (shape from raw tensor shape `[17408, 5120]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether fused with gate projection = UNKNOWN (UK-004); runtime dtype = UNKNOWN (UK-002, UK-004) |

### 5.3 AC-19: MLP SiLU Gate Output

| Field | Value |
|---|---|
| Identity | `mlp_silu_gate[L]` — SiLU activation applied to gate projection output |
| Originating operator | OC-5 (DenseGatedSiLUMLP): SiLU(gate_proj(x)) |
| Consuming operator | Elementwise multiply with up_proj output (OC-5 internal) |
| Shape | `[B, S, I]` = `[B, S, 17408]` |
| Dtype | UNKNOWN (UK-002 — may be FP32 for SiLU numerical stability; config does not confirm) |
| Element count | `B × S × 17408` |
| Byte size | `B × S × 17408 × E_silu` where E_silu = UNKNOWN (UK-002, UK-004) |
| Lifetime | From SiLU computation through elementwise multiply; released after multiply |
| Reuse relationship | May be fused with elementwise multiply; may be computed in-place on gate buffer |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-5 §3.5 §6.4 (SiLU gate stage); SET0-05 §6 (activation); SET0-05 §7; T4.1 §3.5 RM-027 |
| Classification | VERIFIED FACT (structure: SiLU on gate); UNKNOWN (runtime dtype, UK-002) |
| Unknown/conditional | Whether SiLU is computed in-place or as separate tensor = UNKNOWN; whether activation dtype is upcast = UNKNOWN (UK-002, UK-004); B, S = UNKNOWN (UK-009) |

### 5.4 AC-20: MLP Elementwise Product (Gate × Up)

| Field | Value |
|---|---|
| Identity | `mlp_gate_up_product[L]` — elementwise product SiLU(gate) ⊙ up_proj |
| Originating operator | OC-5 (DenseGatedSiLUMLP): elementwise multiplication |
| Consuming operator | down_proj (OC-5 internal) |
| Shape | `[B, S, I]` = `[B, S, 17408]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 17408` |
| Byte size | `B × S × 17408 × E` |
| Lifetime | From elementwise multiply through down_proj; may be fused with down_proj |
| Reuse relationship | May be fused with down_proj (gate_up_product + down_proj in one kernel) |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-5 §3.5 §6.4 (gated multiply stage); SET0-05 §7; SET0-05 §8 (structure diagram); T4.1 §3.5 RM-028 |
| Classification | VERIFIED FACT (structure: SiLU(gate) ⊙ up_proj) |
| Unknown/conditional | Whether elementwise product and down_proj are fused = UNKNOWN (UK-004); B, S = UNKNOWN (UK-009); runtime dtype = UNKNOWN (UK-002, UK-004) |

### 5.5 AC-21: MLP Down Projection Output

| Field | Value |
|---|---|
| Identity | `mlp_down_out[L]` — output of down_proj at layer L |
| Originating operator | OC-5 (DenseGatedSiLUMLP): `mlp.down_proj.weight [5120, 17408]` |
| Consuming operator | Residual addition + next RMSNorm (OC-6) in residual path |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From down projection through residual addition; then reusable as layer output or residual buffer |
| Reuse relationship | May be written in-place into the residual accumulation buffer |
| Persistence category | ACTIVATION (per-layer) |
| Provenance | SET3-OC-5 §3.5 §6.4 (down projection stage); SET0-05 §8; SET0-08 §2; T4.1 §3.5 RM-029 |
| Classification | VERIFIED FACT (shape from raw tensor shape `[5120, 17408]`) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether fused with gate_up_product = UNKNOWN (UK-004); runtime dtype = UNKNOWN (UK-002, UK-004); residual topology = UNKNOWN (UK-006) |

---

## 6. LM-Head and Output Activations

### 6.1 AC-22: Final LayerNorm Output

| Field | Value |
|---|---|
| Identity | `final_norm_out` — output of final RMSNorm before LM head |
| Originating operator | OC-7 (FinalRMSNorm): `model.language_model.norm.weight [5120]` |
| Consuming operator | OC-8 (LMHead) |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From final norm output through LM head consumption; released after logits computed |
| Reuse relationship | May be fused with LM head; may reuse last layer's output buffer |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-7 §3.7; SET3 §6.5 (full language stack dataflow); T4.1 §3.1, RM-029 |
| Classification | VERIFIED FACT (shape structure, operator relationship) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether final norm is applied or skipped = UNKNOWN (UK-005); whether fused with LM head = UNKNOWN |

### 6.2 AC-23: LM-Head Logits Output

| Field | Value |
|---|---|
| Identity | `logits` — output of LM head projection |
| Originating operator | OC-8 (LMHead): `lm_head.weight [248320, 5120]` |
| Consuming operator | Sampling / log-softmax (runtime, not in SET3 operator model) |
| Shape | `[B, S, V]` = `[B, S, 248320]` |
| Dtype | UNKNOWN (UK-004 — may be FP32 for log-softmax stability) |
| Element count | `B × S × 248320` |
| Byte size | `B × S × 248320 × E_logits` where E_logits = UNKNOWN (UK-004) |
| Lifetime | From LM head output through sampling; released after token selected |
| Reuse relationship | May be partially materialized (top-k only); may overwrite input buffer |
| Persistence category | OUTPUT (inference-request scope) |
| Provenance | SET3-OC-8 §3.8; SET3 §6.5 (full language stack dataflow); T4.1 §3.2 RM-011 |
| Classification | VERIFIED FACT (shape structure: vocab_size × hidden_size → vocab_size); UNKNOWN (runtime dtype, UK-004) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether logits materialized in full or via streaming/top-k = UNKNOWN; runtime dtype = UNKNOWN (UK-004) |

### 6.3 AC-24: Layer L Output (Residual Post-MLP)

| Field | Value |
|---|---|
| Identity | `layer_output[L]` — output after token mixer + MLP residual additions |
| Originating operator | Layer L residual path output |
| Consuming operator | Layer L+1 input (RMSNorm at L+1) |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From layer L output through layer L+1 RMSNorm consumption; then reusable as layer L+1 input |
| Reuse relationship | May overwrite layer L's intermediate buffers; may be the same buffer as layer L+1's input |
| Persistence category | ACTIVATION (per-layer) |
| Provenance | SET3 §6.1 (decoder-layer dataflow); SET0-05 §13 (MLP placement); SET3 §6.5 (full stack); T4.1 §3.5 RM-029, §3.7 RM-030 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); exact residual topology = UNKNOWN (UK-006); pre/post-norm = UNKNOWN (UK-005) |

---

## 7. Vision-to-Language Activations

### 7.1 AC-25: Vision Input Pixel Values

| Field | Value |
|---|---|
| Identity | `pixel_values` — raw image/video pixel input |
| Originating operator | OC-2 (VisionEncoder) |
| Consuming operator | Vision encoder patch embedding |
| Shape | `[B, num_frames, 3, H, W]` |
| Dtype | UNKNOWN (config declares BF16 for model tensors; input pixel dtype = implementation-dependent) |
| Element count | `B × num_frames × 3 × H × W` |
| Byte size | UNKNOWN — H, W runtime resolution = UNKNOWN (UK-009) |
| Lifetime | From input through vision encoder consumption |
| Reuse relationship | Conditional |
| Persistence category | INPUT / OUTPUT |
| Provenance | SET3-OC-2 §3.2; SET0-06 §2; T4.1 §3.8 RM-034 |
| Classification | VERIFIED FACT (shape structure, config); UNKNOWN (runtime dtype, H/W) |
| Unknown/conditional | Runtime H, W = UNKNOWN (UK-009); input dtype = UNKNOWN; whether vision encoder invoked = UNKNOWN (UK-007) |

### 7.2 AC-26: Vision Encoder Activations

| Field | Value |
|---|---|
| Identity | `vision_layer_activations[L_v]` — intermediate activations per vision block (27 blocks) |
| Originating operator | OC-2 (VisionEncoder) per vision block |
| Consuming operator | Next vision block |
| Shape | `[N_patches, 1152]` or similar per vision block (CONDITIONAL) |
| Dtype | UNKNOWN (UK-004) |
| Element count | UNKNOWN — depends on input resolution and patch count |
| Byte size | UNKNOWN — depends on input resolution, patch count, and dtype |
| Lifetime | Per-vision-block, forward pass only |
| Reuse relationship | Conditional |
| Persistence category | ACTIVATION (per-vision-block) |
| Provenance | SET3-OC-2 §3.2; SET0-06 §2 (config: hidden=1152, depth=27, patch_size=16); T4.1 §3.8 RM-035 |
| Classification | VERIFIED FACT (config); UNKNOWN (exact activation shapes, per-tensor vision naming) |
| Unknown/conditional | Exact per-tensor vision tensor naming = UNKNOWN (SET3-OC-2 §2.1); vision activation shapes derived from config but not verified from checkpoints; whether vision encoder invoked = UNKNOWN (UK-007) |

### 7.3 AC-27: Vision-to-Language Projection Output

| Field | Value |
|---|---|
| Identity | `vision_language_features` — projected vision features for language merging |
| Originating operator | OC-2 (VisionEncoder) vision-language merger |
| Consuming operator | Language embedding sequence (merged with token embeddings) |
| Shape | `[N, 5120]` — output width matches language hidden_size (VERIFIED FACT) |
| Dtype | BF16 (conditional; runtime dtype UNKNOWN, UK-004) |
| Element count | `N × 5120` where N = number of vision tokens (UNKNOWN) |
| Byte size | `N × 5120 × E` where N = UNKNOWN |
| Lifetime | From vision encoder output through language embedding sequence merge |
| Reuse relationship | Conditional |
| Persistence category | ACTIVATION / TRANSIENT |
| Provenance | SET3-OC-2 §2.5 (output width = 5120 matches hidden_size); SET0-06 §3; T4.1 §3.8 RM-036 |
| Classification | VERIFIED FACT (output width match); UNKNOWN (fusion mechanism, UK-007) |
| Unknown/conditional | **Exact multimodal fusion mechanism = UNKNOWN (UK-007).** Whether vision features are added (requiring same shape) or concatenated (requiring projection to match) is UNKNOWN. This affects the shape of the merged sequence, which is UNKNOWN. N (number of vision tokens) = UNKNOWN. Whether vision encoder is invoked = UNKNOWN (UK-007). |

---

## 8. MTP-Related Activations (Conditional)

### 8.1 AC-28: MTP Projection Activations

| Field | Value |
|---|---|
| Identity | `mtp_activations` — intermediate activations flowing through the MTP subnetwork |
| Originating operator | OC-11 (MTPHead): `mtp.fc.weight [5120, 10240]`, `mtp.layers.0.*` (15 tensors total) |
| Consuming operator | MTP output merge into language stack |
| Shape | Conditioned on MTP internal structure; fc input `[B, S, 10240]`, fc output `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-002, UK-004) |
| Element count | UNKNOWN — depends on whether MTP is actively executed |
| Byte size | UNKNOWN — depends on whether MTP is actively executed |
| Lifetime | UNKNOWN — depends on whether MTP is actively executed |
| Reuse relationship | Unknown |
| Persistence category | ACTIVATION / TRANSIENT (if executed) |
| Provenance | SET3-OC-11 §3.11; SET0-06 §6; SET0-08 §3 (exact MTP tensor shapes); T4.1 §3.1 RM-009 |
| Classification | VERIFIED FACT (checkpoint tensor existence, shapes, dtype, count = 15); UNKNOWN (MTP active runtime execution, UK-008) |
| Unknown/conditional | **Whether MTP is actively executed during ordinary generation = UNKNOWN (UK-008).** SET0-06 §6: "MTP active runtime execution = UNKNOWN." SET0-08 §4: "MTP active runtime execution = UNKNOWN." SET3-OC-11 §2.6: "UNKNOWN: active runtime execution path." SET1-T1.6 §4: "MTP runtime execution behavior: UNKNOWN." All 15 MTP checkpoint tensors are VERIFIED (existence, shapes, dtype); the exact runtime activation, scheduling, and memory behavior of MTP is UNKNOWN. If MTP is not actively executed, `mtp_activations` do not exist at runtime. |

### 8.2 AC-29: MTP Pre-Final-Norm Embeddings

| Field | Value |
|---|---|
| Identity | `mtp_pre_fc_norm_embedding` / `mtp_pre_fc_norm_hidden` — MTP pre-final-norm normalization inputs |
| Originating operator | OC-11 (MTPHead) |
| Consuming operator | OC-11 final normalization |
| Shape (embedding) | `[B, S, 5120]` or `[num_mtp_tokens, 5120]` (CONDITIONAL) |
| Shape (hidden) | `[B, S, 5120]` or `[num_mtp_tokens, 5120]` (CONDITIONAL) |
| Dtype | BF16 (conditional) |
| Element count | UNKNOWN — depends on MTP runtime execution |
| Byte size | UNKNOWN |
| Lifetime | UNKNOWN |
| Persistence category | ACTIVATION (if executed) |
| Provenance | SET0-08 §3 (tensor shapes); SET0-06 §6; T4.1 §3.1 RM-009; config `mtp_use_dedicated_embeddings = false` (VERIFIED — SET3-OC-11 §2.6) |
| Classification | VERIFIED FACT (checkpoint tensor `mtp.pre_fc_norm_embedding.weight [5120]`, `mtp.pre_fc_norm_hidden.weight [5120]`); UNKNOWN (runtime activation, UK-008) |
| Unknown/conditional | MTP runtime execution = UNKNOWN (UK-008); whether embeddings are shared with language model = UNKNOWN (`mtp_use_dedicated_embeddings = false` is VERIFIED FACT config but runtime behavior is UNKNOWN) |

---

## 9. Positional and Rotary Buffers

### 9.1 AC-30: RoPE Cos/Sin Buffers (Precomputed or Per-Token)

| Field | Value |
|---|---|
| Identity | `rope_cos`, `rope_sin` — rotary position embedding cosine/sine tables |
| Originating operator | OC-3 (Qwen3_5Attention) RoPE application |
| Consuming operator | Q/K rotation in full-attention layers |
| Shape | `[S, 64]` (rotary_dim = 64 = 256 × 0.25) if precomputed per sequence; or `[262144, 64]` if precomputed for max_position_embeddings |
| Dtype | UNKNOWN (UK-004) |
| Element count | `S × 64` (per-sequence) or `262144 × 64` (for max_length precomputation) |
| Byte size | `S × 64 × E_rope` (per-sequence) or `262144 × 64 × E_rope` (max_length) — E_rope = UNKNOWN (UK-004) |
| Lifetime | WORKSPACE / STATIC (if precomputed) or TRANSIENT (if computed per-token/step) |
| Reuse relationship | Precomputed tables may be cached across all full-attention layers; per-token computation may be fused |
| Persistence category | CONDITIONAL (workspace/static or transient) |
| Provenance | SET3-OC-3 §3.3 (RoPE config); SET0-04 §4 (rotary dimension = 64); SET0-04 §4 (MRoPE sections); T4.1 §3.3 RM-016 |
| Classification | VERIFIED FACT (config fields: rope_theta=1e7, partial_rotary_factor=0.25, rotary_dim=64, max_position_embeddings=262144); UNKNOWN (runtime allocation strategy, UK-003) |
| Unknown/conditional | **Exact semantics of MRoPE sections in every execution path = UNKNOWN (UQ-006, SET0-04 §17).** Whether RoPE cos/sin tables are precomputed and cached or computed on-the-fly per token = UNKNOWN (implementation detail, downstream SET5). Whether the full RoPE table for `max_position_embeddings = 262144` is precomputed = UNKNOWN. MRoPE section semantics = UNKNOWN (UQ-006). |

### 9.2 AC-31: Position IDs

| Field | Value |
|---|---|
| Identity | `position_ids` — positional indices for token positions |
| Originating operator | Input preprocessing |
| Consuming operator | OC-3 (RoPE), OC-4 (positional injection) |
| Shape | `[B, S]` |
| Dtype | int64 / int32 (UNKNOWN at runtime — config does not specify) |
| Element count | `B × S` |
| Byte size | `B × S × E_pos` where E_pos = 4 or 8 bytes (UNKNOWN) |
| Lifetime | INPUT / OUTPUT — inference request scope |
| Reuse relationship | Conditional |
| Persistence category | INPUT / OUTPUT |
| Provenance | SET3 §6.2 (RoPE stage); SET0 §6 (RoPE config); T4.1 §3.8 RM-038 |
| Classification | VERIFIED FACT (concept: position_ids exist); UNKNOWN (runtime representation) |
| Unknown/conditional | Whether position IDs are explicit tensors or implicit (computed from sequence position) = UNKNOWN; whether positions are 1D or 2D (MRoPE) = UNKNOWN (UQ-006); B, S = UNKNOWN (UK-009) |

---

## 10. Input / Output Runtime Buffers

### 10.1 AC-32: Input Token IDs

| Field | Value |
|---|---|
| Identity | `input_ids` — tokenized input representation |
| Originating operator | Input preprocessing / tokenizer |
| Consuming operator | OC-1 (LanguageEmbedding) |
| Shape | `[B, S]` |
| Dtype | int64 / int32 (UNKNOWN at runtime — implementation-dependent) |
| Element count | `B × S` |
| Byte size | `B × S × E_ids` where E_ids = 4 or 8 bytes (UNKNOWN) |
| Lifetime | INPUT — inference request scope |
| Reuse relationship | Conditional |
| Persistence category | INPUT / OUTPUT |
| Provenance | SET3-OC-1 §3.1; SET3 §6.1; T4.1 §3.8 RM-037 |
| Classification | VERIFIED FACT (concept: input_ids exist for embedding lookup); UNKNOWN (runtime dtype, B, S) |
| Unknown/conditional | B = UNKNOWN (UK-009); S = UNKNOWN (UK-009); token ID dtype = UNKNOWN |

### 10.2 AC-33: Attention Mask / Causal Mask

| Field | Value |
|---|---|
| Identity | `attention_mask` / `causal_mask` — causal attention mask |
| Originating operator | Input preprocessing |
| Consuming operator | OC-3 (Qwen3_5Attention) causal attention stage |
| Shape | `[B, 1, S, S]` or `[B, S, 1, S]` (CONDITIONAL — exact layout UNKNOWN) |
| Dtype | BF16 / float (UNKNOWN at runtime) |
| Element count | `B × S × S` (approximate) |
| Byte size | `B × S × S × E_mask` where E_mask = UNKNOWN |
| Lifetime | TRANSIENT / WORKSPACE |
| Reuse relationship | Conditional — may be applied implicitly via kernel masking |
| Persistence category | TRANSIENT / WORKSPACE |
| Provenance | SET3-OC-3 §3.3 §6.2 (Causal Attention stage); SET0-04 §3.3; T4.1 §3.8 RM-039 |
| Classification | VERIFIED FACT (existence); CONDITIONAL MODEL (shape, exact representation) |
| Unknown/conditional | Whether causal mask is materialized as a tensor or applied implicitly via kernel masking = UNKNOWN; exact mask layout = UNKNOWN (UK-003); whether linear-attention layers use an attention mask = UNKNOWN; B, S = UNKNOWN (UK-009) |

### 10.3 AC-34: Final Output Hidden States

| Field | Value |
|---|---|
| Identity | `hidden_states_out` — final hidden states before LM head |
| Originating operator | OC-7 (FinalRMSNorm) |
| Consuming operator | OC-8 (LMHead) |
| Shape | `[B, S, H]` = `[B, S, 5120]` |
| Dtype | BF16 (conditional; runtime computation dtype UNKNOWN, UK-004) |
| Element count | `B × S × 5120` |
| Byte size | `B × S × 5120 × E` |
| Lifetime | From final norm output through LM head consumption; released after logits |
| Reuse relationship | May reuse last layer's output buffer if final norm is fused |
| Persistence category | OUTPUT (inference-request scope) |
| Provenance | SET3 §6.5 (full language stack dataflow); SET3-OC-7 §3.7; T4.1 §3.8 RM-040 |
| Classification | VERIFIED FACT (shape structure) |
| Unknown/conditional | B, S = UNKNOWN (UK-009); whether fused with last layer output = UNKNOWN |

---

## 11. Linear-Attention Intermediate Buffers (Conditional)

### 11.1 AC-35: Linear-Attention Gated-Delta Intermediates

| Field | Value |
|---|---|
| Identity | `gated_delta_intermediates[L]` — intermediate tensors for gated-delta-rule computation |
| Originating operator | OC-4 (Qwen3_5GatedDeltaNet) gated delta rule stage |
| Consuming operator | Recurrent state update (OC-4 internal) |
| Shape | UNKNOWN — depends on the exact linear-attention algorithm (UK-001) |
| Dtype | UNKNOWN (UK-002) |
| Element count | UNKNOWN |
| Byte size | UNKNOWN |
| Lifetime | TRANSIENT / WORKSPACE — per-linear-attention-layer |
| Reuse relationship | Conditional |
| Persistence category | TRANSIENT / WORKSPACE |
| Provenance | SET3-OC-4 §3.4 §6.3 (Gated Delta Rule stage); SET0-04 §7 (linear attention implementation); T4.1 §3.10.3 RM-044 |
| Classification | CONDITIONAL MODEL (existence implied by SET3 dataflow; exact shape = UNKNOWN, depends on UK-001) |
| Unknown/conditional | **Exact linear-attention algorithm = UNKNOWN (UK-001).** The gated-delta-rule intermediate buffer shapes and count are dependent on the unresolved algorithm. No intermediate buffer shape is asserted as VERIFIED FACT. Runtime dtype = UNKNOWN (UK-002). B, S = UNKNOWN (UK-009). |

### 11.2 AC-36: Linear-Attention Convolution State (Activation Boundary Only)

| Field | Value |
|---|---|---|
| Identity | `la_conv_state_boundary[L]` — activation-side observation that linear-attention QKV projection output feeds a downstream convolution state transition |
| Originating operator | OC-10 (CausalConv1D) within OC-4 (Qwen3_5GatedDeltaNet) (boundary only) |
| Consuming operator | Downstream linear-attention state (T4.5 — NOT modeled here) |
| Shape | UNKNOWN at state level. Activation-side boundary: QKV projection output `[B, S, 10240]` (AC-14). The runtime convolution state buffer shape is NOT established — UNKNOWN (UK-012, T4.5 domain). |
| Dtype | Activation-side: BF16 (conditional, UK-002, UK-004). Convolution state dtype = UNKNOWN (UK-002) — NOT modeled here |
| Element count | UNKNOWN at state level. Activation-side: `B × S × 10240` (AC-14 consumed). |
| Byte size | UNKNOWN at state level. Activation-side: `B × S × 10240 × E`. |
| Lifetime | T4.3 boundary observation only: QKV projection output is consumed by the convolution stage. |
| Reuse relationship | Activation-side: QKV output consumed at conv boundary; then transfer ownership to downstream state (T4.5 models recurrence state, cycling, allocation). |
| Persistence category | ACTIVATION BOUNDARY — the convolution state buffer itself is STATEFUL and belongs to T4.5 |
| Provenance | SET3-OC-4 §3.4 (state model); SET3-OC-10 §3.10 (CausalConv1D); SET0-04 §8; T4.1 §3.4 RM-020 |
| Classification | ACTIVATION BOUNDARY (the boundary observation); convolution state buffer shape = UNKNOWN (UK-012 — T4.5 domain) |
| Unknown/conditional | Exact runtime convolution state allocation = UNKNOWN (UK-012, T4.5) — NOT resolved in T4.3; exact linear-attention algorithm = UNKNOWN (UK-001, T4.5) — NOT resolved in T4.3; runtime state dtype = UNKNOWN (UK-002, T4.5) — NOT resolved in T4.3 |
| Downstream boundary | QKV projection activation = T4.3 (AC-14). Convolution / recurrence state = T4.5 UNKNOWN |
## 12. Activation Shape Model

The following table consolidates all activation object shapes parameterized by B (batch), S (sequence length), and the structural constants defined in §2. Runtime dtype and B/S values remain UNKNOWN (UK-004, UK-009).

| AC ID | Object | Shape (B, S parameterized) | Element Count | Byte Size | Classification |
|---|---|---|---|---|---|
| AC-01 | `embeddings` (embed output) | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-02 | `layer_input[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-03 | `rmsnorm_pre_out[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-04 | `rmsnorm_post_out[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-05 | `fa_q_out` | `[B, S, 12288]` | `B·S·12288` | `B·S·12288·E` | VERIFIED FACT |
| AC-05 | `fa_k_out` | `[B, S, 1024]` | `B·S·1024` | `B·S·1024·E` | VERIFIED FACT |
| AC-05 | `fa_v_out` | `[B, S, 1024]` | `B·S·1024` | `B·S·1024·E` | VERIFIED FACT |
| AC-06 | `fa_q_norm_out` | `[B, S, 256]` | `B·S·256` | `B·S·256·E` | VERIFIED FACT |
| AC-06 | `fa_k_norm_out` | `[B, S, 256]` | `B·S·256` | `B·S·256·E` | VERIFIED FACT |
| AC-07 | `kv_store_boundary[L]` (K, V boundary) | UNKNOWN (state: T4.4) | UNKNOWN | UNKNOWN | ACTIVATION BOUNDARY |
| AC-08 | `qk_product[L]` | `[B, 24, S, S]` | `B·24·S²` | `B·24·S²·E_qk` | CONDITIONAL MODEL |
| AC-09 | `attention_weights[L]` | `[B, 24, S, S]` | `B·24·S²` | `B·24·S²·E_sm` | CONDITIONAL MODEL |
| AC-10 | `attn_weighted_sum[L]` | `[B, 24, S, 256]` | `B·24·S·256` | `B·24·S·256·E_attn` | CONDITIONAL MODEL |
| AC-11 | `attn_output_gate_out[L]` | UNKNOWN | UNKNOWN | UNKNOWN | UNKNOWN |
| AC-12 | `fa_output[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-13 | `residual[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-14 | `la_qkv_out[L]` | `[B, S, 10240]` | `B·S·10240` | `B·S·10240·E` | VERIFIED FACT |
| AC-15 | `la_z_out[L]` | `[B, S, 6144]` | `B·S·6144` | `B·S·6144·E` | VERIFIED FACT |
| AC-16 | `la_output[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-17 | `mlp_gate_out[L]` | `[B, S, 17408]` | `B·S·17408` | `B·S·17408·E` | VERIFIED FACT |
| AC-18 | `mlp_up_out[L]` | `[B, S, 17408]` | `B·S·17408` | `B·S·17408·E` | VERIFIED FACT |
| AC-19 | `mlp_silu_gate[L]` | `[B, S, 17408]` | `B·S·17408` | `B·S·17408·E_silu` | VERIFIED FACT / UNKNOWN (dtype) |
| AC-20 | `mlp_gate_up_product[L]` | `[B, S, 17408]` | `B·S·17408` | `B·S·17408·E` | VERIFIED FACT |
| AC-21 | `mlp_down_out[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-22 | `final_norm_out` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-23 | `logits` | `[B, S, 248320]` | `B·S·248320` | `B·S·248320·E_logits` | VERIFIED FACT / UNKNOWN (dtype) |
| AC-24 | `layer_output[L]` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-25 | `pixel_values` | `[B, num_frames, 3, H, W]` | `B·num_frames·3·H·W` | UNKNOWN (H, W, dtype) | VERIFIED FACT (structure) |
| AC-26 | `vision_layer_activations[L_v]` | UNKNOWN (config-derived) | UNKNOWN | UNKNOWN | VERIFIED FACT (config) / UNKNOWN (shapes) |
| AC-27 | `vision_language_features` | `[N, 5120]` | `N·5120` | `N·5120·E` | VERIFIED FACT (width) / UNKNOWN (N) |
| AC-28 | `mtp_activations` | Conditioned on MTP execution | UNKNOWN | UNKNOWN | VERIFIED FACT (weights) / UNKNOWN (runtime) |
| AC-29 | `mtp_pre_fc_norm_*` | `[B, S, 5120]` (CONDITIONAL) | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT (tensors) / UNKNOWN (runtime) |
| AC-30 | `rope_cos/sin` | `[S, 64]` or `[262144, 64]` | `S×64` or `262144×64` | Parameterized | VERIFIED FACT (config) / UNKNOWN (strategy) |
| AC-31 | `position_ids` | `[B, S]` | `B·S` | `B·S·E_pos` | VERIFIED FACT (concept) / UNKNOWN (representation) |
| AC-32 | `input_ids` | `[B, S]` | `B·S` | `B·S·E_ids` | VERIFIED FACT (concept) / UNKNOWN (dtype) |
| AC-33 | `causal_mask` | `[B, 1, S, S]` or `[B, S, 1, S]` (CONDITIONAL) | `B·S²` (approx) | `B·S²·E_mask` | VERIFIED FACT (existence) / CONDITIONAL (layout) |
| AC-34 | `hidden_states_out` | `[B, S, 5120]` | `B·S·5120` | `B·S·5120·E` | VERIFIED FACT |
| AC-35 | `gated_delta_intermediates[L]` | UNKNOWN (UK-001) | UNKNOWN | UNKNOWN | CONDITIONAL MODEL |
| AC-36 | `la_conv_state_boundary[L]` (activation boundary) | UNKNOWN (state: T4.5) | UNKNOWN | UNKNOWN | ACTIVATION BOUNDARY |

Where: `E` = bytes_per_element (2 for BF16, parameterized), `E_qk` = UNKNOWN (UK-002, UK-004), `E_sm` = UNKNOWN (UK-004), `E_attn` = UNKNOWN (UK-004), `E_silu` = UNKNOWN (UK-002), `E_logits` = UNKNOWN (UK-004), `E_pos` = 4 or 8 (UNKNOWN), `E_ids` = 4 or 8 (UNKNOWN), `E_mask` = UNKNOWN. `E_state` (KV-cache / linear-attention state dtype) is NOT within T4.3 scope — it is T4.4/T4.5 UNKNOWN (UK-004).

---

## 13. Activation Lifetime Model

### 13.1 Lifetime Semantics

Activation lifetimes are defined relative to the SET3-established execution flow (§6.1–§6.5). The model distinguishes structural coexistence from implementation-specific scheduling.

### 13.2 Lifetime Ordering Framework

Activations are classified by their lifetime scope within the execution graph:

```text
Lifetime phases (structural):
  T_INIT    — Input preprocessing phase (input_ids, position_ids, attention_mask)
  T_EMBED   — Embed phase (embeddings)
  T_LAYER   — Per-layer phase (layer_input, rmsnorm_pre, token_mixer_activations,
              mlp_activations, residual, layer_output)
  T_FINAL   — Final norm + LM head phase (final_norm_out, logits, hidden_states_out)
  T_VISION  — Vision encoder phase — CONDITIONAL
  T_MTP     — MTP phase — CONDITIONAL
  T_ROPE    — RoPE table lifecycle — CONDITIONAL
```

Within T_LAYER, the intra-layer lifetime ordering (derived from SET3 §6.1) is:

```text
Layer L:
  1. layer_input[L]           → consumed by RMSNorm pre
  2. rmsnorm_pre_out[L]       → consumed by token mixer
  3. token_mixer_activations  → consumed by RMSNorm post (through residual)
  4. rmsnorm_post_out[L]      → consumed by MLP
  5. mlp_activations          → consumed by residual + layer_output[L]
  6. layer_output[L]          → becomes layer_input[L+1]
```

The exact intra-layer ordering (pre-norm vs post-norm, which residuals exist) is UNKNOWN (UK-005, UK-006).

### 13.3 Lifetime Categories

| Category | Definition | Verdict Source |
|---|---|---|
| TRANSIENT | Per token-step, per-layer; released after consuming operator completes | SET3 dataflow §6.1–§6.4 |
| PERSISTENT (per-layer) | Per layer, survives token-step but released at layer completion | SET3 §6.1 (layer-scoped activations) |
| STATEFUL | Persists across token-steps (KV cache, linear-attention state) | SET3 §6.2, §6.3, §7.1 |
| CONDITIONAL | Lifetime depends on unresolved runtime mechanism | UK-001, UK-003, UK-005, UK-006, UK-007, UK-008 |
| REQUEST-SCOPE | Persists for the full inference request (input_ids, logits) | SET3 §6.1 (input preprocessing) |
| STATIC | Precomputed once per session (RoPE tables if precomputed) | SET0-04 §4, SET3-OC-3 §3.3 |

### 13.4 Layer-Scoped Coexistence (Structural)

Within a single language layer L (SET3 §6.1 dataflow):

```text
Simultaneously live within Layer L (structural necessity):
  - layer_input[L]                 (AC-02)  [B, S, 5120]
  - rmsnorm_pre_out[L]             (AC-03)  [B, S, 5120]
  - token_mixer output             (AC-12/AC-16)  [B, S, 5120]
  - residual[L]                    (AC-13)  [B, S, 5120]
  - rmsnorm_post_out[L]            (AC-04)  [B, S, 5120]
  - mlp intermediates              (AC-17–AC-20)  up to 2 × [B, S, 17408]
  - mlp_down_out[L]                (AC-21)  [B, S, 5120]
  - layer_output[L]                (AC-24)  [B, S, 5120]
```

For full-attention layers, additional simultaneously live activations include:

```text
  - fa_q_out                       (AC-05)  [B, S, 12288]
  - fa_k_out                       (AC-05)  [B, S, 1024]
  - fa_v_out                       (AC-05)  [B, S, 1024]
  - fa_q_norm_out                  (AC-06)  [B, S, 256]
  - fa_k_norm_out                  (AC-06)  [B, S, 256]
  - qk_product                     (AC-08)  [B, 24, S, S]
  - attention_weights              (AC-09)  [B, 24, S, S]
  - attn_weighted_sum              (AC-10)  [B, 24, S, 256]
  - attn_output_gate_out           (AC-11)  UNKNOWN
  - fa_output[L]                   (AC-12)  [B, S, 5120]
```

For linear-attention layers, additional simultaneously live activations include:

```text
  - la_qkv_out                     (AC-14)  [B, S, 10240]
  - la_z_out                       (AC-15)  [B, S, 6144]
  - gated_delta_intermediates      (AC-35)  UNKNOWN
  - la_output[L]                   (AC-16)  [B, S, 5120]
```

**Classification:** VERIFIED FACT (dataflow structure from SET3 §6.1–§6.4). CONDITIONAL on exact kernel fusion (UK-003) and exact algorithm (UK-001).

### 13.5 Cross-Layer Non-Overlap (Structural)

Activations from non-adjacent layers can be released before later layers begin, subject to:

- Sequential execution within the language stack (SET3 §6.5 establishes sequential layer processing)
- Forward-pass activation retention (T4.3 scope is forward-pass activations only; backward-pass retention is downstream SET6 concern)
- **Classification:** VERIFIED FACT for forward-pass structure; CONDITIONAL on execution schedule (UK-003 implementation detail)

### 13.6 Conditional Lifetimes

| AC ID | Condition | Evidence |
|---|---|---|
| AC-25, AC-26, AC-27 | Vision encoder actively invoked | UK-007 — fusion mechanism UNKNOWN |
| AC-28, AC-29 | MTP actively executed | UK-008 — runtime execution UNKNOWN |
| AC-30 | Precomputed vs per-token RoPE | UK-003 — implementation detail |
| AC-33 | Materialized vs implicit masking | UK-003 — implementation detail |

---

## 14. Buffer Overlap and Reuse Analysis

This section identifies structurally derivable overlap and reuse opportunities. All claims below are CONDITIONAL on unresolved implementation details (UK-003, UK-004) unless explicitly marked VERIFIED FACT.

### 14.1 Structural Reuse Opportunities

1. **Layer input = layer output buffer (sequential overwrite):** VERIFIED FACT (structure). Layer L's `layer_output[L]` structurally becomes `layer_input[L+1]`. Whether these are the same buffer or separate buffers is UNKNOWN (UK-003). This is a structural necessity (sequential execution, SET3 §6.5).

2. **RMSNorm pre-output overwrites layer input:** CONDITIONAL MODEL. RMSNorm normalizes its input and writes output. If in-place or buffer-reuse, the output buffer may overwrite the input. Whether this is done is UNKNOWN (UK-003).

3. **Token mixer output overwrites residual buffer:** CONDITIONAL MODEL. The token mixer output is added to the residual. The sum may be written in-place into the residual buffer. Whether this is done is UNKNOWN (UK-003, UK-006).

4. **MLP down-proj output overwrites token mixer output buffer:** CONDITIONAL MODEL. After the MLP, its output is added to the residual. The MLP down-proj output may reuse the token mixer output buffer. Whether this is done is UNKNOWN (UK-003, UK-006).

5. **Full-attention QK product reuses Q or K norm buffers:** CONDITIONAL MODEL. The QK product consumes Q and K norm outputs. Whether intermediate buffers are allocated or fused is UNKNOWN (UK-003).

### 14.2 Cross-Layer Overlap

Activations from layer L and layer L+2 are structurally non-overlapping under sequential execution (SET3 §6.5). This enables:

- **Buffer pool recycling across non-adjacent layers:** CONDITIONAL MODEL. A workspace allocator could recycle the `rmsnorm_pre_out`, `rmsnorm_post_out`, and token-mixer intermediate buffers between layers L and L+2 if execution is strictly sequential and no recomputation is needed. Whether such a pool exists = UNKNOWN (UK-003).

### 14.3 Persistent vs Transient Classification

| AC ID | Persistence | Rationale |
|---|---|---|
| AC-01 | TRANSIENT | Single use, per forward pass |
| AC-02 | TRANSIENT | One per layer, reusable across layers |
| AC-03 | TRANSIENT | One per layer, usable after RMSNorm |
| AC-04 | TRANSIENT | One per layer, usable after RMSNorm |
| AC-05 (Q/K/V) | TRANSIENT | Consumed by attention/KV cache store |
| AC-05 (Q) | TRANSIENT | Released after QK product |
| AC-06 | TRANSIENT | Consumed by RoPE + QK product |
| AC-07 | ACTIVATION BOUNDARY | KV cache store boundary; state persistence = T4.4 UNKNOWN (UK-011) |
| AC-08 | WORKSPACE/TRANSIENT | QK product, may be fused |
| AC-09 | WORKSPACE/TRANSIENT | Softmax output, may be fused |
| AC-10 | WORKSPACE/TRANSIENT | Weighted sum, may be fused |
| AC-11 | TRANSIENT | Gate output, may be fused |
| AC-12 | TRANSIENT | Layer output, reusable |
| AC-13 | TRANSIENT | Residual, reusable |
| AC-14 | TRANSIENT | Per-layer, consumed by delta rule |
| AC-15 | TRANSIENT | Per-layer, consumed by delta rule |
| AC-21 | TRANSIENT | MLP output, reusable |
| AC-22 | TRANSIENT | Final norm output (single) |
| AC-23 | OUTPUT | Logits (request scope) |
| AC-25 | INPUT | Vision input (conditional) |
| AC-26 | TRANSIENT | Vision block activations (conditional) |
| AC-27 | TRANSIENT | VL features (conditional) |
| AC-28 | TRANSIENT | MTP (conditional — only if activated) |
| AC-30 | STATIC/CONDITIONAL | Precomputed if strategy dictates (UK-003) |
| AC-31 | INPUT | Position IDs (request scope) |
| AC-32 | INPUT | Input IDs (request scope) |
| AC-33 | WORKSPACE/TRANSIENT | May be implicit (UK-003) |
| AC-34 | OUTPUT | Final hidden states (request scope) |
| AC-35 | WORKSPACE/TRANSIENT | Linear-attention intermediate (UK-001) |
| AC-36 | ACTIVATION BOUNDARY | Conv state boundary; state persistence = T4.5 UNKNOWN (UK-012) |

Note: Activations classified as STATEFUL are excluded from the transient peak activation-memory formulas in §16. AC-07 and AC-36 are retained as activation-boundary observations (not state objects); the full-attention KV-cache state (T4.4) and linear-attention recurrence state (T4.5) remain UNKNOWN and are excluded from activation-memory formulas.

---

## 15. Parameterized Activation-Memory Formulas

All formulas assume BF16 computation (E = 2) unless parameterized differently. B and S are UNKNOWN (UK-009). Runtime computation dtype is UNKNOWN (UK-002, UK-004).

### 15.1 Language-Only Activation Memory (Per-Layer, One Layer at a Time)

Under the structural assumption of sequential layer execution with no recomputation, the simultaneously-live activations within one language layer L are:

**Full-attention layer L (16 of 64 layers):**

```
A_fa_layer(B, S, E) =
  B·S·5120·E          (AC-02: layer_input)
+ B·S·5120·E          (AC-03: rmsnorm_pre_out — may overlap with AC-02)
+ B·S·12288·E         (AC-05: Q projection)
+ B·S·1024·E          (AC-05: K projection)
+ B·S·1024·E          (AC-05: V projection)
+ B·S·256·E           (AC-06: Q norm)
+ B·S·256·E           (AC-06: K norm)
+ B·24·S²·E_qk        (AC-08: QK product — E_qk = UNKNOWN, UK-002)
+ B·24·S²·E_sm        (AC-09: softmax — E_sm = UNKNOWN, UK-004)
+ B·24·S·256·E_attn   (AC-10: weighted sum — E_attn = UNKNOWN, UK-004)
+ A_mlp(B, S, E)      (AC-17–AC-21: MLP intermediates, see §15.3)
+ B·S·5120·E          (AC-12: fa_output / AC-13: residual / AC-24: layer_output)

Simplified (structural upper bound, assuming no fusion, E=2, E_qk=E_sm=E_attn=2):
A_fa_layer_max(B, S) = B·S·(5120 + 5120 + 12288 + 1024 + 1024 + 256 + 256 + 24·256)·E + B·24·S²·E_qk + B·24·S²·E_sm + B·24·S·256·E_attn + A_mlp_max(B, S) + B·S·5120·E

Applying E=2, E_qk=2, E_sm=2, E_attn=2:
= B·S·(5120 + 5120 + 12288 + 1024 + 1024 + 256 + 256 + 24·256 + 5120)·2 + B·24·S²·2 + B·24·S²·2
= B·S·(31224 + 5120)·2 + B·24·S²·(2 + 2)
= B·S·36344·2 + B·S²·96
= B·S·72688 + B·S²·96 + A_mlp_max(B, S)

Dimensional audit:
- B·S terms (all [B, S, D] or [B, 24, S, 256] → O(B·S)): 5120+5120+12288+1024+1024+256+256+6144+5120 = 36344, ×E=2 → 72688
- B·S² terms (all [B, 24, S, S] → O(B·S²)): 24+24 = 48, ×E_qk=E_sm=2 → 96
- Weighted sum [B, 24, S, 256] is O(B·S), not O(B·S²): correctly classified above
```

**Linear-attention layer L (48 of 64 layers):**

```
A_la_layer(B, S, E) =
  B·S·5120·E          (AC-02: layer_input)
+ B·S·5120·E          (AC-03: rmsnorm_pre_out — may overlap)
+ B·S·10240·E         (AC-14: la_qkv_out)
+ B·S·6144·E          (AC-15: la_z_out)
+ A_gdelta(B, S, E)   (AC-35: gated-delta intermediates — UNKNOWN, UK-001)
+ A_mlp(B, S, E)      (AC-17–AC-21: MLP intermediates)
+ B·S·5120·E          (AC-16: la_output / AC-13: residual / AC-24: layer_output)

Simplified (structural upper bound, assuming no fusion, E=2, A_gdelta parameterized):
A_la_layer_max(B, S) = B·S·(5120 + 5120 + 10240 + 6144 + 5120)·E + A_gdelta(B, S) + A_mlp_max(B, S)

Applying E=2:
= B·S·(5120 + 5120 + 10240 + 6144 + 5120)·2 + A_gdelta(B, S) + A_mlp_max(B, S)
= B·S·26624·2 + A_gdelta(B, S) + A_mlp_max(B, S)
= B·S·53248 + A_gdelta(B, S) + A_mlp_max(B, S)

Dimensional audit:
- B·S terms: 5120+5120+10240+6144+5120 = 26624, ×E=2 → 53248
- A_gdelta: parameterized per UK-001 (shapes UNKNOWN)
- A_mlp_max: included (see §15.3)
```

### 15.3 MLP Activation Memory (Per-Layer)

```
A_mlp(B, S, E) =
  B·S·5120·E          (AC-04: rmsnorm_post_out)
+ B·S·17408·E         (AC-17: gate_proj output)
+ B·S·17408·E         (AC-18: up_proj output — may overlap with AC-17 post-SiLU)
+ B·S·17408·E_silu    (AC-19: SiLU gate — E_silu = UNKNOWN, UK-002)
+ B·S·17408·E         (AC-20: gate × up product — may overlap with AC-17/18)
+ B·S·5120·E          (AC-21: down_proj output)

Simplified (structural upper bound, no fusion, E=2, E_silu=2):
A_mlp_max(B, S) = B·S·(5120 + 17408 + 17408 + 17408 + 17408 + 5120)·E

Applying E=2 (and E_silu=2 for the SiLU gate):
= B·S·(5120 + 17408 + 17408 + 17408 + 17408 + 5120)·2
= B·S·72432·2
= B·S·144864

Dimensional audit:
- All 6 terms are [B, S, 17408] or [B, S, 5120] → O(B·S)
- Sum: 5120+17408+17408+17408+17408+5120 = 72432, ×E=2 → 144864
- SiLU gate (AC-19) uses E_silu=2 (same as E, so E=2 applies consistently)
```

### 15.4 Language Stack: Embed + Final Phase

```
A_embed_final(B, S, E) =
  B·S·5120·E          (AC-01: embeddings — same shape, consumed once)
+ B·S·5120·E          (AC-22: final_norm_out — single, not per-layer)
+ B·S·248320·E_logits   (AC-23: logits — E_logits = UNKNOWN, UK-004)

Simplified (structural upper bound, E=2 for norm, E_logits=2):
A_embed_final_max(B, S) = B·S·(5120 + 5120)·E + B·S·248320·E_logits

Applying E=2, E_logits=2:
= B·S·(5120 + 5120)·2 + B·S·248320·2
= B·S·10240·2 + B·S·496640
= B·S·20480 + B·S·496640
= B·S·517120

Dimensional audit:
- AC-01 embeddings [B, S, 5120]: 5120·E = 5120·2 = 10240
- AC-22 final_norm [B, S, 5120]: 5120·E = 5120·2 = 10240
- AC-23 logits [B, S, 248320]: 248320·E_logits = 248320·2 = 496640
- Sum: 10240 + 10240 + 496640 = 517120
```

### 15.5 Vision Activations (Conditional)

```
A_vision(B, S_v, E) =
  A_pixel_values(B, num_frames, 3, H_vid, W_vid)    (AC-25 — UNKNOWN H_vid, W_vid)
+ Σ_{Lv=0}^{26} A_vision_block(Lv)                    (AC-26 — 27 blocks, UNKNOWN shapes)
+ N·5120·E                                             (AC-27 — N vision tokens = UNKNOWN)

Classification: CONDITIONAL MODEL — depends on:
  - Whether vision encoder is invoked (UK-007)
  - Runtime image resolution H_vid, W_vid (UK-009)
  - Number of vision tokens N (UNKNOWN, depends on merge ratio)
  - Exact vision block activation shapes (UNKNOWN)
  - Runtime state dtype (UK-002)
```

### 15.6 MTP Activations (Conditional)

```
A_mtp(B, S, E) = UNKNOWN — depends on:
  - Whether MTP is actively executed (UK-008)
  - If executed, intermediate shapes and count (UNKNOWN)
  - Runtime dtype (UK-002)

Classification: CONDITIONAL MODEL — only non-zero if MTP is activated
```

### 15.7 RoPE Buffer (Conditional)

```
A_rope(S, E_rope) =
  If precomputed for max_position_embeddings: 262144 × 64 × 2 × E_rope
  If precomputed per sequence:    2 × S × 64 × E_rope  (cos + sin)
  If computed per-token:         0 (fused)

Classification: CONDITIONAL MODEL — allocation strategy = UNKNOWN (UK-003)
```

### 15.8 Input/Output Buffers

```
A_io(B, S) =
  B·S·E_ids        (AC-31: input_ids — E_ids = UNKNOWN)
+ B·S·E_pos        (AC-32: position_ids — E_pos = UNKNOWN)
+ B·S²·E_mask      (AC-33: causal_mask — E_mask = UNKNOWN, CONDITIONAL materialized)
+ B·S·5120·E       (AC-34: hidden_states_out)

Classification: VERIFIED FACT (existence) / CONDITIONAL (exact representation, UK-003)
```

---

## 16. Bounded and Conditional Cases

### 16.1 Fusion Uncertainty (UK-003)

Whether kernel fusion occurs determines whether intermediate buffers (AC-05 Q/K/V, AC-08 QK product, AC-09 softmax, AC-10 weighted sum, AC-14 QKV, AC-15 Z, AC-17–AC-21) are materialized or fused into preceding/following kernels. Each can be either:

- **Materialized (no fusion):** buffer is allocated, consuming the full byte size
- **Fused (in-place or streaming):** buffer is not materialized, contributing 0 bytes to peak

This is not resolved from available evidence. The parameterized formulas in §15 present the **materialized (no-fusion)** case as a **structural upper bound** (worst-case materialization bound). The **fully-fused** case is a lower bound on memory (a bound on memory savings). Actual runtime behavior is UNKNOWN (UK-003).

**Classification:** CONDITIONAL MODEL (bounded between materialized and fully-fused cases).

### 16.2 Runtime Dtype Uncertainty (UK-002, UK-004)

- BF16 activation computation (E=2): VERIFIED FACT from config.
- Runtime computation dtype for specific intermediates (QK product, softmax, attention weights, SiLU gate, logits): UNKNOWN (UK-002, UK-004). May be FP32 (E=4) for numerical stability.

**Classification:** UNKNOWN (runtime dtype for specific intermediates).

### 16.3 Residual Topology Uncertainty (UK-005, UK-006)

- Whether RMSNorm is pre-norm or post-norm: UNKNOWN (UK-005).
- Exact residual connection structure (which residuals exist): UNKNOWN (UK-006).
- Whether residuals are accumulated in-place or via separate buffers: UNKNOWN (UK-006).

**Classification:** UNKNOWN (UK-005, UK-006). These affect which activations coexist but do not change the per-activation shape model.

### 16.4 Linear-Attention Algorithm Uncertainty (UK-001)

- Exact linear-attention algorithm: UNKNOWN (UK-001).
- Gated-delta intermediate shapes: UNKNOWN (UK-001).
- Convolution state buffer shape: UNKNOWN (UK-012).

**Classification:** CONDITIONAL MODEL (shapes are UNKNOWN-dependent); UNKNOWN (algorithm details).

### 16.5 Runtime Dimensions Uncertainty (UK-009)

- Batch size B: UNKNOWN (UK-009).
- Sequence length S: UNKNOWN (UK-009).
- Vision resolution H, W: UNKNOWN (UK-009).

All formulas are parameterized by B and S. No single numeric peak is asserted.

**Classification:** UNKNOWN (UK-009). Formulas remain parameterized.

### 16.6 Conditional Execution of Vision and MTP

- Vision encoder invocation: UNKNOWN (UK-007).
- MTP execution: UNKNOWN (UK-008).

**Classification:** UNKNOWN (UK-007, UK-008). Activation memory for vision and MTP is CONDITIONAL — only present if the respective subsystem is active.

---

## 17. UNKNOWN / CONDITIONAL Register

All SET3 UNKNOWNs carried forward, preserved without silent resolution:

| UK | Description | T4.3 Impact | Classification |
|---|---|---|---|
| UK-001 | Exact linear-attention algorithm | AC-35 shapes (boundary), AC-36 state (T4.5) | UNKNOWN → CONDITIONAL MODEL (T4.5 boundary) |
| UK-002 | Runtime computation dtype for intermediates | AC-19, AC-23, AC-08, AC-09, AC-10 dtypes | UNKNOWN → CONDITIONAL MODEL |
| UK-003 | Whether kernels are fused | Materialization of AC-05, AC-06, AC-08, AC-09, AC-10, AC-11, AC-14, AC-15, AC-17–AC-20, AC-33 | UNKNOWN → CONDITIONAL MODEL |
| UK-004 | Runtime computation dtype (general) | Affects E values throughout | UNKNOWN → parameterized |
| UK-005 | Whether RMSNorm is pre- or post-norm (placement) | AC-03, AC-04, AC-22 placement | UNKNOWN → CONDITIONAL |
| UK-006 | Exact residual connection structure | AC-13, AC-11 gate, AC-12, AC-16 topology | UNKNOWN → CONDITIONAL |
| UK-007 | Exact vision-to-language fusion mechanism | AC-27, AC-25–27 activation presence | UNKNOWN → CONDITIONAL |
| UK-008 | MTP active runtime execution | AC-28, AC-29 activation presence | UNKNOWN → CONDITIONAL |
| UK-009 | Runtime batch size B and sequence length S | All parameterized formulas | UNKNOWN → parameterized |
| UK-010 | Whether attention scaling is applied and how | AC-08, AC-09 | UNKNOWN → parameterized |
| UK-011 | KV cache allocation and paging strategy | AC-07 allocation | UNKNOWN → (T4.4 domain) |
| UK-012 | Exact runtime convolution state allocation | AC-36 state (T4.5) | UNKNOWN → CONDITIONAL (T4.5 boundary) |

### 17.1 Additional Unknowns Not in SET3 Register (Discovered During T4.3)

| ID | Description | T4.3 Impact | Classification |
|---|---|---|---|
| UQ-006 | Exact semantics of MRoPE sections in every execution path | AC-30 scope, AC-31 layout | UNKNOWN → CONDITIONAL |

> **Note:** UQ-006 is documented in SET0-04 §17 but not formally numbered in the SET3 UK register. It is surfaced here as a CONDITIONAL dependency. No SET3 UNKNOWN has been silently resolved.

---

## 18. Peak Activation Memory (Within T4.3 Scope)

### 18.1 Scope Constraint

T4.3 produces the **activation lifetime and shape model only**. The final SET4 peak runtime-memory model (including weight memory, state memory, workspace memory, and their combined peak) belongs to T4.7 and is **not** produced here.

### 18.2 Bounded Peak Activation Memory (Language Stack Only)

Under the structural assumptions of:
- Sequential layer execution (SET3 §6.5)
- No fusion (UK-003 worst case for memory)
- BF16 computation (E=2) for materialized intermediates
- Ignoring cross-layer overlap for the conservative bound

The peak simultaneously-live activation memory for the language stack is determined by the **phase model**: the execution decomposes into temporally distinct phases (T_INIT, T_EMBED, T_LAYER, T_FINAL, T_VISION, T_MTP) per §13.2. Under sequential layer execution, per-layer phases are mutually exclusive in time. The embed phase and final phase are also temporally distinct from layer phases. Therefore the peak is the maximum across phases, not the sum:

```
A_peak_language(B, S, E) = max(
  A_fa_layer_max(B, S, E),      (full-attention layer phase — worst layer)
  A_la_layer_max(B, S, E),      (linear-attention layer phase)
  A_embed_final_max(B, S, E)    (embed + final + logits phase — distinct phases)
)
```

The full-attention layer has the larger per-layer activation footprint because of the 24×S² attention matrices (E_qk, E_sm):

A_fa_layer_max(B, S, E=2) = B·S·72688 + B·S²·96 + A_mlp_max(B, S, E=2)
                         = B·S·72688 + B·S²·96 + B·S·144864
                         = B·S·217552 + B·S²·96

A_la_layer_max(B, S, E=2) = B·S·53248 + A_gdelta(B, S) + A_mlp_max(B, S, E=2)
                         = B·S·53248 + B·S·144864 + A_gdelta(B, S)
                         = B·S·198112 + A_gdelta(B, S)     (A_gdelta = UNKNOWN per UK-001)

A_embed_final_max(B, S, E=2) = B·S·517120

**This is a STRUCTURAL UPPER BOUND (no-fusion assumption).** It does not account for:
- Kernel fusion (UK-003) — could reduce peak
- Buffer reuse / in-place operations (UK-003, UK-006) — could reduce peak
- Cross-layer buffer recycling (§14.2) — could reduce peak
- Backward-pass activation retention — NOT in T4.3 scope
- The relative magnitudes of A_mlp_max and A_embed_final_max depend on B, S
  which are UNKNOWN (UK-009); the exact max cannot be resolved without B, S

**Classification:** CONDITIONAL MODEL (bounded upper bound, parameterized by B, S).

### 18.3 What Is NOT Included

- **Weight memory** — modeled in T4.2 (not this document)
- **Full-attention KV-cache state** — modeled in T4.4 (not this document)
- **Linear-attention state (conv + recurrence)** — modeled in T4.5 (not this document)
- **Workspace memory / allocator overhead** — modeled in T4.6 (not this document)
- **Final combined peak runtime memory** — modeled in T4.7 (not this document)
- **Backward-pass activation retention** — SET6 domain

### 18.4 Summary

The peak activation memory for the language forward-pass, **parameterized** and **bounded above** by the no-fusion assumption, is:

```
A_peak_language_max(B, S, E=2) = max(
  B·S·217552 + B·S²·96,              // A_fa_layer_max (full-attention layer phase)
  B·S·198112 + A_gdelta(B, S),       // A_la_layer_max (linear-attention layer phase, A_gdelta = UNKNOWN)
  B·S·517120                        // A_embed_final_max (embed + final + logits phase)
)
```

This formula is conditional on:
- B, S being known (UK-009)
- Runtime dtype being BF16 (UK-002, UK-004)
- No kernel fusion (UK-003)
- No backward pass (SET6 domain)
- No cross-layer buffer reuse (UK-003)
- A_gdelta (linear-attention gated-delta intermediates) = UNKNOWN (UK-001)

A fully-fused, maximally-reused execution path would have lower peak activation memory, but the exact reduction is UNKNOWN (UK-003).

---

## 19. Downstream Dependency Mapping

T4.3 explicitly identifies what T4.4, T4.6, and T4.7 will consume:

### 19.1 T4.4 Consumption (Full-Attention KV-Cache State Model)

T4.4 will consume:

- **AC-07** (`kv_store_boundary[L]`): Activation-side boundary observation that K/V projection outputs (AC-05) feed a downstream KV-cache state transition. Shape `[B, 4, S, 256]` is NOT established in T4.3; it is T4.4 UNKNOWN (UK-011). T4.4 extends this from the activation boundary to the full KV cache state across all 16 full-attention layers × 2 (K+V) × all tokens.
- **Provenance:** T4.1 RM-012 (K cache), RM-013 (V cache); SET0-04 §3.2; SET3-OC-3 §3.3
- **Parameterization inherited:** `B` (UK-009), `S` (UK-009), state dtype `E_state` (UK-004)
- **T4.4-specific resolution needed:** UK-011 (KV cache allocation and paging strategy)

### 19.2 T4.6 Consumption (Workspace Memory Model)

T4.6 will consume:

- **AC-08** (`qk_product[L]`): Shape `[B, 24, S, S]`, used as workspace buffer
- **AC-09** (`attention_weights[L]`): Shape `[B, 24, S, S]`, workspace buffer
- **AC-10** (`attn_weighted_sum[L]`): Shape `[B, 24, S, 256]`, workspace buffer
- **AC-33** (`causal_mask`): Whether materialized as workspace or implicit (UK-003)
- **AC-30** (`rope_cos/sin`): Whether precomputed (workspace/static) or per-token (fused) (UK-003)
- **§14.1 reuse candidates:** AC-03, AC-04, AC-12, AC-16, AC-21 as workspace-pooled buffers
- **Parameterization inherited:** `B` (UK-009), `S` (UK-009), workspace dtype (UK-004)

### 19.3 T4.7 Consumption (Final Peak Runtime Memory Model)

T4.7 will consume:

- **§15 formulas** (A_fa_layer, A_la_layer, A_mlp, A_embed_final): For computing the language-stack activation peak
- **§16 conditional cases** (fusion bounds, dtype bounds): For bounding the activation contribution under UK-002, UK-003, UK-004
- **§18 peak formula** (A_peak_language_max): As the activation-memory component of the combined peak
- **AC-07, AC-36**: As the state-memory component — ACTIVATION BOUNDARY observations only (T4.4, T4.5 will model the actual state)
- **T4.2 weight-residency model**: As the weight-memory component
- T4.4 (KV-cache state), T4.5 (linear-attention state), T4.6 (workspace) contributions
- **All UK/UNKNOWN markers**: Must be resolved or parameterized in the final model

### 19.4 T4.5 (Not Requested, Listed for Completeness)

T4.5 (linear-attention state) will consume:
- **AC-35** (`gated_delta_intermediates`): Activation boundary observation — exact shapes UNKNOWN (UK-001, T4.5 domain)
- **AC-36** (`la_conv_state_boundary`): Activation boundary observation — exact state shape UNKNOWN (UK-012, T4.5 domain)

---

## 20. Do-Not-Run Compliance

The following actions were NOT performed during T4.3:

- ✗ No inference engine implemented
- ✗ No memory allocator implemented
- ✗ No runtime memory management implemented
- ✗ No kernels designed or optimized
- ✗ No throughput or latency benchmarked
- ✗ No workload placement performed
- ✗ No scheduling performed
- ✗ No streaming or paging implemented
- ✗ No full-attention KV-cache state modeled in detail beyond the activation boundary (AC-07)
- ✗ No linear-attention recurrent/conv state modeled beyond the activation boundary (AC-35, AC-36)
- ✗ No final SET4 peak runtime-memory model produced (§18 presents only the bounded activation-only peak within T4.3 scope)
- ✗ No SET4-T4.4 work performed
- ✗ No SET4-T4.5 work performed
- ✗ No SET4-T4.6 work performed
- ✗ No SET4-T4.7 work performed
- ✗ No SET4-T4.8 work performed
- ✗ No SET4-T4.9 work performed
- ✗ No SET5 begun
- ✗ No SET3 UNKNOWN silently resolved — all UK-001 through UK-013 preserved
- ✗ No SET1, SET2, or SET3 historical evidence modified
- ✗ No unrelated repository files modified

---

## 21. T4.3 Completeness Assessment

### 21.1 Required Activation Categories Addressed

| Required Category | Section(s) | Status |
|---|---|---|
| 1. Input and embedding activations | §3.1 AC-01, §10 AC-32 | ✅ Addressed |
| 2. Vision-to-language interfaces | §7 AC-25, AC-26, AC-27 | ✅ Addressed (conditional) |
| 3. Decoder-layer input/output activations | §3.2 AC-02, §6 AC-24, §6 AC-34 | ✅ Addressed |
| 4. Full-attention intermediate activations | §3.5–§3.12 AC-05, AC-06, AC-08–AC-12 | ✅ Addressed |
| 5. Linear-attention intermediate activations | §4 AC-14, AC-15, AC-16, AC-35, AC-36 (boundary) | ✅ Addressed (AC-36 down-scoped to boundary) |
| 6. MLP intermediate activations (gated-SiLU) | §5 AC-17–AC-21 | ✅ Addressed |
| 7. RMSNorm and normalization intermediates | §3.3 AC-03, §3.4 AC-04, §6 AC-22 | ✅ Addressed |
| 8. Residual-path activation lifetimes | §3.13 AC-13, §3.12 AC-12 | ✅ Addressed (UK-006 marked UNKNOWN) |
| 9. LM-head and output activations | §6 AC-22, AC-23 | ✅ Addressed |
| 10. MTP-related activations | §8 AC-28, AC-29 | ✅ Addressed (conditional) |
| 11. Persistent vs transient categories | §14.3 | ✅ Addressed |
| 12. Activation lifetime intervals | §13 | ✅ Addressed |
| 13. Buffer overlap and reuse | §14 | ✅ Addressed |
| 14. Parameterized formulas | §15 | ✅ Addressed |
| 15. Peak activation-memory within scope | §18 | ✅ Addressed (bounded, conditional) |
| 16. DOWNSTREAM dependencies (T4.4, T4.6, T4.7) | §19 | ✅ Addressed |

### 21.2 Acceptance Criteria Verification

| Criterion | Status | Evidence |
|---|---|---|
| Activation inventory complete within scope | ✅ | §3, §4, §5, §6, §7, §8, §10, §11 — 36 activation objects |
| All material activation shapes traced to evidence | ✅ | §12 shape model table with provenance per AC |
| Activation lifetimes explicitly modeled | ✅ | §13 lifetime model |
| Coexistence and reuse addressed | ✅ | §13.4, §14 |
| Parameterized formulas exist | ✅ | §15 |
| Bounded/conditional cases where runtime UNKNOWN | ✅ | §16 |
| Relevant UNKNOWNs remain classified | ✅ | §17 register (UK-001 through UK-013, UQ-006) |
| Activation memory separate from state/workspace/weight | ✅ | §1.4, §14.3, §18.3 |
| No unsupported runtime behavior asserted | ✅ | All conditional claims marked CONDITIONAL MODEL or UNKNOWN |
| Document persisted | ✅ | docs/set-4/03-activation-lifetime-model.md |
| (Pushed/remotely verified) | Pending | Next step |

### 21.3 UNRESOLVED Items (Carried to Downstream)

| Item | Downstream T | Reason |
|---|---|---|
| UK-001 (linear-attention algorithm) | T4.5 | Affects AC-35 shapes (boundary), AC-36 state |
| UK-002 (intermediate dtype) | T4.7 | Affects peak byte calculations |
| UK-003 (fusion strategy) | T4.6, T4.7 | Affects buffer overlap and peak |
| UK-004 (runtime computation dtype) | T4.7 | Affects all E values |
| UK-005 (pre/post-norm) | T4.3 boundary / T4.4 | Affects coexistence |
| UK-006 (residual topology) | T4.3 boundary | Affects coexistence |
| UK-007 (vision fusion) | T4.3 boundary | Affects AC-25–27 presence |
| UK-008 (MTP execution) | T4.3 boundary | Affects AC-28–29 presence |
| UK-009 (B, S) | T4.7 | All formulas parameterized |
| UK-010 (attention scaling) | T4.7 | Affects QK product magnitude |
| UK-011 (KV cache allocation) | T4.4 | Affects AC-07 boundary observation |
| UK-012 (conv state allocation) | T4.5 | Affects AC-36 boundary observation |
| UQ-006 (MRoPE semantics) | T4.7 | Affects AC-30, AC-31 |

---

## Appendix A: Activation Object Index (AC-01 through AC-36)

| AC ID | Object Name | Type | Classification |
|---|---|---|---|
| AC-01 | `embeddings` | Transient | VERIFIED FACT |
| AC-02 | `layer_input[L]` | Transient | VERIFIED FACT |
| AC-03 | `rmsnorm_pre_out[L]` | Transient | VERIFIED FACT |
| AC-04 | `rmsnorm_post_out[L]` | Transient | VERIFIED FACT |
| AC-05 | `fa_q_out`, `fa_k_out`, `fa_v_out` | Transient | VERIFIED FACT |
| AC-06 | `fa_q_norm_out`, `fa_k_norm_out` | Transient | VERIFIED FACT |
| AC-07 | `kv_store_boundary[L]` | Activation Boundary | ACTIVATION BOUNDARY (state: T4.4 UNKNOWN) |
| AC-08 | `qk_product[L]` | Workspace | CONDITIONAL MODEL |
| AC-09 | `attention_weights[L]` | Workspace | CONDITIONAL MODEL |
| AC-10 | `attn_weighted_sum[L]` | Workspace | CONDITIONAL MODEL |
| AC-11 | `attn_output_gate_out[L]` | Transient | UNKNOWN |
| AC-12 | `fa_output[L]` | Transient | VERIFIED FACT |
| AC-13 | `residual[L]` | Transient | VERIFIED FACT |
| AC-14 | `la_qkv_out[L]` | Transient | VERIFIED FACT |
| AC-15 | `la_z_out[L]` | Transient | VERIFIED FACT |
| AC-16 | `la_output[L]` | Transient | VERIFIED FACT |
| AC-17 | `mlp_gate_out[L]` | Transient | VERIFIED FACT |
| AC-18 | `mlp_up_out[L]` | Transient | VERIFIED FACT |
| AC-19 | `mlp_silu_gate[L]` | Transient | VERIFIED FACT / UNKNOWN (dtype) |
| AC-20 | `mlp_gate_up_product[L]` | Transient | VERIFIED FACT |
| AC-21 | `mlp_down_out[L]` | Transient | VERIFIED FACT |
| AC-22 | `final_norm_out` | Transient | VERIFIED FACT |
| AC-23 | `logits` | Output | VERIFIED FACT / UNKNOWN (dtype) |
| AC-24 | `layer_output[L]` | Transient | VERIFIED FACT |
| AC-25 | `pixel_values` | Input | VERIFIED FACT (structure) |
| AC-26 | `vision_layer_activations[L_v]` | Transient | VERIFIED FACT (config) / UNKNOWN |
| AC-27 | `vision_language_features` | Transient | VERIFIED FACT (width) / UNKNOWN |
| AC-28 | `mtp_activations` | Transient | VERIFIED FACT (weights) / UNKNOWN |
| AC-29 | `mtp_pre_fc_norm_*` | Transient | VERIFIED FACT / UNKNOWN (runtime) |
| AC-30 | `rope_cos/sin` | Static/Conditional | VERIFIED FACT (config) / UNKNOWN |
| AC-31 | `position_ids` | Input | VERIFIED FACT (concept) / UNKNOWN |
| AC-32 | `input_ids` | Input | VERIFIED FACT (concept) / UNKNOWN |
| AC-33 | `causal_mask` | Workspace | VERIFIED FACT / CONDITIONAL |
| AC-34 | `hidden_states_out` | Output | VERIFIED FACT |
| AC-35 | `gated_delta_intermediates[L]` | Workspace | CONDITIONAL MODEL |
| AC-36 | `la_conv_state_boundary[L]` | Activation Boundary | ACTIVATION BOUNDARY (state: T4.5 UNKNOWN) |

---

## Appendix B: Classification Legend

```text
VERIFIED FACT:
  Directly supported by raw SET1 tensor metadata, SET0 configuration fields,
  or established SET3 operator/dataflow evidence.

DOCUMENTED CAPABILITY:
  Sourced from authoritative external documentation; not promoted to VERIFIED FACT.

DERIVED FINDING:
  Arithmetic or logical combination of VERIFIED FACT evidence. Explicitly labeled.

CONDITIONAL MODEL:
  A memory property expressed as an explicit dependency on an unresolved
  runtime behavior (UK-001 through UK-013, UQ-006). Bounded, parameterized.

UNKNOWN:
  Runtime behavior or structural detail not established by available evidence.
  Treated as a boundary, not silently resolved.
```
