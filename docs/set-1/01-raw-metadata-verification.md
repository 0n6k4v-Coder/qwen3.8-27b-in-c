# SET 1 — Raw Metadata Verification

## Document Status

- Task ID: `SET1-T1.4`
- Verification scope: Raw checkpoint metadata integrity
- Status: **VERIFIED PASS**
- Target model: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`
- Raw metadata root: `model/official/raw-checkpoint-metadata/`

> This document records the verification result for the raw checkpoint metadata acquisition. It does not replace the raw evidence and does not perform parameter or byte reconstruction.

---

## 1. Authoritative Source

Official model repository:

`https://huggingface.co/Qwen/Qwen3.8-27B`

Pinned revision:

`1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`

Primary persisted raw metadata:

```text
model/official/raw-checkpoint-metadata/
├── config.json
├── manifest.json
├── model.safetensors.index.json
└── shards/
    ├── model-00001-of-00018.header.json
    ├── ...
    └── model-00018-of-00018.header.json
```

The raw metadata is the canonical checkpoint-metadata evidence for SET 1.

`model/official/TENSOR-METADATA.md` is intentionally not part of this evidence chain and must not be recreated.

---

## 2. Verification Scope

The verification covered:

- all 18 Safetensors shard-header artifacts
- complete 1,199-tensor raw inventory
- tensor name integrity
- tensor shape integrity
- tensor dtype integrity
- `data_offsets` structural integrity
- tensor-to-shard assignment
- RAW ↔ official index reconciliation
- MTP structural spot-check

This verification did **not** perform:

- parameter reconstruction
- byte accounting
- subsystem accounting
- runtime analysis
- hardware analysis
- SET 1 closure

---

## 3. Shard Completeness

Expected shard count:

`18`

Verified shard headers:

```text
model-00001-of-00018.header.json
model-00002-of-00018.header.json
model-00003-of-00018.header.json
model-00004-of-00018.header.json
model-00005-of-00018.header.json
model-00006-of-00018.header.json
model-00007-of-00018.header.json
model-00008-of-00018.header.json
model-00009-of-00018.header.json
model-00010-of-00018.header.json
model-00011-of-00018.header.json
model-00012-of-00018.header.json
model-00013-of-00018.header.json
model-00014-of-00018.header.json
model-00015-of-00018.header.json
model-00016-of-00018.header.json
model-00017-of-00018.header.json
model-00018-of-00018.header.json
```

Result:

**18 / 18 — PASS**

No missing expected shard and no unexpected shard was identified in the verified acquisition set.

---

## 4. Tensor Inventory

Verified raw tensor inventory:

`1,199 tensors`

The inventory was established from the complete set of 18 acquired shard headers.

Result:

**1,199 / 1,199 — PASS**

No duplicate tensor name or missing tensor was identified during the raw structural verification.

---

## 5. Raw Tensor Record Integrity

For the acquired raw header records, the expected raw metadata fields were present and structurally valid:

- tensor name
- `dtype`
- `shape`
- `data_offsets`

The raw dtype values were preserved as recorded by the checkpoint metadata. No normalization or reinterpretation was applied.

The `data_offsets` records were structurally represented as `[start, end]` pairs with valid ordering in the inspected raw headers.

Result:

**PASS**

---

## 6. Config and Index Presence

Verified artifacts:

```text
model/official/raw-checkpoint-metadata/config.json
model/official/raw-checkpoint-metadata/model.safetensors.index.json
```

Both artifacts are part of the acquired raw checkpoint metadata set and were used as verification inputs.

Result:

**PASS**

---

## 7. RAW ↔ Official Index Reconciliation

The complete raw shard inventory was reconciled against the official `weight_map` in:

`model/official/raw-checkpoint-metadata/model.safetensors.index.json`

The reconciliation covered the complete tensor inventory rather than sample-only checks.

Verified agreement included shard-boundary transitions such as:

- layer 4 crossing from shard 1 to shard 2
- layer 15 crossing from shard 4 to shard 5
- layer 21 crossing from shard 6 to shard 7
- layer 29 crossing from shard 8 to shard 9
- layer 37 crossing from shard 10 to shard 11
- layer 45 crossing from shard 12 to shard 13
- layer 53 crossing from shard 14 to shard 15
- layer 61 crossing from shard 16 to shard 17
- `model.language_model.norm.weight` remaining in shard 16
- `lm_head.weight` residing in shard 18
- MTP tensors residing in shard 18

Result:

**RAW ↔ INDEX — PASS**

No shard-assignment contradiction was identified.

---

## 8. MTP Structural Spot-Check

The raw metadata contains the expected MTP tensor family.

Verified examples:

```text
mtp.layers.0.mlp.gate_proj.weight
shape: [17408, 5120]
dtype: BF16
```

```text
mtp.layers.0.mlp.up_proj.weight
shape: [17408, 5120]
dtype: BF16
```

The complete MTP tensor family is present in shard 18.

Result:

**PASS**

No parameter contribution or byte accounting is performed in this document.

---

## 9. Payload Boundary

The acquired artifacts are checkpoint metadata/header evidence rather than full tensor payloads.

The raw shard artifacts used for this verification contain Safetensors header metadata only; they are not full `.safetensors` tensor payload files.

Result:

**HEADER-ONLY METADATA EVIDENCE — PASS**

---

## 10. Provenance and Evidence Classification

### VERIFIED FACT

- Official model: `Qwen/Qwen3.8-27B`
- Pinned revision: `1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0`
- 18 shard headers are present
- 1,199 tensors are present in the raw inventory
- Raw tensor records contain name, dtype, shape, and data offsets
- Official Safetensors index is present
- RAW ↔ INDEX shard assignment reconciliation passes
- MTP tensor family is present in shard 18

### DERIVED FINDING

- The raw metadata acquisition is structurally complete and suitable as the canonical raw-metadata foundation for subsequent SET 1 analysis.

### UNKNOWN

The following are intentionally not established by this verification:

- runtime-active parameter count
- runtime MTP execution behavior
- runtime memory consumption
- hardware placement
- performance characteristics

---

## 11. Verification Conclusion

**SET1-T1.4 — TENSOR SHAPE / DTYPE / OFFSET AUDIT: VERIFIED PASS**

The acquired raw checkpoint metadata is sufficiently complete, internally consistent, and structurally reconciled with the official checkpoint index to serve as the canonical raw metadata foundation for subsequent SET 1 work.

This verification does not authorize parameter reconstruction, byte accounting, or SET 1 closure by itself. It establishes the tensor-level structural foundation required by the subsequent SET 1 accounting tasks.

---

## 12. Next Control State

Completed:

```text
SET1-T1.1 — Raw Metadata Acquisition
✅ PASS

SET1-T1.2 — Raw Metadata Verification
✅ PASS

SET1-T1.4 — Tensor Shape / Dtype / Offset Audit
✅ PASS
```

The next planned analytical task is:

```text
SET1-T1.5 — Tensor Parameter Reconstruction
```

`ROADMAP.md` persistence remains controlled separately by the project protocol and is not modified by this document.