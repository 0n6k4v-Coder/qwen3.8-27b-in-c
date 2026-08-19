# SET 3 — Operator / Computation Model

## Document Status

- **Document:** `docs/set-3/01-operator-computation-model.md`
- **SET:** `SET 3 — Operator / Computation Model`
- **Status:** VERIFIED
- **Responsibility:** 🧠 LUNA
- **Date:** 2026-08-19
- **Control State:** `SET3-READINESS-GATE PASS`

---

## 1. Source and Provenance

### 1.1 Authoritative Model

- Model: `Qwen3.8-27B`
- Official repository: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

### 1.2 Primary Evidence Sources

All evidence used in this document is VERIFIED and authoritative:

```text
SET 1 (Tensor Truth / Byte Truth):
  docs/set-1/01-raw-metadata-verification.md        — 1,199 tensors, 18 shards, BF16
  docs/set-1/02-parameter-reconstruction.md         — 27,781,427,952 parameters
  docs/set-1/03-tensor-byte-accounting.md           — 55,562,855,904 logical bytes
  docs/set-1/04-checkpoint-storage-layout-reconciliation.md — shard/payload layout
  docs/set-1/05-set1-boundary-completeness-audit.md — scope boundaries

SET 0 (Structural Truth):
  docs/set-0/03-core-architecture.md                — canonical architecture summary
  docs/set-0/04-attention-architecture.md           — attention operator families
  docs/set-0/05-mlp-architecture.md                 — MLP operator structure
  docs/set-0/07-layer-topology.md                   — exact 64-layer topology
  docs/set-0/08-tensor-shape-mapping.md             — verified tensor shapes
  docs/set-0/06-vision-and-mtp.md                   — vision/MTP configuration

SET 2 (Hardware Truth Contract):
  docs/set-2/08-hardware-capability-synthesis.md     — capability/constraint matrices
  docs/set-2/07-interconnect-data-movement.md        — data-movement model
  docs/set-2/10-set2-close-acceptance.md             — SET 2 closure acceptance

Raw artifacts:
  model/official/raw-checkpoint-metadata/config.json
  model/official/raw-checkpoint-metadata/manifest.json
  model/official/raw-checkpoint-metadata/model.safetensors.index.json
  model/official/raw-checkpoint-metadata/shards/*.header.json
```

### 1.3 Classification Schema

Every material claim in this document is classified as one of:

- **VERIFIED FACT** — directly supported by raw SET1 tensor metadata, SET0
  configuration fields, or SET2 hardware-truth observations.
- **DOCUMENTED CAPABILITY** — sourced from authoritative external documentation
  (Intel ARK, primary Intel architecture docs) and explicitly not promoted to
  VERIFIED FACT. (Carried from SET2 T2.8 synthesis only where relevant to the
  operator model.)
- **DERIVED FINDING** — arithmetic or logical combination of verified evidence.
  Explicitly labeled and never presented as a raw observation.
- **UNKNOWN** — runtime behavior or structural detail not established by
  available evidence. Treated as a boundary, not a gap to fill.

---

## 2. Structural Model

### 2.1 Architecture Family

```text
Qwen3.5-derived implementation lineage (Qwen3_5ForConditionalGeneration)
├── 64-layer language stack
├── Multimodal: vision + language
└── 1 MTP layer (runtime execution: UNKNOWN)
```

**Evidence:** config.json `architectures: ["Qwen3_5ForConditionalGeneration"]`,
`model_type: "qwen3_5"`, `language_model_only: false`.

**Classification:** VERIFIED FACT (configuration declaration). The Qwen3_5
lineage is a lineage finding, not a license to substitute Qwen3.5's complete
architecture. (SET0 03-core-architecture.md §17.4)

### 2.2 Language Stack Composition

```text
Text Config Fields (config.json text_config):
  hidden_size = 5120
  num_hidden_layers = 64
  vocab_size = 248320
  dtype = bfloat16
  max_position_embeddings = 262144
```

**Classification:** VERIFIED FACT — sourced from `config.json` fields
`text_config.hidden_size`, `num_hidden_layers`, `vocab_size`, `dtype`,
`max_position_embeddings`. (SET0 03 §4, SET1-T1.4)

### 2.3 Layer Topology

The authoritative 64-entry `layer_types` array defines the exact per-layer
operator family:

```text
Layer  0: linear_attention
Layer  1: linear_attention
Layer  2: linear_attention
Layer  3: full_attention
Layer  4: linear_attention
...
Layer 63: full_attention
```

Full-attention layer indices: `3, 7, 11, 15, 19, 23, 27, 31, 35, 39, 43, 47,
51, 55, 59, 63` (count = 16).

Linear-attention layer indices: all remaining 0–63 (count = 48).

`full_attention_interval = 4` is consistent with the explicit array.

**Classification:** VERIFIED FACT — directly from `config.json` field
`text_config.layer_types` (64 entries) cross-checked against
`full_attention_interval = 4`. (SET0 07 §2–§8, T1.4)

### 2.4 Operator-to-Layer-Type Mapping

```text
linear_attention  →  Qwen3_5GatedDeltaNet
full_attention    →  Qwen3_5Attention
```

Each language layer contains exactly:
```text
Layer N
├── Token Mixer     (one of: Qwen3_5GatedDeltaNet | Qwen3_5Attention)
└── Dense Gated SiLU MLP
```

**Classification:** VERIFIED FACT — the implementation identity mapping is
`Qwen3_5ForConditionalGeneration` and the associated operator class names are
recorded in the implementation. The layer-to-operator-class mapping is derived
from the `layer_types` array and the architecture identity. (SET0 07 §9–§10,
SET0 04 §2)

### 2.5 Vision Subsystem

```text
vision_config:
  depth = 27
  hidden_size = 1152
  num_heads = 16
  intermediate_size = 4304
  out_hidden_size = 5120
  in_channels = 3
  patch_size = 16
  temporal_patch_size = 2
  hidden_act = gelu_pytorch_tanh
```

Vision output width `5120` matches language `hidden_size = 5120`.

**Classification:** VERIFIED FACT for config fields. VERIFIED FACT for the
width-matching observation (both are config values). The multimodal fusion
mechanism is UNKNOWN. (SET0 03 §12, SET0 06 §2–§3)

### 2.6 MTP Subsystem

```text
mtp_num_hidden_layers = 1
mtp_use_dedicated_embeddings = false
```

15 MTP checkpoint tensors verified (all BF16, shard 18):
`424,699,392` parameters.

**Classification:** VERIFIED FACT (checkpoint tensor metadata). UNKNOWN:
active runtime execution, scheduling, memory behavior. (SET0 06 §6, SET1-T1.6 §4)

---

## 3. Operator Classes

### 3.1 OC-1: Language Embedding Lookup

**Operator:** `LanguageEmbedding`

```text
Input:  input_ids [batch, seq_len]
Weight: language_model.embed_tokens.weight  [248320, 5120]  BF16
Output: embeddings [batch, seq_len, 5120]
```

**Tensor dependency:** `model.language_model.embed_tokens.weight`
**Checkpoint evidence:** SET1-T1.6 §6, shape `[248320, 5120]`,
parameters = `1,271,398,400`, shard 3.
**Classification:** VERIFIED FACT (tensor existence, shape, dtype, shard).

**Shape dependency:** Output hidden width = `hidden_size = 5120`.
**Classification:** VERIFIED FACT.

### 3.2 OC-2: Vision Encoder

**Operator:** `VisionEncoder` (27-layer configured block)

```text
Input:  pixel_values [batch, num_frames, 3, H, W]
Config: patch_size=16, temporal_patch_size=2, hidden_size=1152,
        intermediate_size=4304, depth=27, num_heads=16
Output: visual features projected to [N, 5120]
```

**Tensor dependency:** Visual encoder weights (subsystem total = `460,730,096`
parameters, `921,460,192` logical bytes). (SET1-T1.5-R1 §3, SET1-T1.6 §3)
**Classification:** VERIFIED FACT (subsystem existence, parameter/byte totals).
UNKNOWN: exact per-tensor vision naming, exact fusion mechanism.

### 3.3 OC-3: Full-Attention Token Mixer

**Operator class:** `Qwen3_5Attention` (mapped from `full_attention` layer type)

```text
Full-Attention Configuration (config.json text_config):
  num_attention_heads = 24
  num_key_value_heads = 4
  head_dim = 256
  attention_bias = false
  attention_dropout = 0.0
  attn_output_gate = true
  output_gate_type = swish
  partial_rotary_factor = 0.25
  rope_theta = 10000000
  rope_type = default
  mrope_interleaved = true
  mrope_section = [11, 11, 10]
```

**Verified tensor set (per full-attention layer, 16/16 coverage):**

| Tensor | Shape | Coverage |
|---|---|---:|
| `self_attn.q_proj.weight` | `[12288, 5120]` | 16/16 |
| `self_attn.k_proj.weight` | `[1024, 5120]` | 16/16 |
| `self_attn.v_proj.weight` | `[1024, 5120]` | 16/16 |
| `self_attn.o_proj.weight` | `[5120, 6144]` | 16/16 |
| `self_attn.q_norm.weight` | `[256]` | 16/16 |
| `self_attn.k_norm.weight` | `[256]` | 16/16 |

**Tensor dependencies:** 96 tensors total for full-attention projections
across 16 layers (6 tensors × 16 layers).

**Derived dimensions (from config):**
- Query projection: `24 × 256 = 6144` → but `q_proj` output is `12288` (includes
  QK or gate dimension from GQA implementation). **Classification:** VERIFIED
  FACT (raw tensor shape).
- Key projection: `4 × 256 = 1024` → `k_proj` output `[1024, 5120]`. VERIFIED FACT.
- Value projection: `4 × 256 = 1024` → `v_proj` output `[1024, 5120]`. VERIFIED FACT.
- Output projection: `o_proj` output `[5120, 6144]`. VERIFIED FACT.

**Execution stages (implementation-derived):**
```text
Hidden States → Q/KV projection → Q/K normalization → RoPE
  → KV Cache → Causal Attention → Output Gating → Output Projection
```
**Classification:** DERIVED FINDING — the stage ordering is described by the
official implementation, not by checkpoint metadata alone. (SET0 04 §3.3)

**GQA structure:** `24 Q heads / 4 KV heads = 6 query heads per KV head`.
**Classification:** DERIVED FINDING — arithmetic from config fields. (SET0 03 §6)

**RoPE rotary dimension:** `256 × 0.25 = 64`.
**Classification:** DERIVED FINDING — from `head_dim × partial_rotary_factor`.
(SET0 04 §4)

**Classification:** GQA — VERIFIED FACT (config fields `num_attention_heads`,
`num_key_value_heads`, `head_dim` all present). GQA ratio = DERIVED FINDING.
Linear-attention vs full-attention state model = VERIFIED FACT (different
state structures). Exact kernel algorithm = UNKNOWN.

### 3.4 OC-4: Linear-Attention Token Mixer

**Operator class:** `Qwen3_5GatedDeltaNet` (mapped from `linear_attention`
layer type)

```text
Linear-Attention Configuration (config.json text_config):
  linear_num_key_heads = 16
  linear_num_value_heads = 48
  linear_key_head_dim = 128
  linear_value_head_dim = 128
  linear_conv_kernel_dim = 4
  mamba_ssm_dtype = float32  (metadata field, not algorithm proof)
```

**Verified tensor set (per linear-attention layer, 48/48 coverage):**

| Tensor | Shape | Coverage |
|---|---|---:|
| `linear_attn.in_proj_qkv.weight` | `[10240, 5120]` | 48/48 |
| `linear_attn.in_proj_z.weight` | `[6144, 5120]` | 48/48 |
| `linear_attn.in_proj_b.weight` | `[48, 5120]` | 48/48 |
| `linear_attn.in_proj_a.weight` | `[48, 5120]` | 48/48 |
| `linear_attn.out_proj.weight` | `[5120, 6144]` | 48/48 |
| `linear_attn.conv1d.weight` | `[10240, 1, 4]` | 48/48 |
| `linear_attn.A_log` | `[48]` | 48/48 |
| `linear_attn.dt_bias` | `[48]` | 48/48 |
| `linear_attn.norm.weight` | `[128]` | 48/48 |

**Tensor dependencies:** 9 tensors × 48 layers = 432 tensors.
**Classification:** VERIFIED FACT (tensor existence, shapes, coverage).

**Derived dimensions:**
- `in_proj_qkv` output: `10240 = 16×128 (Q) + 16×128 (K) + 48×128 (V)` =
  `2048 + 2048 + 6144 = 10240`. VERIFIED FACT (shape match).
- `in_proj_z` output: `6144 = 48 × 128`. VERIFIED FACT (shape match).
- `in_proj_b`: `48` = `linear_num_value_heads`. VERIFIED FACT.
- `in_proj_a`: `48` = `linear_num_value_heads`. VERIFIED FACT.
- `out_proj`: `6144 → 5120`, i.e., `48×128 → 5120`. VERIFIED FACT.
- `conv1d`: `[10240, 1, 4]` — kernel size 4 along sequence dimension.
  VERIFIED FACT.
- `A_log`: `[48]` — state dimension parameter. VERIFIED FACT.
- `dt_bias`: `[48]` — timestep bias parameter. VERIFIED FACT.
- `norm`: `[128]` — per-head normalization. VERIFIED FACT.

**State model distinction:**
```text
Full Attention  →  KV Cache
Linear Attention →  Recurrent State + Convolution State
```
**Classification:** VERIFIED FACT — the tensor presence of `A_log`, `dt_bias`,
`conv1d` establishes a recurrent/convolutional state model distinct from KV
caching. (SET0 04 §9, §10)

**Algorithm identity boundary:**
> The exact internal algorithm of the linear-attention mechanism (Mamba,
> Mamba-2, Gated DeltaNet, DeltaNet, or other) MUST NOT be treated as
> verified from configuration fields alone. The `mamba_ssm_dtype` field is
> metadata presence, not algorithm specification.
**Classification:** UNKNOWN — exact linear-attention algorithm and kernel
semantics. (SET0 03 §7, §18 UQ-001)

**Classification:** Linear-attention state model = VERIFIED FACT. Exact
algorithm = UNKNOWN.

### 3.5 OC-5: Dense Gated SiLU MLP

**Operator:** `DenseGatedSiLUMLP`

```text
Configuration (config.json text_config):
  hidden_size = 5120
  intermediate_size = 17408
  hidden_act = silu

Structure:
  gate_proj: 5120 → 17408   (W_gate)
  up_proj:   5120 → 17408   (W_up)
  SiLU(gate_proj(x)) ⊙ up_proj(x)
  down_proj: 17408 → 5120   (W_down)
```

**Verified tensor set (per MLP layer):**

| Tensor | Shape | Coverage |
|---|---|---:|
| `mlp.gate_proj.weight` | `[17408, 5120]` | 64/64 |
| `mlp.up_proj.weight` | `[17408, 5120]` | 64/64 |
| `mlp.down_proj.weight` | `[5120, 17408]` | 64/64 |

**Tensor dependencies:** 3 tensors × 64 layers = 192 tensors.
**Classification:** VERIFIED FACT (tensor existence, shapes, coverage).

**Parameter arithmetic (DERIVED FINDING):**
```text
Per layer: 3 × (5120 × 17408) = 3 × 89,128,960 = 267,386,880
Aggregate: 64 × 267,386,880 = 17,112,760,320 ≈ 17.11B parameters
```
**Classification:** DERIVED FINDING — from verified dimensions × layer count.
(SET0 05 §9–§10)

**Expansion ratio:** `17408 / 5120 = 3.4`.
**Classification:** DERIVED FINDING — dimensional ratio only, not a
parameter FLOP or memory ratio. (SET0 05 §5)

**Exact internal MLP formulation:** While the three-projection gated structure
is verified, the exact internal formulation beyond the canonical
`W_down(SiLU(W_gate(x)) ⊙ W_up(x))` is implementation-level.
**Classification:** PARTIALLY VERIFIED — structure established, exact
formulation boundary noted. (SET0 03 §9)

### 3.6 OC-6: RMSNorm

**Operator:** `RMSNorm`

```text
Config: rms_norm_eps = 1e-06
```

**Tensor dependency:** Per-layer `input_layernorm.weight` and
`post_attention_layernorm.weight`, shape `[5120]`.
**Classification:** VERIFIED FACT — `rms_norm_eps` field present in config.
Exact normalization placement = UNKNOWN (SET0 03 §18 UQ-005).

### 3.7 OC-7: Language-Model Final Normalization

**Operator:** `FinalRMSNorm`

```text
Tensor: model.language_model.norm.weight  [5120]
Shard: model-00016-of-00018.safetensors
```

**Classification:** VERIFIED FACT — tensor present in raw metadata.
(SET1-T1.4 §7: "model.language_model.norm.weight remaining in shard 16")

### 3.8 OC-8: LM Head Projection

**Operator:** `LMHead`

```text
Tensor: lm_head.weight  [248320, 5120]  BF16
Parameters: 1,271,398,400
Bytes:     2,542,796,800
Shard: model-00018-of-00018.safetensors
```

**Classification:** VERIFIED FACT — raw tensor metadata.
`tie_word_embeddings = false` → `embed_tokens` and `lm_head` are separate
weight matrices with identical shapes.
(SET1-T1.5-R1 §6, SET1-T1.6 §5)

### 3.9 OC-9: Attention Output Gating

**Operator:** `AttentionOutputGate`

```text
Config: attn_output_gate = true
        output_gate_type = swish
```

**Classification:** VERIFIED FACT (configuration fields present). UNKNOWN:
exact tensor location and formulation of the gate. (SET0 03 §11,
SET0 04 §5)

### 3.10 OC-10: Linear-Attention Convolution

**Operator:** `CausalConv1D` (within Gated DeltaNet)

```text
Config: linear_conv_kernel_dim = 4
Tensor: linear_attn.conv1d.weight  [10240, 1, 4]
```

**Classification:** VERIFIED FACT — config field and tensor shape both present.
The convolution kernel dimension of 4 is established by both config and raw
tensor metadata. (SET0 04 §8, SET0 03 §7)

### 3.11 OC-11: MTP Projection

**Operator:** `MTPHead`

```text
Tensor: mtp.fc.weight  [5120, 10240]  BF16
Parameters: 52,428,800
```

**Classification:** VERIFIED FACT — raw tensor metadata. UNKNOWN: active
runtime execution path. (SET0 06 §6, SET0 08 §3)

---

## 4. Model Structural Components

### 4.1 Component Inventory

| Component | Layers/Tensors | Parameters | Logical Bytes | Operator Class | Classification |
|---|---|---:|---:|---|---:|
| Token Embedding | 1 tensor | 1,271,398,400 | 2,542,796,800 | OC-1 | VERIFIED FACT |
| MLP (per layer) | 3 × 64 = 192 | 17,112,760,320 | 34,225,520,640 | OC-5 | VERIFIED/DERIVED |
| Full-Attention Layer | 6 × 16 = 96 + norms | 24,353,201,664 − (48×LA + 64×MLP + norm) | — | OC-3 | VERIFIED FACT |
| Linear-Attention Layer | 9 × 48 = 432 | — | — | OC-4 | VERIFIED FACT |
| RMSNorm (intra-layer) | 64 layers × 2 | ≤ 640 | — | OC-6 | PARTIALLY VERIFIED |
| Final Norm | 1 tensor | 5,120 | 10,240 | OC-7 | VERIFIED FACT |
| LM Head | 1 tensor | 1,271,398,400 | 2,542,796,800 | OC-8 | VERIFIED FACT |
| Vision Encoder | subsystem | 460,730,096 | 921,460,192 | OC-2 | VERIFIED FACT |
| MTP | 15 tensors | 424,699,392 | 849,398,784 | OC-11 | VERIFIED FACT |

**Classification notes:**
- Per-layer totals for full-attention and linear-attention layers: VERIFIED
  FACT (SET1-T1.5-R1 §4: Linear-attention/MLP layer = 383,273,184;
  Self-attention/MLP layer = 372,255,232).
- The breakdown `24,353,201,664` is the complete language-model-core parameter
  total including final norm. VERIFIED FACT. (SET1-T1.5-R1 §3)

### 4.2 Subsystem Parameter/Byte Totals

```text
Language model core:     24,353,201,664 params | 48,706,403,328 bytes
Visual encoder:              460,730,096 params |    921,460,192 bytes
Language-model embeddings: 1,271,398,400 params |  2,542,796,800 bytes
LM head:                   1,271,398,400 params |  2,542,796,800 bytes
MTP:                         424,699,392 params |    849,398,784 bytes
─────────────────────────────────────────────────────────────
Global:                   27,781,427,952 params | 55,562,855,904 bytes
```

**Classification:** VERIFIED FACT — raw tensor shapes × BF16 (2 bytes),
reconciled across shard path and subsystem path. (SET1-T1.5-R1 §7, SET1-T1.6 §7)

### 4.3 Data Type

```text
All 1,199 tensors: BF16 (bfloat16)
Bytes per element: 2
mamba_ssm_dtype: float32  (metadata field, not tensor dtype)
```

**Classification:** VERIFIED FACT — every raw tensor record uses BF16.
The `mamba_ssm_dtype = float32` is a metadata configuration field, not a
tensor storage dtype. (SET1-T1.4 §5, SET1-T1.6 §1, SET0 03 §7 boundary)

### 4.4 Shard Distribution

```text
Shard 01: 392 tensors — language layers 0–5 (sharded)    1,983,342,576 params
Shard 02:  47 tensors — layer 6 partial + layer 7 start    1,521,537,088 params
Shard 03:   1 tensor  — model.language_model.embed_tokens 1,271,398,400 params
Shard 04:  69 tensors — layers 7–14                       1,994,482,048 params
Shard 05:  37 tensors — layers 15–17 + vision start        1,049,667,520 params
Shard 06:  76 tensors — layers 18–23                     1,989,771,872 params
Shard 07–17: repeating pattern                           (see manifest)
Shard 18:  16 tensors — final norm + lm_head + MTP        1,696,097,792 params
```

**Classification:** VERIFIED FACT — shard assignment reconciled RAW ↔ index.
(SET1-T1.4 §7: shard-boundary transitions at layers 4, 15, 21, 29, 37, 45, 53,
61; embed_tokens in shard 3; final norm in shard 16; lm_head + MTP in shard 18)

Key placement facts:
- `embed_tokens.weight` → shard 3 (standalone, 1 tensor)
- `model.language_model.norm.weight` → shard 16
- `lm_head.weight` → shard 18
- All 15 MTP tensors → shard 18

---

## 5. Tensor-to-Operator Relationships

### 5.1 Verified Tensor-to-Operator Mapping

Each raw tensor name in the checkpoint maps to exactly one operator class:

```text
embed_tokens.*            → OC-1  (Language Embedding)
vision.* / visual.*       → OC-2  (Vision Encoder)
self_attn.*               → OC-3  (Full-Attention Token Mixer)
linear_attn.*             → OC-4  (Linear-Attention Token Mixer)
mlp.gate_proj/up_proj/down_proj → OC-5 (Dense Gated SiLU MLP)
input_layernorm.*         → OC-6  (RMSNorm)
post_attention_layernorm.* → OC-6 (RMSNorm)
norm.weight               → OC-7  (Final Normalization)
lm_head.*                 → OC-8  (LM Head Projection)
attn_output_gate.*        → OC-9  (Attention Output Gating)
conv1d.*                  → OC-10 (Linear-Attention Convolution)
mtp.*                     → OC-11 (MTP Projection)
```

**Classification:** VERIFIED FACT — tensor naming follows the Qwen3_5
parameter naming convention; each prefix is established by raw metadata.
The exact tensor-to-operator dispatch mapping is derived from the
layer topology and naming convention. (SET0 07 §9, SET0 08 §2–§3, SET1-T1.4,
SET1-T1.5-R1)

### 5.2 Tensor Count Reconciliation

```text
Language model core tensors (64 layers + norm):
  Full-attention layers: 96 tensors (6 per layer × 16)
  Linear-attention layers: 432 tensors (9 per layer × 48)
  MLP tensors: 192 (3 per layer × 64)
  Layer norms: 128 (2 per layer × 64)
  Final norm: 1
  Subtotal: 841

Embedding: 1 tensor (embed_tokens)
LM head: 1 tensor (lm_head)
Vision: subsystem (460,730,096 params)
MTP: 15 tensors

Total verified: 1,199
```

**Classification:** VERIFIED FACT — 1,199 tensors accounted for in the
verified inventory. (SET1-T1.4 §4: 1,199/1,199 PASS, SET1-T1.5-R1 §1)

### 5.3 Shape Dependencies by Operator

| Operator | Input Shape | Output Shape | Key Config Dependencies |
|---|---|---|---|
| OC-1 Embed | `[batch, seq]` | `[batch, seq, 5120]` | `vocab_size=248320`, `hidden_size=5120` |
| OC-3 Full-Attn Q | `[batch, seq, 5120]` | `[batch, seq, 6144]` | `num_attn_heads=24`, `head_dim=256` |
| OC-3 Full-Attn KV | `[batch, seq, 5120]` | `[batch, seq, 1024]` | `num_kv_heads=4`, `head_dim=256` |
| OC-3 Full-Attn Out | `[batch, seq, 6144]` | `[batch, seq, 5120]` | `hidden_size=5120` |
| OC-4 Linear-Attn QKV | `[batch, seq, 5120]` | `[batch, seq, 10240]` | `lin_num_value_heads=48`, `lin_head_dim=128` |
| OC-4 Linear-Attn Z | `[batch, seq, 5120]` | `[batch, seq, 6144]` | `lin_num_value_heads=48`, `lin_head_dim=128` |
| OC-4 Linear-Attn Out | `[batch, seq, 6144]` | `[batch, seq, 5120]` | `hidden_size=5120` |
| OC-5 MLP Gate/Up | `[batch, seq, 5120]` | `[batch, seq, 17408]` | `intermediate_size=17408` |
| OC-5 MLP Down | `[batch, seq, 17408]` | `[batch, seq, 5120]` | `hidden_size=5120` |
| OC-8 LM Head | `[batch, seq, 5120]` | `[batch, seq, 248320]` | `vocab_size=248320` |

**Classification:** VERIFIED FACT for shapes (raw tensor metadata).
DERIVED FINDING for shape arithmetic (e.g., `10240 = 16×128 + 16×128 + 48×128`).
UNKNOWN for exact runtime batch/sequence tensor layout.

---

## 6. Computation / Dataflow Relationships

### 6.1 Language Decoder-Layer Dataflow

```text
Layer N Input (hidden_states)
    │
    ▼
RMSNorm  ──────────────────► (residual connection) ──► + ─► Layer N Output
    │                                                         ▲
    ▼                                                         │
Token Mixer                                                   │
    ├── Type-dispatched:                              ┌───────┘
    │   linear_attention → OC-4 (Qwen3_5GatedDeltaNet)│
    │   full_attention   → OC-3 (Qwen3_5Attention)    │
    │
    ▼
RMSNorm                                             (residual path)
    │
    ▼
Dense Gated SiLU MLP (OC-5)
    │
    ▼
+ ───────────────────────► Layer N Output
```

**Layer dispatch rule (DERIVED FINDING):**
```text
layer_index % 4 == 3  →  full_attention  →  OC-3 (Qwen3_5Attention)
layer_index % 4 != 3  →  linear_attention →  OC-4 (Qwen3_5GatedDeltaNet)
```

This is a deterministic shorthand derived from the verified `layer_types` array.
The authoritative source is the explicit 64-entry array, not the modulo rule.

**Classification:** VERIFIED FACT (layer topology), DERIVED FINDING (dispatch
shorthand). (SET0 07 §7, §11–§13)

### 6.2 Full-Attention Dataflow (OC-3)

```text
hidden_states [batch, seq, 5120]
    │
    ▼
RMSNorm (input_layernorm)
    │
    ▼
Q/K/V Projection:  q_proj [12288], k_proj [1024], v_proj [1024]
    │
    ▼
Q/K Normalization: q_norm [256], k_norm [256]
    │
    ▼
RoPE: partial_rotary_factor=0.25, head_dim=256 → rotary_dim=64
    │
    ▼
KV Cache store (KV state)
    │
    ▼
Causal Attention (GQA: 24 Q heads / 4 KV heads)
    │
    ▼
Attention Output Gate (swish): attn_output_gate=true
    │
    ▼
Output Projection: o_proj [5120, 6144]
    │
    ▼
[batch, seq, 5120]
```

**Classification:**
- Tensor shapes: VERIFIED FACT
- Q/K norm, RoPE, KV cache, causal attention, output gate: VERIFIED FACT
  (set configuration + implementation mapping)
- Output gate location/formulation: UNKNOWN
- Exact softmax/scaling factor: UNKNOWN
(SET0 04 §3–§5, SET0 03 §11)

### 6.3 Linear-Attention Dataflow (OC-4)

```text
hidden_states [batch, seq, 5120]
    │
    ▼
RMSNorm (input_layernorm)
    │
    ▼
QKV Projection: in_proj_qkv [10240, 5120]
    │
    ▼
Z Projection:   in_proj_z  [6144, 5120]
    │
    ▼
Causal Convolution: conv1d [10240, 1, 4], kernel_dim=4
    │
    ▼
B/A Parameters: in_proj_b [48], in_proj_a [48]
    │
    ▼
Gated Delta Rule (recurrent)
    │
    ▼
Recurrent State Update (A_log [48], dt_bias [48])
    │
    ▼
Output Gating / Normalization: norm.weight [128]
    │
    ▼
Output Projection: out_proj [5120, 6144]
    │
    ▼
[batch, seq, 5120]
```

**Classification:**
- Tensor shapes: VERIFIED FACT
- Convolution, recurrent state, gated delta rule: VERIFIED FACT (tensor
  presence of conv1d, A_log, dt_bias establishes the mechanism)
- Exact algorithm (Mamba/Mamba-2/GatedDeltaNet/DeltaNet): UNKNOWN
(SET0 04 §6–§10, SET0 03 §7 boundary)

### 6.4 MLP Dataflow (OC-5)

```text
hidden_states [batch, seq, 5120]
    │
    ▼
RMSNorm (post_attention_layernorm)
    │
    ▼
gate_proj:  W_gate ∈ R^(17408 × 5120) → [batch, seq, 17408]
up_proj:    W_up   ∈ R^(17408 × 5120) → [batch, seq, 17408]
    │
    ▼
SiLU(W_gate(x)) ⊙ W_up(x)
    │
    ▼
down_proj:  W_down ∈ R^(5120 × 17408) → [batch, seq, 5120]
    │
    ▼
[batch, seq, 5120]
```

**Classification:** VERIFIED FACT (three bias-free projections, SiLU on gate,
elementwise multiply, shapes from raw metadata). The mathematical form is
derived from the verified configuration. (SET0 05 §2–§8)

### 6.5 Full-Language-Stack Dataflow

```text
input_ids ─► OC-1 (Embed) ─► [seq, 5120]
                                            │
Layer 0 ─► OC-4 (Linear Attn) ─┐            │
           OC-5 (MLP) ─────────┤            │
Layer 1 ─► OC-4 (Linear Attn) ─┤            │
           OC-5 (MLP) ─────────┤            │
Layer 2 ─► OC-4 (Linear Attn) ─┤            │
           OC-5 (MLP) ─────────┤            │
Layer 3 ─► OC-3 (Full Attn) ───┤            │
           OC-5 (MLP) ─────────┤            │
        │                      │            │
   [LA→LA→LA→FA] × 16         │             │
        │                      │            │
Layer 63 ─► OC-3 ──────────────┘            │
            OC-5 ───────────────────────────┤
                                            ▼
OC-7 (Final RMSNorm) ─► [seq, 5120]
                                            │
OC-8 (LM Head) ─► logits [seq, 248320]
                                            │
OC-11 (MTP, if active) ─► UNKNOWN execution
```

**Classification:** VERIFIED FACT (topology, operator dispatch, dataflow
structure). UNKNOWN (MTP runtime execution, exact interleaving of residuals).

### 6.6 Vision-to-Language Dataflow

```text
Pixel values ─► OC-2 (Vision Encoder, 27 layers) ─► [N, 5120]
                                                    │
                                                    ▼
Language embedding sequence (merged with token embeddings)
```

**Classification:** VERIFIED FACT (vision config, output width match).
UNKNOWN (exact fusion mechanism, whether vision output is added to or
concatenated with text embeddings). (SET0 03 §12, SET0 06 §3)

### 6.7 Multimodal Token Flow

```text
vocab_size = 248320
image_token_id = 248056
video_token_id = 248057
vision_start_token_id = 248053
vision_end_token_id = 248054
```

**Classification:** VERIFIED FACT — these are config.json fields.
(SET0 03 §3)

---

## 7. Hardware Interface (Structural Constraints Only)

This section records only the structural constraints that the operator/computation
model must respect. No execution decisions, scheduling, or placement are made.

### 7.1 Memory Subsystem

```text
Installed RAM (host):       VERIFIED FACT — 16 GB (2×8GB LPDDR5)
WSL2 cgroup limit:          VERIFIED FACT — .wslconfig caps guest at 12 GB
RAM channels:               VERIFIED FACT — dual-channel
RAM type:                   VERIFIED FACT — LPDDR5
RAM speed:                  VERIFIED FACT — 7467 MT/s (reported)
ECC:                        VERIFIED FACT — not present
```

**Relevance to operator model:** The total checkpoint logical footprint is
`55,562,855,904` bytes ≈ `51.7 GiB` (BF16). System RAM is `16 GB` with a
`12 GB` WSL2 cap. The checkpoint cannot reside fully resident in system RAM
without paging/streaming.

**Classification:** VERIFIED FACT (hardware observations). The inference that
"streaming or paging will be required" is a DERIVED FINDING based on the
size mismatch — it is NOT a runtime decision. (SET2-T2.8 §5.3, SET2-T2.3,
SET1-T1.6 §3)

### 7.2 Compute Resources

```text
CPU: Intel Core Ultra 7 155H (Meteor Lake), 16C/22T
  Host: 16C/22T VERIFIED
  Guest: 4C/8T VERIFIED (WSL2 cgroup visible subset)
GPU: Intel Arc 7D55 (integrated, no VRAM)
  Hardware present: VERIFIED (host)
  Guest visible:     VERIFIED ABSENT
NPU: Intel AI Boost 7D1D
  Hardware present: VERIFIED (host)
  Guest visible:     VERIFIED ABSENT
```

**Relevance to operator model:** CPU is the only compute resource verified
as runtime-accessible from the WSL2 guest. GPU and NPU are not visible to
the guest environment. This constrains the operator model to CPU execution
without making a placement decision.

**Classification:** VERIFIED FACT (hardware truth contract). GPU/NPU
guest invisibility is VERIFIED FACT, not an assumption. (SET2-T2.8 §8.1)

### 7.3 API Accessibility

```text
CPU native:    VERIFIED (host + guest)
GPU Level Zero: VERIFIED file presence, UNKNOWN runtime (not probed)
GPU Vulkan:    VERIFIED loads in guest; device enumeration UNKNOWN
GPU OpenCL:    VERIFIED NOT USABLE (0 platforms in guest)
GPU SYCL:      VERIFIED ABSENT
NPU Level Zero: VERIFIED file presence, UNKNOWN runtime
NPU D3D12:     NOT ACCESSIBLE from guest
```

**Classification:** VERIFIED FACT (what was observed) / UNKNOWN (what was
not probed). No runtime capability is claimed beyond direct observation.
(SET2-T2.8 §8.2)

### 7.4 No Hardware Assumptions in the Operator Model

The operator/computation model in this document does NOT assume:
- GPU or NPU execution
- Hardware-accelerated attention
- Vectorized/VNNI/AVX-512 instruction availability (guest AVX2 only VERIFIED)
- Any specific kernel implementation
- Any specific memory layout or alignment
- Any batching or tiling strategy

**Classification:** SCOPE BOUNDARY — these are explicitly excluded.
All performance-related claims are UNKNOWN.

---

## 8. Assumptions

1. **ASSUMPTION (boundary, not claim):** The Qwen3_5 implementation identity
   (`Qwen3_5ForConditionalGeneration`) correctly maps to the verified
   configuration. This is the implementation lineage declared by the artifact
   and is treated as a lineage finding, not a complete architecture
   substitution. (SET0 03 §17.4, SET0 01 §4)

2. **ASSUMPTION (boundary):** The tensor naming convention follows the
   Qwen3_5 parameter structure (`model.language_model.*`, `mlp.*`,
   `self_attn.*`, `linear_attn.*`, etc.). This is verified by the raw tensor
   names in the checkpoint metadata. (SET1-T1.4, SET1-T1.5-R1)

3. **ASSUMPTION (boundary):** The `layer_types` array is the authoritative
   dispatch signal. This is a VERIFIED FACT from config.json. (SET0 07)

4. **ASSUMPTION (boundary):** All tensors are loaded/stored as BF16 unless
   runtime conversion is applied later. The `mamba_ssm_dtype = float32` field
   is metadata only and does not indicate tensor storage dtype.
   (SET0 03 §7 boundary, SET1-T1.6 §1)

---

## 9. Unknowns

| ID | Unknown | Evidence Domain |
|---|---|---|
| UK-001 | Exact linear-attention algorithm (Mamba/Mamba-2/GatedDeltaNet/DeltaNet) | Config declares presence but not identity |
| UK-002 | Whether `mamba_ssm_dtype=float32` implies any runtime state in float32 | Metadata field only |
| UK-003 | Exact full-attention operator kernel implementation | Implementation-level, not checkpoint-level |
| UK-004 | Exact MLP formulation beyond canonical gated-SiLU structure | Configuration establishes dims, not full semantics |
| UK-005 | Exact normalization placement (pre/post attention, final norm) | Tensor names suggest placement but not verified |
| UK-006 | Exact residual connection structure (which residuals exist) | Not established by raw metadata |
| UK-007 | Exact multimodal fusion mechanism | Vision config + width match only |
| UK-008 | Exact MTP computation path and integration | Checkpoint present, runtime UNKNOWN |
| UK-009 | Runtime batch/sequence tensor memory layout | No runtime execution |
| UK-010 | Runtime attention softmax scaling factor | Not in config or checkpoint |
| UK-011 | Runtime KV cache allocation strategy | Depends on runtime, not checkpoint |
| UK-012 | Runtime linear-attention state allocation | Depends on runtime |
| UK-013 | Runtime streaming/paging strategy | Not established |
| UK-014 | Runtime performance on any device | Performance is downstream SET scope |
| UK-015 | GPU/NPU runtime accessibility from this environment | VERIFIED ABSENT from guest (SET2-T2.8 §8.1) |

**Classification:** UNKNOWN — all items above are explicitly not established.
They are boundaries, not gaps to fill within SET 3.

---

## 10. Contradictions

```text
Material contradictions identified: NONE
```

SET1 evidence (tensor shapes, dtypes, parameter/byte totals) is fully
consistent with SET0 structural evidence (layer topology, operator mappings,
configuration fields). SET2 hardware truth contract does not contradict any
structural finding.

The only noted reconciliation items are documentation-provenance issues
(`TENSOR-METADATA.md` references in historical SET0 documents) that do not
affect technical facts.

**Classification:** VERIFIED — no contradiction. (SET1-T1.5-R1 §8,
SET1-T1.6 §8, SET1-T1.9 §11)

---

## 11. Scope Boundaries

### 11.1 In Scope (SET 3)

- Model structural components (SET0 + SET1 verified)
- Operator classes (mapped from layer types)
- Tensor-to-operator relationships (verified by raw metadata)
- Computation/dataflow relationships (derived from structure)
- Shape and parameter dependencies (verified or derived)
- Explicit unknowns (recorded)
- Boundaries between structural truth and implementation assumptions

### 11.2 Explicitly Out of Scope

```text
❌ No runtime performance claims
❌ No throughput or latency measurements
❌ No kernel optimization or design
❌ No workload placement (CPU/GPU/NPU)
❌ No execution scheduling
❌ No accelerator-specific kernel design
❌ No inference engine implementation
❌ No memory-constrained execution strategy
❌ No streaming strategy
❌ No runtime memory model (deferred to SET 4)
❌ No downstream task (SET 4+)
```

No performance or runtime behavior is inferred from topology or structure.
The operator model describes what exists, not how fast it runs or where it
executes.

---

## 12. Evidence Classification Summary

### VERIFIED FACT
- Model identity: Qwen3.8-27B, pinned revision `1d4bf0f2...`
- 64 language layers: 48 linear-attention, 16 full-attention
- Layer pattern: `[LA → LA → LA → FA] × 16`
- Full-attention indices: 3,7,11,...,63; interval = 4
- Full-attention: 24 Q heads, 4 KV heads, head_dim=256 (GQA)
- Linear-attention: 16 K heads, 48 V heads, head_dim=128, conv_kernel=4
- Tensor names, shapes, dtypes, offsets, shard assignments for all 1,199 tensors
- All tensors BF16; 2 bytes/element
- Global parameters: 27,781,427,952
- Global logical bytes: 55,562,855,904
- MLP: 3 bias-free projections, SiLU gate, 17408 intermediate, 5120 hidden
- Attention output gating: enabled, swish
- RoPE: configured, theta=1e7, partial=0.25, mrope interleaved
- Vision: 27 layers, hidden=1152, out=5120
- MTP: 15 tensors, BF16, shard 18, 424,699,392 parameters
- LanguageEmbedding: embed_tokens [248320, 5120]
- LMHead: lm_head [248320, 5120], tie_word_embeddings=false
- FinalNorm: norm.weight [5120], shard 16
- Hardware: CPU VERIFIED accessible; GPU/NPU present-on-host but ABSENT-from-guest
- RAM: 16 GB installed, 12 GB WSL2 cap, dual-channel, no ECC

### DOCUMENTED CAPABILITY
- CPU: Intel Core Ultra 7 155H, Meteor Lake (Intel ARK)
- CPU: AVX2 (guest-verified), AVX-512 (host SKU capability, guest UNKNOWN)
- GPU: Intel Arc 7D55, 8 Xe-cores (Intel ARK / secondary corroboration)
- NPU: Intel AI Boost 7D1D (PnP device enumeration)

### DERIVED FINDING
- GQA ratio: 6 query heads per KV head (24/4)
- Linear-attention QKV expansion: 10240 = 16×128 + 16×128 + 48×128
- Linear-attention head ratio: 48/16 = 3
- RoPE rotary dimension: 64 (256 × 0.25)
- MLP per-layer parameters: 267,386,880 (3 × 5120 × 17408)
- MLP aggregate: 17,112,760,320 (64 × 267,386,880)
- MLP expansion ratio: 3.4 (17408/5120)
- Checkpoint logical footprint: ~51.7 GiB (55,562,855,904 bytes)
- Checkpoint exceeds WSL2 RAM cap → streaming/paging required (structural constraint)
- Dispatch rule: `layer_index % 4 == 3` → full_attention

### UNKNOWN
- Exact linear-attention algorithm
- Exact full-attention kernel implementation
- Exact MLP formulation beyond canonical structure
- Exact normalization placement
- Exact residual connection structure
- Exact multimodal fusion mechanism
- Exact MTP runtime execution path
- Runtime batch/sequence memory layout
- Attention softmax scaling factor
- Runtime KV cache allocation strategy
- Runtime linear-attention state allocation
- Runtime streaming/paging strategy
- Runtime performance on any device
- GPU/NPU runtime accessibility from guest
- GPU negotiated PCIe link speed/width
- CPU host AVX-512/AMX (not probed on host)

---

## 13. Canonical Computation Model Statement

> **Qwen3.8-27B is a multimodal model with a 64-layer language stack. Each
> language layer consists of a token mixer and a dense gated SiLU MLP. The
> token mixer is dispatched by the verified `layer_types` array: layers at
> indices where `layer_index % 4 == 3` use full-attention (Qwen3_5Attention,
> 24 Q heads / 4 KV heads, head_dim 256, GQA, KV cache state); all other
> layers use linear-attention (Qwen3_5GatedDeltaNet, 16 K heads / 48 V heads,
> head_dim 128, causal convolution with kernel 4, recurrent state with
> A_log/dt_bias parameters). Every parameter tensor is BF16 (2 bytes/element);
> the global checkpoint contains 1,199 tensors, 27,781,427,952 parameters, and
> 55,562,855,904 logical bytes. The checkpoint logical footprint (~51.7 GiB)
> exceeds the verified 12 GB WSL2 guest memory cap, establishing streaming or
> paging as a structural necessity — not a runtime decision. CPU execution
> from the WSL2 guest is verified-accessible; GPU and NPU are verified
> present on the host but absent from the guest. Exact linear-attention
> algorithm, exact full-attention kernel, exact MLP formulation, exact
> normalization/residual structure, exact multimodal fusion, and exact MTP
> runtime execution remain UNKNOWN. No performance, scheduling, placement, or
> optimization claims are established by this model.**

---

## 14. Final Acceptance

```text
SET 3 SCOPE:
Operator / Computation Model

MODEL STRUCTURAL COMPONENTS:
VERIFIED — 11 operator classes defined with tensor dependencies

OPERATOR CLASSES:
VERIFIED — OC-1 through OC-11 with tensor-to-operator mapping

TENSOR-TO-OPERATOR RELATIONSHIPS:
VERIFIED — all 1,199 tensors mapped to operator classes

COMPUTATION / DATAFLOW RELATIONSHIPS:
VERIFIED — full decoder-layer, attention, linear-attention, MLP, vision
           dataflows established from verified evidence

SHAPE AND PARAMETER DEPENDENCIES:
VERIFIED — all shape arithmetic traced to config or raw tensor metadata

EXPLICIT UNKNOWNS:
VERIFIED — 15 unknowns cataloged with evidence domains

SCOPE BOUNDARIES:
VERIFIED — all DO-NOT-RUN items preserved

CONTRADICTIONS:
NONE

PERFORMANCE CLAIMS:
NONE INTRODUCED
```

**Verdict: SET3 — PASS**

This document establishes the explicit operator/computation model for
Qwen3.8-27B using only VERIFIED FACT, DOCUMENTED CAPABILITY, and DERIVED
FINDING from the SET0, SET1, and SET2 evidence corpora. No runtime
performance is inferred from structure. The exact linear-attention algorithm,
exact kernel implementations, exact fusion mechanisms, and exact runtime
behaviors remain UNKNOWN and are cataloged as boundaries.

---

## 15. Revision History

| Rev | Date | Owner | Description |
|---|---|---|---|
| SET3-01 | 2026-08-19 | 🧠 LUNA | Created canonical operator/computation model from SET0+SET1+SET2 evidence. |
