# SET 0 — Artifact Provenance

## Document Status

- Document: `01-artifact-provenance.md`
- SET: `SET 0 — Model Reconnaissance`
- Status: VERIFIED
- Purpose: Canonical provenance record for the Qwen3.8-27B research artifact

---

## 1. Model Identity

- Model: `Qwen3.8-27B`
- Publisher: `Qwen`
- Official Repository: `Qwen/Qwen3.8-27B`

Official repository:

https://huggingface.co/Qwen/Qwen3.8-27B

---

## 2. Pinned Artifact Revision

The research baseline is pinned to:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
````

All artifact acquisition and verification performed during SET 0 must
refer to this exact revision unless a later revision is explicitly
introduced and independently verified.

---

## 3. Artifact Acquisition

### Primary Artifact

```text
model/official/config.json
```

Acquisition source:

```text
https://huggingface.co/Qwen/Qwen3.8-27B/resolve/1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0/config.json
```

Acquisition result:

```text
HTTP 200
```

Local file size:

```text
4312 bytes
```

---

## 4. Local Artifact Integrity

SHA-256 of the locally stored file:

```text
191e0af232104ed8b65258cf3fb2b842e288008baca7633c11b82a1ac7203aab
```

The local `config.json` was validated as syntactically valid JSON.

---

## 5. Official Artifact Comparison

The official `config.json` was retrieved again from the exact pinned
revision and compared against the local artifact.

Comparison method:

```text
Official pinned config.json
        ↓
Temporary local copy
        ↓
Byte-level comparison
        ↓
Local model/official/config.json
```

Result:

```text
IDENTICAL
```

Both files:

```text
4312 bytes
```

No byte-level differences were detected.

Therefore:

```text
Local config.json
=
Official Qwen3.8-27B config.json
at revision 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

---

## 6. Hugging Face File Metadata

The official HTTP response exposed the following ETag:

```text
706cebd746c4b6f2b1d1f892630867acfdfd3df8
```

The ETag was identified by the Executor as a Git blob SHA-1 identifier,
not as the SHA-256 checksum of the file.

Therefore it is recorded as HTTP/repository metadata and is NOT used as
the local SHA-256 integrity value.

---

## 7. Provenance Record

The project currently establishes the following provenance chain:

```text
Official Qwen Repository
        ↓
Qwen/Qwen3.8-27B
        ↓
Pinned revision
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
        ↓
Official config.json
        ↓
Local acquisition
model/official/config.json
        ↓
SHA-256
191e0af232104ed8b65258cf3fb2b842e288008baca7633c11b82a1ac7203aab
        ↓
Byte-level comparison
        ↓
VERIFIED IDENTICAL
```

---

## 8. Verification History

### SET0-T01 — Workspace Verification

Result:

```text
PASS
```

Established that the project workspace was initially empty and Git was
not initialized.

### SET0-T02 — Official Artifact Source

Result:

```text
PASS
```

Established:

* official repository identity
* publisher identity
* pinned revision
* official source record

### SET0-T03 — config.json Acquisition

Result:

```text
PASS
```

Established:

* exact pinned revision used
* `config.json` acquired successfully
* JSON validity
* local SHA-256
* no model weights or unrelated model artifacts downloaded

### SET0-T03-R1 — Artifact Identity Reconciliation

Result:

```text
PASS
```

Established that the apparent difference between:

```text
Qwen3.8-27B
```

and:

```text
Qwen3_5ForConditionalGeneration
qwen3_5
qwen3_5_text
```

is consistent with the documented Qwen3.5 architectural foundation /
implementation lineage rather than evidence of an incorrect repository
artifact.

Canonical evidence:

```text
model/official/IDENTITY-RECONCILIATION.md
```

### SET0-T04 — config.json Integrity Verification

Result:

```text
PASS
```

Established that the local `config.json` is byte-identical to the
official file at the pinned revision.

Canonical evidence:

```text
model/official/CONFIG-VERIFICATION.md
```

---

## 9. Current Provenance Status

```text
Official Repository       VERIFIED
Publisher                 VERIFIED
Pinned Revision           VERIFIED
Artifact Acquisition      VERIFIED
Local SHA-256             VERIFIED
JSON Integrity            VERIFIED
Byte-Level Comparison     VERIFIED
Identity Reconciliation   VERIFIED
```

Overall provenance status:

```text
VERIFIED
```

---

## 10. Scope Boundary

This document establishes provenance and artifact integrity only.

It does NOT establish:

* complete model architecture
* complete layer topology
* parameter count
* tensor inventory
* checkpoint size
* runtime memory requirements
* hardware execution strategy
* CPU/GPU/NPU workload placement
* inference-engine design
* performance characteristics

Those topics are established separately through subsequent SET 0 / SET 1
research tasks.

---

## 11. Source Hierarchy

For model-specific facts, the project uses the following authority order:

```text
1. Official Qwen model artifact
2. Official Qwen Hugging Face repository
3. Official Qwen documentation/model card
4. Local verified copy of the official artifact
5. Derived calculations based on verified artifact data
```

Third-party summaries, community conversions, and unofficial model
descriptions are not authoritative sources for model ground truth.

---

## 12. Reproducibility Rule

The canonical baseline for this research is:

```text
Repository:
Qwen/Qwen3.8-27B

Revision:
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Future research claims referencing the SET 0 model baseline should be
traceable to this revision or explicitly identify a newer verified
revision.

A revision change MUST NOT be treated as an implicit update.

Any new revision must undergo independent provenance and integrity
verification before becoming a new project baseline.

---

## 13. Canonical Evidence References

Primary local evidence:

```text
model/official/SOURCE.md
model/official/config.json
model/official/IDENTITY-RECONCILIATION.md
model/official/CONFIG-VERIFICATION.md
```

This document synthesizes the provenance information from those records
without replacing the underlying evidence.

---

## 14. Final Acceptance

```text
PROVENANCE STATUS:
VERIFIED

ARTIFACT INTEGRITY:
VERIFIED

BASELINE REVISION:
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0

PRIMARY VERIFIED ARTIFACT:
model/official/config.json
```

SET 0 provenance baseline is established and reproducible.