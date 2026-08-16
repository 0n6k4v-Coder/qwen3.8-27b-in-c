# SET 0 — Parameter / Byte Accounting

## Document Status

- Document: `09-parameter-byte-accounting.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T18`, final MTP reconciliation: `SET0-CLOSE-FIX-FINALIZE`
- Status: **CANONICAL FINAL — SET 0 MTP ACCOUNTING RECONCILED**
- Responsibility: 🧠 LUNA
- Primary evidence: `model/official/TENSOR-METADATA.md`
- Supporting evidence: `model/official/model.safetensors.index.json`, `docs/set-0/08-tensor-shape-mapping.md`

This document is the canonical SET 0 parameter and logical BF16 byte accounting
record. T17 historical information is retained as historical/superseded.

---

# 1. Source and Method

Model:

```text
Qwen3.8-27B
```

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary tensor metadata:

```text
model/official/TENSOR-METADATA.md
```

Verified inventory:

```text
1,199 tensors
18 shards
BF16
```

Accounting rule:

```text
parameter_count = product(shape dimensions)
bytes = parameter_count × 2
```

---

# 2. Historical T17 Accounting — HISTORICAL / SUPERSEDED

T17 recorded:

```text
TOTAL PARAMETERS:
27,781,417,712

TOTAL RAW TENSOR BYTES:
55,562,835,424
```

The T17 global result was superseded by the independently reconciled raw
metadata total below. Historical T17 values are retained for provenance and
must not be treated as the current canonical totals.

---

# 3. Canonical Global Totals

The complete raw tensor inventory independently establishes:

```text
GLOBAL PARAMETERS:
27,781,427,952

GLOBAL LOGICAL BF16 BYTES:
55,562,855,904
```

The MTP correction does **not** alter these global totals. The global total is
independently established from the complete raw tensor inventory.

Therefore:

```text
27,781,427,952 × 2
=
55,562,855,904 bytes
```

### Evidence Classification

```text
VERIFIED FACT
```

for the raw inventory and metadata totals; the multiplication is a direct
arithmetic derivation.

---

# 4. MTP Tensor Inventory

Raw metadata verifies:

```text
MTP tensors = 15
MTP dtype = BF16
MTP shard = model-00018-of-00018.safetensors
```

The 15 tensors are the exact tensors recorded in
`docs/set-0/08-tensor-shape-mapping.md`.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 5. MTP Parameter / Byte Accounting — CANONICAL

The corrected MTP subsystem total is:

```text
MTP PARAMETERS:
424,699,392

MTP LOGICAL BF16 BYTES:
849,398,784
```

Because MTP tensors are BF16:

```text
424,699,392 × 2
=
849,398,784 bytes
```

### Evidence Classification

```text
DERIVED FINDING
```

for the subtotal arithmetic from the verified raw tensor metadata.

---

# 6. MTP Reconciliation Root Cause

The previous MTP subtotal was:

```text
246,441,472 parameters
492,882,944 bytes
```

The previous subtotal omitted exactly two verified MTP tensors:

```text
mtp.layers.0.mlp.gate_proj.weight
shape: [17408, 5120]
parameters: 89,128,960

mtp.layers.0.mlp.up_proj.weight
shape: [17408, 5120]
parameters: 89,128,960
```

Combined omission:

```text
89,128,960 + 89,128,960
=
178,257,920 parameters
```

Therefore:

```text
246,441,472
+
178,257,920
=
424,699,392 parameters
```

and the corresponding byte correction is:

```text
178,257,920 × 2
=
356,515,840 bytes
```

so:

```text
492,882,944
+
356,515,840
=
849,398,784 bytes
```

### Root Cause Classification

```text
VERIFIED FACT
```

The two omitted tensor identities, shapes, and parameter contributions are
directly supported by `model/official/TENSOR-METADATA.md`.

---

# 7. MTP Runtime Boundary

Checkpoint accounting includes all verified MTP tensors regardless of runtime
activation semantics.

Preserve the following boundary:

```text
MTP checkpoint tensors:
VERIFIED

MTP exact tensor metadata:
VERIFIED

MTP active runtime execution:
UNKNOWN
```

Do not infer ordinary-generation MTP execution from checkpoint presence.

---

# 8. Global Reconciliation After MTP Correction

The corrected MTP subtotal is a correction to subsystem accounting only.

It does **not** change:

```text
27,781,427,952 parameters
55,562,855,904 bytes
```

The reason is that these global values are independently established by the
complete raw tensor inventory, not by adding the previously erroneous MTP
subtotal to other subsystem estimates.

Canonical chain:

```text
RAW METADATA
      ↓
MTP TENSOR MAP
      ↓
MTP ACCOUNTING
      ↓
GLOBAL ACCOUNTING
      ↓
CANONICAL DOCUMENTS
```

---

# 9. T17 Historical Boundary

T17 information remains historical/superseded.

In particular, the previous MTP subtotal:

```text
246,441,472 parameters
492,882,944 bytes
```

must not be used as current accounting. It is retained only as provenance for
the reconciliation described in Section 6.

---

# 10. Final Accounting Summary

```text
MTP TENSORS:
15

MTP PARAMETERS:
424,699,392

MTP LOGICAL BF16 BYTES:
849,398,784

GLOBAL PARAMETERS:
27,781,427,952

GLOBAL LOGICAL BF16 BYTES:
55,562,855,904
```

---

# 11. Evidence Classification Summary

## VERIFIED FACT

```text
1,199 tensors
18 shards
BF16 tensor storage
MTP tensor count = 15
MTP tensor names, shapes, dtype, and shard mapping
The two omitted MTP tensor identities
89,128,960 parameters per omitted tensor
Combined omission = 178,257,920 parameters
Global raw tensor total = 27,781,427,952 parameters
Global logical BF16 bytes = 55,562,855,904
```

## DERIVED FINDING

```text
MTP total = 424,699,392 parameters
MTP logical bytes = 849,398,784
BF16 byte conversions and reconciliation arithmetic
```

## UNKNOWN

```text
MTP active runtime execution semantics
MTP runtime memory
MTP scheduling
Hardware placement
```

---

# 12. Final Acceptance

```text
MTP TENSOR COUNT:
15

MTP PARAMETERS:
424,699,392

MTP LOGICAL BF16 BYTES:
849,398,784

GLOBAL PARAMETERS:
27,781,427,952

GLOBAL LOGICAL BF16 BYTES:
55,562,855,904

ROOT CAUSE:
VERIFIED

MTP EXACT METADATA:
VERIFIED

MTP ACTIVE RUNTIME EXECUTION:
UNKNOWN

T17:
HISTORICAL / SUPERSEDED

SET 0 MTP ACCOUNTING:
RECONCILED
```
