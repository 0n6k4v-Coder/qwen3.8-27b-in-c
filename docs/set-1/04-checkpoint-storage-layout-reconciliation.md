# SET 1 — Checkpoint Storage / Physical Layout Reconciliation

## Document Status

- **Document:** `04-checkpoint-storage-layout-reconciliation.md`
- **SET:** `SET 1 — Tensor / Byte-Level Audit`
- **Task ID:** `SET1-T1.8`
- **Task:** Checkpoint Storage / Physical Layout Reconciliation
- **Result:** **VERIFIED PASS**
- **Responsible Role:** 🧠 LUNA
- **Execution Support:** 🛠 EXECUTOR only where explicitly required
- **Date:** 2026-08-16

---

## 1. Objective

This task establishes and reconciles the boundary between:

1. **logical tensor bytes** established by `SET1-T1.6`; and
2. the **physical checkpoint storage representation** exposed by the RAW Safetensors metadata.

The purpose is to establish what can be asserted about checkpoint storage without confusing logical tensor payload size with complete physical checkpoint storage size.

This task is a **storage-structure / evidence-reconciliation task**.

It does **not** establish runtime memory, runtime residency, hardware requirements, loader behavior, streaming behavior, or performance.

---

## 2. Source of Truth

### Authoritative upstream

- **Model:** `Qwen/Qwen3.8-27B`
- **Pinned revision:** `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`
- **Upstream repository:** `Qwen/Qwen3.8-27B`

### Project RAW evidence

```text
model/official/raw-checkpoint-metadata/
```

Relevant artifacts:

```text
model/official/raw-checkpoint-metadata/config.json
model/official/raw-checkpoint-metadata/manifest.json
model/official/raw-checkpoint-metadata/model.safetensors.index.json
model/official/raw-checkpoint-metadata/shards/*.header.json
```

### Supporting SET 1 evidence

```text
docs/set-1/01-raw-metadata-verification.md
docs/set-1/02-parameter-reconstruction.md
docs/set-1/03-tensor-byte-accounting.md
```

### Historical cross-check only

```text
docs/set-0/09-parameter-byte-accounting.md
```

The historical `model/official/TENSOR-METADATA.md` artifact is intentionally removed and must not be recreated.

---

## 3. Verified Foundation from Previous SET 1 Tasks

The following values are established inputs and are not replaced by T1.8:

```text
Tensor count:
1,199

Shard count:
18

Dtype:
BF16

Global parameters:
27,781,427,952

Global logical BF16 bytes:
55,562,855,904
```

MTP:

```text
15 tensors
424,699,392 parameters
849,398,784 logical bytes
```

Embedding:

```text
1,271,398,400 parameters
2,542,796,800 logical bytes
```

LM head:

```text
1,271,398,400 parameters
2,542,796,800 logical bytes
```

T1.8 does not redesign the parameter or logical-byte accounting established by T1.5-R1 and T1.6.

---

# 4. Physical Safetensors Structure

## 4.1 Verified storage model

The persisted acquisition record states that the project obtained each shard using header-only HTTP Range retrieval consisting of:

```text
8-byte uint64 little-endian prefix
+
JSON header bytes
```

and explicitly states that **no tensor payload data was transferred during acquisition**.

This acquisition model corresponds to the Safetensors file structure:

```text
┌──────────────────────────────────────┐
│ 8-byte header-length prefix          │
├──────────────────────────────────────┤
│ JSON metadata header                 │
├──────────────────────────────────────┤
│ Tensor payload region                │
│                                      │
│ tensor payload 1                     │
│ tensor payload 2                     │
│ ...                                  │
│ tensor payload N                     │
└──────────────────────────────────────┘
```

The first two regions are metadata/storage structure; `data_offsets` describe tensor payload locations within the payload region.

### Evidence Classification

**VERIFIED FACT**

- 18 shard headers were acquired and persisted.
- The acquisition record explicitly identifies the 8-byte prefix + JSON-header retrieval method.
- Tensor payloads were not downloaded as part of the header-only acquisition.

---

# 5. Header / Payload Boundary

For a Safetensors shard, the physical representation has three conceptual regions:

```text
8-byte prefix
        ↓
JSON header
        ↓
tensor payload region
```

The project manifest records `header_length_bytes` for shard headers and records the corresponding header hashes and prefix hashes.

Therefore, for a shard with:

```text
header_length_bytes = H
```

and a final tensor offset end:

```text
last_tensor_end = E
```

the physical file-length relationship is structurally:

```text
physical_shard_size
=
8
+
H
+
E
```

provided that the shard follows the standard Safetensors layout represented by the verified RAW metadata.

This relationship is a **structural derivation**, not a claim that the repository currently persists a canonical global physical-file-size total.

### Evidence Classification

**VERIFIED FACT**

- A distinct 8-byte prefix and JSON header exist in the storage format.
- The project acquisition record records header-length information.

**DERIVED FINDING**

- Physical shard file length can be expressed as `8 + header_length_bytes + final_tensor_end_offset` when all terms are available for that shard.

---

# 6. Tensor `data_offsets` Semantics

Each RAW tensor record contains:

```text
data_offsets = [start, end]
```

The tensor payload span is:

```text
offset_span = end - start
```

The logical tensor byte rule established by T1.6 is:

```text
logical_bytes
=
product(shape dimensions)
×
bytes_per_element(dtype)
```

For this checkpoint all tensors are BF16:

```text
bytes_per_element = 2
```

T1.6 established that the RAW offset spans are structurally consistent with the reconstructed logical tensor bytes and reported:

```text
No unexplained offset-span mismatch
```

Therefore the storage interpretation is:

```text
data_offsets
      ↓
physical tensor payload span
      ↓
end - start
      ↓
reconciles with logical tensor bytes
```

`data_offsets` are treated as RAW storage-layout evidence and as a consistency check. They do not replace the T1.5-R1 parameter reconstruction or the T1.6 logical-byte accounting.

### Representative RAW examples

Examples from the persisted shard headers include:

```text
shape: [5120]
logical bytes: 10,240

data_offsets: [0, 10,240]
span: 10,240
```

```text
shape: [48]
logical bytes: 96

data_offsets: [10,240, 10,336]
span: 96
```

```text
shape: [10240, 5120]
logical bytes: 104,857,600

data_offsets: [1,075,392, 105,932,992]
span: 104,857,600
```

```text
shape: [17408, 5120]
logical bytes: 178,257,920

data_offsets: [410,020,288, 588,278,208]
span: 178,257,920
```

```text
shape: [248320, 5120]
logical bytes: 2,542,796,800

data_offsets: [0, 2,542,796,800]
span: 2,542,796,800
```

These examples demonstrate the relationship between logical tensor size and physical tensor payload span.

---

# 7. Per-Shard Payload Layout

The verified T1.6 per-shard logical-byte totals can be interpreted as the reconciled tensor-payload span for each shard because the RAW tensor offset spans are consistent with tensor logical bytes.

| Shard | Tensor Count | Reconciled Tensor Payload Span |
|---:|---:|---:|
| 01 | 392 | 3,966,685,152 |
| 02 | 47 | 3,043,074,176 |
| 03 | 1 | 2,542,796,800 |
| 04 | 69 | 3,988,964,096 |
| 05 | 37 | 2,099,335,040 |
| 06 | 76 | 3,979,543,744 |
| 07 | 30 | 2,108,755,392 |
| 08 | 76 | 3,979,543,744 |
| 09 | 30 | 2,108,755,392 |
| 10 | 76 | 3,979,543,744 |
| 11 | 30 | 2,108,755,392 |
| 12 | 76 | 3,979,543,744 |
| 13 | 30 | 2,108,755,392 |
| 14 | 76 | 3,979,543,744 |
| 15 | 30 | 2,108,755,392 |
| 16 | 77 | 3,979,553,984 |
| 17 | 30 | 2,108,755,392 |
| 18 | 16 | 3,392,195,584 |
| **Total** | **1,199** | **55,562,855,904** |

The total reconciles to the global logical BF16-byte total established by T1.6:

```text
SUM(all 18 shard tensor payload spans)
=
55,562,855,904 bytes
```

### Important interpretation

These are **tensor payload spans**, not complete physical shard file sizes.

Complete physical shard storage additionally includes the file prefix and JSON header.

---

# 8. Header / Metadata Overhead

The physical storage representation includes metadata/header bytes that are not part of the logical tensor payload total.

For each shard:

```text
physical storage
=
8-byte prefix
+
JSON header
+
raw tensor payload region
```

The project manifest records `header_length_bytes` for the acquired shard headers.

Therefore header/storage overhead is structurally identifiable where the corresponding header-length evidence is available.

However, T1.8 does **not** redefine the entire checkpoint's physical byte count as a new canonical accounting total unless all shard-level physical-length components have been independently and persistently reconciled.

### Evidence Classification

**VERIFIED FACT**

The prefix and JSON-header regions exist separately from the tensor payload region.

**DERIVED FINDING**

The non-payload metadata contribution of a shard is structurally represented by:

```text
8 + header_length_bytes
```

### UNKNOWN

The following are not established by T1.8:

- filesystem allocation size;
- transport/download overhead;
- filesystem/page-cache behavior;
- runtime memory residency;
- allocator overhead.

---

# 9. Logical Bytes vs Physical Checkpoint Storage

The fundamental accounting distinction is:

```text
LOGICAL TENSOR BYTES
=
SUM(all tensor payload sizes)
=
55,562,855,904 bytes
```

whereas:

```text
PHYSICAL CHECKPOINT STORAGE
=
SUM(all shard physical file sizes)
```

and each shard physically contains at least the structural components:

```text
8-byte prefix
+
JSON metadata header
+
tensor payload region
```

Therefore:

```text
logical tensor bytes
≠
complete physical checkpoint storage
```

and:

```text
logical tensor bytes
≠
whole-shard file size
```

This distinction is now part of the SET 1 storage truth boundary.

---

# 10. Global Storage Boundary

## Global logical tensor bytes

**VERIFIED**

```text
55,562,855,904 bytes
```

## Total tensor payload span

**VERIFIED / DERIVED**

```text
55,562,855,904 bytes
```

This reconciles with the global logical-byte total.

## Global physical checkpoint size

**NOT CANONICALIZED BY T1.8**

The complete physical checkpoint size is structurally derivable from the per-shard prefix length, header lengths, and final payload boundaries:

```text
SUM(
    8
    + header_length_bytes
    + final_tensor_end_offset
)
```

However, T1.8 does not promote a separately calculated global number to a canonical project fact without an explicit persisted physical-size reconciliation artifact.

Therefore the current evidence classification is:

```text
GLOBAL LOGICAL TENSOR BYTES:
VERIFIED

TOTAL VERIFIED TENSOR PAYLOAD SPAN:
VERIFIED / DERIVED

GLOBAL PHYSICAL CHECKPOINT SIZE:
NOT CANONICALIZED BY T1.8
```

This is not a contradiction. It is a deliberate evidence boundary.

---

# 11. Gaps, Alignment, and Unexplained Regions

T1.8 does not assume padding or alignment bytes solely because physical file storage exists.

The verified RAW tensor offset sequences show the tensor payload spans used for the accounting, and T1.6 found no unexplained offset-span mismatch.

Therefore:

```text
Tensor payload gap:
NO UNEXPLAINED GAP IDENTIFIED IN VERIFIED RAW OFFSET EVIDENCE
```

This statement does not establish filesystem allocation or transport-level padding.

Those remain outside scope.

---

# 12. Evidence Classification

## VERIFIED FACT

- Model: `Qwen/Qwen3.8-27B`.
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`.
- 18 shard headers are part of the verified RAW evidence set.
- The acquisition record documents 8-byte prefix + JSON-header Range retrieval.
- No tensor payload data was transferred during the header-only acquisition.
- RAW tensor metadata contains `data_offsets` for tensors.
- Tensors are BF16.
- Global logical tensor bytes are `55,562,855,904`.
- Tensor payload spans reconcile with logical tensor bytes.
- The physical Safetensors representation contains metadata/header storage in addition to tensor payload storage.

## DERIVED FINDING

- `offset_span = end - start` represents the tensor payload span described by the RAW offsets.
- Per-shard tensor payload spans reconcile to the T1.6 per-shard logical-byte totals.
- The sum of all shard tensor payload spans is `55,562,855,904` bytes.
- For a shard with known header length `H` and final tensor end `E`, physical shard length is structurally `8 + H + E`.
- Logical tensor bytes and complete physical checkpoint storage are distinct quantities.

## UNKNOWN

- A separately canonicalized global physical checkpoint-size total.
- Filesystem allocation size.
- Filesystem/page-cache effects.
- Transport/download overhead.
- Runtime-resident memory.
- Allocator overhead.
- Hardware-specific storage behavior.

---

# 13. SET 1 Impact

## T1.4 — Tensor Shape / Dtype / Offset Audit

**UNCHANGED**

The storage-layout evidence is consistent with the existing RAW structural audit.

## T1.5-R1 — Tensor Parameter Reconstruction

**UNCHANGED**

```text
27,781,427,952 parameters
```

## T1.6 — Tensor Logical Byte Accounting

**UNCHANGED**

```text
55,562,855,904 logical BF16 bytes
```

## T1.7 — Final Evidence Reconciliation

**UNCHANGED**

The T1.8 findings extend the evidence chain and do not introduce a contradiction with T1.7.

---

# 14. Boundary Conditions

This document does **not** establish:

- runtime resident memory;
- allocator overhead;
- KV-cache memory;
- activation memory;
- hardware memory capacity;
- GPU/NPU placement;
- memory mapping strategy;
- loader implementation;
- streaming strategy;
- I/O optimization;
- runtime performance.

These belong to later SETs.

---

# 15. Control Result

```text
SET1-T1.8:
✅ VERIFIED PASS

SET1-T1.9:
🔒 NOT STARTED

SET1-CLOSE:
🔒 NOT STARTED

SET 2:
🔒 NOT STARTED
```

### T1.8 Final Decision

**VERIFIED PASS**

T1.8 establishes the following SET 1 storage truth:

```text
Tensor Truth
      ↓
Logical Byte Truth
      ↓
Tensor Payload Span Truth
      ↓
Safetensors Header / Payload Boundary
      ↓
Physical-vs-Logical Storage Boundary
```

The global logical tensor-byte total remains:

```text
55,562,855,904 bytes
```

and the evidence does not justify replacing that value with a physical-file-size claim.

---

# 16. Next Task Boundary

T1.8 is complete.

The next task is:

```text
SET1-T1.9 — SET 1 Boundary / Completeness Audit
```

T1.9 must determine whether the complete SET 1 technical evidence scope has been satisfied.

T1.9 must not begin formal SET 1 closure automatically.

`SET1-CLOSE` remains a separate formal acceptance task.

---

# 17. Provenance Boundary

This document is grounded in the persisted RAW checkpoint metadata under:

```text
model/official/raw-checkpoint-metadata/
```

The removed:

```text
model/official/TENSOR-METADATA.md
```

is not required and must not be recreated.

The SET 0 document reference to that removed historical artifact remains a documentation-provenance issue only.

---

## Conclusion

`SET1-T1.8` establishes the storage-layer boundary required by SET 1:

```text
logical tensor bytes
        ≠
physical checkpoint storage
```

The verified tensor payload spans reconcile with the global logical byte total:

```text
55,562,855,904 bytes
```

The physical checkpoint representation additionally contains the Safetensors prefix and JSON metadata header. The complete physical checkpoint size is therefore a distinct quantity and is not promoted here as a canonical total without a separately persisted physical-size reconciliation.

This preserves the required evidence discipline:

```text
VERIFIED FACT
+
DERIVED FINDING
+
UNKNOWN
```

without expanding T1.8 into runtime or hardware analysis.