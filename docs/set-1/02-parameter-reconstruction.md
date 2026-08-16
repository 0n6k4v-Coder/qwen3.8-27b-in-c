# SET 1 — Tensor Parameter Reconstruction

## Task

- **Task ID:** SET1-T1.5-R1
- **Task:** Tensor Parameter Reconstruction — Revalidation
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

The RAW tensor shapes are the calculation source.

The previous T1.5 result was explicitly treated as non-canonical and was not used as an input.

---

## Scope

This task reconstructs parameter counts only.

For every tensor:

```text
parameter_count = product(shape dimensions)
```

The reconstruction covers:

1. all 1,199 tensors;
2. all 18 shard totals;
3. subsystem totals;
4. MTP tensor-level reconstruction;
5. embedding and LM-head reconstruction;
6. global reconciliation.

This document does **not** establish:

- byte accounting;
- runtime memory requirements;
- runtime-active parameter count;
- hardware placement;
- SET 1 closure.

---

## Verified Input State

Prior task `SET1-T1.4 — Tensor Shape / Dtype / Offset Audit` established:

- 18/18 shard headers available;
- 1,199/1,199 tensors accounted for;
- tensor names structurally valid;
- shapes structurally valid;
- dtypes structurally valid;
- data offsets structurally valid;
- tensor-to-shard assignments reconciled;
- RAW ↔ official index reconciliation passed.

`model/official/TENSOR-METADATA.md` is not used and must not be recreated.

---

## Reconstruction Method

For a tensor with:

```text
shape = [d1, d2, ..., dn]
```

the parameter count is:

```text
d1 × d2 × ... × dn
```

Parameter counts are derived from RAW shapes only.

The following are not calculation inputs:

- `data_offsets`;
- dtype;
- shard file size;
- checkpoint size;
- `metadata.total_size` from the index;
- previous parameter totals;
- previous generated metadata documents.

Every tensor is assigned to exactly one shard and exactly one aggregation category.

---

# 1. Inventory

```text
Tensors reconstructed: 1,199
Shards:                 18
Missing tensors:         0
Duplicate tensors:      0
Double-counted tensors: 0
```

---

# 2. Per-Shard Parameter Reconciliation

| Shard | Tensor Count | Parameters |
|---:|---:|---:|
| 01 | 392 | 1,983,342,576 |
| 02 | 47 | 1,521,537,088 |
| 03 | 1 | 1,271,398,400 |
| 04 | 69 | 1,994,482,048 |
| 05 | 37 | 1,049,667,520 |
| 06 | 76 | 1,989,771,872 |
| 07 | 30 | 1,054,377,696 |
| 08 | 76 | 1,989,771,872 |
| 09 | 30 | 1,054,377,696 |
| 10 | 76 | 1,989,771,872 |
| 11 | 30 | 1,054,377,696 |
| 12 | 76 | 1,989,771,872 |
| 13 | 30 | 1,054,377,696 |
| 14 | 76 | 1,989,771,872 |
| 15 | 30 | 1,054,377,696 |
| 16 | 77 | 1,989,776,992 |
| 17 | 30 | 1,054,377,696 |
| 18 | 16 | 1,696,097,792 |

Global shard reconciliation:

```text
SUM(shard 01..18)
=
27,781,427,952 parameters
```

---

# 3. Subsystem Reconciliation

| Subsystem | Parameters |
|---|---:|
| Language model core | 24,353,201,664 |
| Visual encoder | 460,730,096 |
| Language-model embeddings | 1,271,398,400 |
| LM head | 1,271,398,400 |
| MTP | 424,699,392 |
| **Global** | **27,781,427,952** |

Reconciliation:

```text
24,353,201,664
+   460,730,096
+ 1,271,398,400
+ 1,271,398,400
+   424,699,392
----------------
=27,781,427,952
```

---

# 4. Language Model Core Reconstruction

The verified raw tensor structure yields the following repeated layer families:

```text
48 × linear-attention / MLP layer
16 × self-attention / MLP layer
```

Derived layer totals:

```text
Linear-attention / MLP layer:
383,273,184 parameters

Self-attention / MLP layer:
372,255,232 parameters
```

Therefore:

```text
48 × 383,273,184
+
16 × 372,255,232
=
24,353,196,544
```

Final language-model normalization:

```text
+ 5,120
=
24,353,201,664
```

This is a derived reconstruction from raw tensor shapes.

---

# 5. MTP Reconstruction

The raw metadata contains exactly **15 MTP tensors**.

| Tensor | Shape | Parameters |
|---|---|---:|
| `mtp.fc.weight` | `[5120, 10240]` | 52,428,800 |
| `mtp.layers.0.input_layernorm.weight` | `[5120]` | 5,120 |
| `mtp.layers.0.mlp.down_proj.weight` | `[5120, 17408]` | 89,128,960 |
| `mtp.layers.0.mlp.gate_proj.weight` | `[17408, 5120]` | 89,128,960 |
| `mtp.layers.0.mlp.up_proj.weight` | `[17408, 5120]` | 89,128,960 |
| `mtp.layers.0.post_attention_layernorm.weight` | `[5120]` | 5,120 |
| `mtp.layers.0.self_attn.k_norm.weight` | `[256]` | 256 |
| `mtp.layers.0.self_attn.k_proj.weight` | `[1024, 5120]` | 5,242,880 |
| `mtp.layers.0.self_attn.o_proj.weight` | `[5120, 6144]` | 31,457,280 |
| `mtp.layers.0.self_attn.q_norm.weight` | `[256]` | 256 |
| `mtp.layers.0.self_attn.q_proj.weight` | `[12288, 5120]` | 62,914,560 |
| `mtp.layers.0.self_attn.v_proj.weight` | `[1024, 5120]` | 5,242,880 |
| `mtp.norm.weight` | `[5120]` | 5,120 |
| `mtp.pre_fc_norm_embedding.weight` | `[5120]` | 5,120 |
| `mtp.pre_fc_norm_hidden.weight` | `[5120]` | 5,120 |
| **MTP total** | | **424,699,392** |

---

# 6. Embedding and LM Head Reconstruction

## Language-model embeddings

```text
model.language_model.embed_tokens.weight
shape: [248320, 5120]

parameters:
1,271,398,400
```

## LM head

```text
lm_head.weight
shape: [248320, 5120]

parameters:
1,271,398,400
```

The equality of these two parameter counts is independently established from their identical raw shapes; it is not assumed.

---

# 7. Global Parameter Reconstruction

Two independent aggregation paths reconcile to the same total.

### Shard path

```text
SUM(all 18 shard parameter totals)
=
27,781,427,952
```

### Subsystem path

```text
24,353,201,664
+   460,730,096
+ 1,271,398,400
+ 1,271,398,400
+   424,699,392
----------------
=27,781,427,952
```

Therefore:

```text
GLOBAL RECONSTRUCTED PARAMETERS
=
27,781,427,952
```

---

# 8. Cross-Document Check

Compared against:

```text
docs/set-0/09-parameter-byte-accounting.md
```

Result:

**CONSISTENT**

The independently reconstructed raw-shape total matches the previously persisted SET 0 global parameter total:

```text
27,781,427,952
```

The MTP reconstruction also matches:

```text
424,699,392
```

The SET 0 document remains historical evidence; RAW metadata remains the primary calculation source for SET 1.

### Documentation Note

`docs/set-0/09-parameter-byte-accounting.md` still references the removed:

```text
model/official/TENSOR-METADATA.md
```

This is a documentation-provenance issue and does not invalidate the T1.5 reconstruction.

That migration is intentionally outside the T1.5 scope.

---

# 9. Evidence Classification

## VERIFIED FACT

- 18 shard headers are available from the T1.4-verified raw metadata.
- 1,199 tensors were structurally verified in T1.4.
- Raw tensor shapes are the reconstruction inputs.
- MTP contains 15 tensors.
- `embed_tokens.weight` shape is `[248320, 5120]`.
- `lm_head.weight` shape is `[248320, 5120]`.

## DERIVED FINDING

- Per-tensor parameter counts.
- Per-shard parameter totals.
- Subsystem parameter totals.
- MTP parameter total: `424,699,392`.
- Global parameter total: `27,781,427,952`.

## UNKNOWN

- Runtime-active parameter count.
- MTP runtime execution semantics.
- Runtime memory consumption.
- Hardware placement.

---

# 10. Scope and Control State

This task did not:

- modify raw metadata;
- modify SET 0 technical documents;
- recreate `TENSOR-METADATA.md`;
- perform byte accounting;
- perform hardware research;
- perform runtime research;
- begin SET 1 closure;
- modify `ROADMAP.md`.

Current control state:

```text
SET1-T1.5-R1
✅ VERIFIED PASS

NEXT:
SET1-T1.6 — Tensor Byte Accounting
```

`ROADMAP.md` remains:

```text
DEFERRED
```

---

# Conclusion

`SET1-T1.5-R1` establishes a verified, RAW-shape-derived parameter reconstruction of:

```text
27,781,427,952 parameters
```

The reconstruction reconciles across:

```text
1,199 tensors
18 shards
subsystems
MTP
embedding
LM head
global total
```

The next Atomic Task is:

```text
SET1-T1.6 — Tensor Byte Accounting
```

Byte accounting is intentionally not included in this document.