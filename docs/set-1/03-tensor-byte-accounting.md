# SET 1 — Tensor Byte Accounting

## Task

- **Task ID:** SET1-T1.6
- **Task:** Tensor Byte Accounting
- **Result:** VERIFIED PASS
- **Responsible Role:** 🧠 LUNA
- **Date:** 2026-08-16

## Source of Truth

### Authoritative Upstream

- Model: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

### Project Raw Evidence

Primary tensor metadata:

```text
model/official/raw-checkpoint-metadata/shards/*.header.json
```

Official tensor index:

```text
model/official/raw-checkpoint-metadata/model.safetensors.index.json
```

RAW tensor `shape` and RAW `dtype` are the authoritative inputs for this byte reconstruction.

---

## Scope

This task reconstructs **logical tensor bytes only**.

For each tensor:

```text
logical_bytes = product(shape dimensions) × bytes_per_element(dtype)
```

The reconstruction covers:

1. all 1,199 tensors;
2. all 18 shard totals;
3. dtype distribution;
4. subsystem logical-byte totals;
5. MTP logical bytes;
6. embedding and LM-head logical bytes;
7. global logical bytes;
8. `data_offsets` span consistency.

This document does **not** establish:

- runtime resident memory;
- allocator overhead;
- KV-cache memory;
- activation memory;
- hardware placement;
- runtime MTP execution behavior;
- SET 1 closure.

---

## Verified Input State

Prior tasks established:

- `SET1-T1.1` — Raw Metadata Acquisition: **PASS**
- `SET1-T1.2` — Raw Metadata Verification: **PASS**
- `SET1-T1.4` — Tensor Shape / Dtype / Offset Audit: **PASS**
- `SET1-T1.5-R1` — Tensor Parameter Reconstruction: **VERIFIED PASS**

Verified inventory:

```text
18 shards
1,199 tensors
27,781,427,952 parameters
```

The removed `model/official/TENSOR-METADATA.md` is not used and must not be recreated.

---

# 1. Dtype Reconciliation

All 1,199 RAW tensor records use `BF16`.

| Dtype | Tensor Count | Parameters | Logical Bytes | Bytes / Element |
|---|---:|---:|---:|---:|
| BF16 | 1,199 | 27,781,427,952 | 55,562,855,904 | 2 |
| **Total** | **1,199** | **27,781,427,952** | **55,562,855,904** | |

Therefore:

```text
BF16 width = 2 bytes / element
```

and:

```text
27,781,427,952 × 2
=
55,562,855,904 logical bytes
```

---

# 2. Per-Shard Byte Reconciliation

| Shard | Tensor Count | Parameters | Logical Bytes |
|---:|---:|---:|---:|
| 01 | 392 | 1,983,342,576 | 3,966,685,152 |
| 02 | 47 | 1,521,537,088 | 3,043,074,176 |
| 03 | 1 | 1,271,398,400 | 2,542,796,800 |
| 04 | 69 | 1,994,482,048 | 3,988,964,096 |
| 05 | 37 | 1,049,667,520 | 2,099,335,040 |
| 06 | 76 | 1,989,771,872 | 3,979,543,744 |
| 07 | 30 | 1,054,377,696 | 2,108,755,392 |
| 08 | 76 | 1,989,771,872 | 3,979,543,744 |
| 09 | 30 | 1,054,377,696 | 2,108,755,392 |
| 10 | 76 | 1,989,771,872 | 3,979,543,744 |
| 11 | 30 | 1,054,377,696 | 2,108,755,392 |
| 12 | 76 | 1,989,771,872 | 3,979,543,744 |
| 13 | 30 | 1,054,377,696 | 2,108,755,392 |
| 14 | 76 | 1,989,771,872 | 3,979,543,744 |
| 15 | 30 | 1,054,377,696 | 2,108,755,392 |
| 16 | 77 | 1,989,776,992 | 3,979,553,984 |
| 17 | 30 | 1,054,377,696 | 2,108,755,392 |
| 18 | 16 | 1,696,097,792 | 3,392,195,584 |
| **Total** | **1,199** | **27,781,427,952** | **55,562,855,904** |

Reconciliation:

```text
SUM(all 18 shard logical bytes)
=
55,562,855,904
```

---

# 3. Subsystem Byte Reconciliation

| Subsystem | Logical Bytes |
|---|---:|
| Language model core | 48,706,403,328 |
| Visual encoder | 921,460,192 |
| Language-model embeddings | 2,542,796,800 |
| LM head | 2,542,796,800 |
| MTP | 849,398,784 |
| **Global** | **55,562,855,904** |

Reconciliation:

```text
48,706,403,328
+   921,460,192
+ 2,542,796,800
+ 2,542,796,800
+   849,398,784
----------------
=55,562,855,904
```

Every tensor is assigned to one aggregation category only.

---

# 4. MTP Byte Reconstruction

The RAW metadata contains exactly 15 MTP tensors.

| Tensor | Shape | Dtype | Parameters | Logical Bytes |
|---|---|---|---:|---:|
| `mtp.fc.weight` | `[5120, 10240]` | BF16 | 52,428,800 | 104,857,600 |
| `mtp.layers.0.input_layernorm.weight` | `[5120]` | BF16 | 5,120 | 10,240 |
| `mtp.layers.0.mlp.down_proj.weight` | `[5120, 17408]` | BF16 | 89,128,960 | 178,257,920 |
| `mtp.layers.0.mlp.gate_proj.weight` | `[17408, 5120]` | BF16 | 89,128,960 | 178,257,920 |
| `mtp.layers.0.mlp.up_proj.weight` | `[17408, 5120]` | BF16 | 89,128,960 | 178,257,920 |
| `mtp.layers.0.post_attention_layernorm.weight` | `[5120]` | BF16 | 5,120 | 10,240 |
| `mtp.layers.0.self_attn.k_norm.weight` | `[256]` | BF16 | 256 | 512 |
| `mtp.layers.0.self_attn.k_proj.weight` | `[1024, 5120]` | BF16 | 5,242,880 | 10,485,760 |
| `mtp.layers.0.self_attn.o_proj.weight` | `[5120, 6144]` | BF16 | 31,457,280 | 62,914,560 |
| `mtp.layers.0.self_attn.q_norm.weight` | `[256]` | BF16 | 256 | 512 |
| `mtp.layers.0.self_attn.q_proj.weight` | `[12288, 5120]` | BF16 | 62,914,560 | 125,829,120 |
| `mtp.layers.0.self_attn.v_proj.weight` | `[1024, 5120]` | BF16 | 5,242,880 | 10,485,760 |
| `mtp.norm.weight` | `[5120]` | BF16 | 5,120 | 10,240 |
| `mtp.pre_fc_norm_embedding.weight` | `[5120]` | BF16 | 5,120 | 10,240 |
| `mtp.pre_fc_norm_hidden.weight` | `[5120]` | BF16 | 5,120 | 10,240 |
| **MTP total** | | | **424,699,392** | **849,398,784** |

---

# 5. Embedding / LM Head Byte Check

## Language-model embeddings

```text
model.language_model.embed_tokens.weight
shape: [248320, 5120]
dtype: BF16
parameters: 1,271,398,400
logical bytes: 2,542,796,800
```

## LM head

```text
lm_head.weight
shape: [248320, 5120]
dtype: BF16
parameters: 1,271,398,400
logical bytes: 2,542,796,800
```

---

# 6. Offset Consistency

For every tensor, the RAW metadata exposes:

```text
data_offsets = [start, end]
```

The structural consistency check compares:

```text
offset_span = end - start
```

against:

```text
logical_bytes = product(shape) × bytes_per_element(dtype)
```

Result:

**PASS**

No unexplained offset-span mismatch was identified in the verified RAW metadata.

Representative checks include:

```text
[5120]
→ 10,240 bytes

[48]
→ 96 bytes

[10240, 5120]
→ 104,857,600 bytes

[17408, 5120]
→ 178,257,920 bytes

[248320, 5120]
→ 2,542,796,800 bytes
```

`data_offsets` are treated as RAW metadata and as a consistency check only. They are not used as the authoritative source for logical byte accounting.

---

# 7. Global Reconciliation

## Shard path

```text
SUM(all 18 shard logical bytes)
=
55,562,855,904
```

## Subsystem path

```text
48,706,403,328
+   921,460,192
+ 2,542,796,800
+ 2,542,796,800
+   849,398,784
----------------
=55,562,855,904
```

## Parameter-width path

```text
27,781,427,952 parameters
× 2 bytes / BF16 element
=
55,562,855,904 logical bytes
```

Therefore:

```text
GLOBAL LOGICAL BF16 BYTES
=
55,562,855,904
```

---

# 8. Cross-Document Check

Compared against:

```text
docs/set-0/09-parameter-byte-accounting.md
docs/set-1/02-parameter-reconstruction.md
```

Result:

**CONSISTENT**

The independently reconstructed values are:

```text
Global parameters:
27,781,427,952

Global logical BF16 bytes:
55,562,855,904

MTP parameters:
424,699,392

MTP logical BF16 bytes:
849,398,784
```

The reconstruction uses RAW metadata as the calculation source; existing documents are comparison targets only.

### Documentation Provenance Note

`docs/set-0/09-parameter-byte-accounting.md` still references the removed:

```text
model/official/TENSOR-METADATA.md
```

This is a documentation-provenance issue and does not invalidate T1.6. The SET 0 document is not modified by this task.

---

# 9. Evidence Classification

## VERIFIED FACT

- 18 RAW shard headers are part of the verified evidence set.
- 1,199 tensors are part of the verified inventory.
- RAW dtype records are BF16.
- RAW shapes and dtypes are the accounting inputs.
- MTP contains exactly 15 tensors.
- `data_offsets` are present in the RAW tensor metadata.

## DERIVED FINDING

- Per-tensor logical byte counts.
- Per-shard logical byte totals.
- Subsystem logical byte totals.
- MTP logical bytes: `849,398,784`.
- Global logical bytes: `55,562,855,904`.

## UNKNOWN

- Runtime resident memory.
- Allocator overhead.
- KV-cache memory.
- Activation memory.
- Runtime MTP execution behavior.
- Hardware-specific memory placement.

---

# 10. Scope and Control State

This task did not:

- modify RAW metadata;
- modify official artifacts;
- modify SET 0 documents;
- modify SET 1 documents;
- recreate `TENSOR-METADATA.md`;
- perform runtime memory analysis;
- perform hardware research;
- perform runtime research;
- begin SET 1 closure;
- modify `ROADMAP.md`.

Current task state:

```text
SET1-T1.6
✅ VERIFIED PASS
```

Next Atomic Task:

```text
SET1-T1.7
```

`ROADMAP.md` remains **DEFERRED** under the current control protocol.

---

# Conclusion

`SET1-T1.6` establishes a verified logical-byte reconstruction of the pinned Qwen3.8-27B checkpoint:

```text
55,562,855,904 logical BF16 bytes
```

The result reconciles across:

```text
1,199 tensors
18 shards
dtype distribution
subsystems
MTP
embedding
LM head
data-offset spans
global total
```

The result is a logical checkpoint-byte model, not a runtime-memory model.

The next task must be defined separately; T1.7 is not started by this document.