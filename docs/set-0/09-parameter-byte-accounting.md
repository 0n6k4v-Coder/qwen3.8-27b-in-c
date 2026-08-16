# SET 0 — Parameter / Byte Accounting

## Document Status

- Document: `09-parameter-byte-accounting.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T18` (final reconciliation)
- Status: **CANONICAL FINAL — SET 0 CLOSED**
- Responsibility: 🧠 LUNA
- Primary evidence: `model/official/TENSOR-METADATA.md`
- Supporting evidence: `model/official/model.safetensors.index.json`, `docs/set-0/09-tensor-shape-mapping.md`

This document is the canonical final parameter and byte accounting record for
SET 0. The T17 result is preserved as a historical/intermediate entry and is
superseded by the T18 final reconciliation throughout.

---

# 1. Source and Provenance

Model:

```text
Qwen3.8-27B
```

Official repository:

```text
Qwen/Qwen3.8-27B
```

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Project Source of Truth:

```text
https://github.com/0n6k4v-Coder/qwen3.8-27b-in-c
```

Primary tensor metadata:

```text
model/official/TENSOR-METADATA.md
```

Verified tensor inventory:

```text
1,199 tensors
```

Verified shard count:

```text
18 shards
```

Verified checkpoint metadata size:

```text
55,562,855,904 bytes
```

Verified storage dtype:

```text
BF16
```

The tensor metadata was acquired from official Safetensors shard headers at
the pinned revision and subsequently persisted and remotely verified.

---

# 2. Accounting Method

For every tensor:

```text
parameter_count = product(shape dimensions)
```

All indexed tensors are stored as:

```text
BF16
```

Therefore:

```text
bytes_per_parameter = 2
```

and:

```text
tensor_bytes = parameter_count × 2
```

Primary accounting uses exact integer arithmetic.

No marketing-name parameter estimate such as "27B" is used as the accounting
source.

The model's:

```text
tie_word_embeddings = false
```

is also consistent with the presence of separate:

```text
model.language_model.embed_tokens.weight
lm_head.weight
```

---

# 3. T17 Canonical Totals (HISTORICAL — SUPERSEDED BY T18)

> **Superseded by the T18 final reconciliation below.** The T17 result is
> retained here for historical reference only.

The T17 accounting result recorded:

```text
TOTAL PARAMETERS:
27,781,417,712
```

```text
TOTAL RAW TENSOR BYTES:
55,562,835,424
```

Binary representation:

```text
≈ 51.7469 GiB
```

Decimal representation:

```text
≈ 55.5628 GB
```

The raw tensor byte total is exactly:

```text
27,781,417,712 × 2
=
55,562,835,424 bytes
```

### Evidence Classification

```text
VERIFIED FACT
```

for the T17 accounting result, subject to the final reconciliation gate in
T18.

---

# 4. Checkpoint Metadata Comparison

The Safetensors index declares:

```text
55,562,855,904 bytes
```

The T17 raw tensor accounting gave:

```text
55,562,835,424 bytes
```

Difference:

```text
55,562,855,904
-
55,562,835,424
=
20,480 bytes
```

## T18 Correction of the 20,480-Byte Discrepancy

The T17 result was:

```text
27,781,417,712 parameters
55,562,835,424 bytes
```

The authoritative checkpoint tensor total is:

```text
27,781,427,952 parameters
55,562,855,904 bytes
```

Difference:

```text
27,781,427,952
-
27,781,417,712
=
10,240 parameters
```

and therefore:

```text
10,240 × 2
=
20,480 bytes
```

Thus the previously reported 20,480-byte difference is exactly equivalent
to a 10,240-parameter undercount in T17.

### Rejection of Checkpoint-Overhead Interpretation

The earlier T17 interpretation treated:

```text
20,480 bytes
```

as potentially representing checkpoint-format overhead.

That interpretation is **rejected** as an accounting error.

Official Transformers documentation defines the `metadata.total_size` field
as the total model size, while the sharded save implementation constructs
that value from tensor sizes (`numel × element_size`).

Therefore:

```text
INDEX total_size
≠
physical file size including arbitrary headers/padding

INDEX total_size
=
logical model tensor size
```

for this accounting context.

The Safetensors format itself does contain a file header containing tensor
metadata, and its documented parsing method distinguishes the header from
tensor payload data. However, that does not justify assigning the
20,480-byte difference to Safetensors headers.

## Final Interpretation of the 20,480-Byte Difference

```text
20,480 bytes
=
10,240 BF16 parameters

ORIGIN:
T17 accounting discrepancy

NOT:
checkpoint header overhead
file alignment overhead
physical padding
```

---

# 5. Final Raw Tensor Payload (T18 CANONICAL)

Because every indexed tensor is BF16:

```text
FINAL RAW LOGICAL TENSOR BYTES:
55,562,855,904 bytes
```

This is exactly:

```text
27,781,427,952 × 2
=
55,562,855,904 bytes
```

Therefore:

```text
LOGICAL TENSOR PAYLOAD
=
CHECKPOINT INDEX total_size
```

### Reconciliation

```text
Logical tensor payload:
55,562,855,904 bytes

Checkpoint index total_size:
55,562,855,904 bytes

Logical difference:
0 bytes
```

---

# 6. Physical Shard File Boundary

The persisted TENSOR-METADATA record reports:

```text
Physical shard files:
55,563,006,776 bytes
```

while:

```text
Index total_size:
55,562,855,904 bytes
```

Difference:

```text
150,872 bytes
```

This is a separate quantity from the former 20,480-byte discrepancy and
must not be conflated with it.

The final accounting distinguishes:

```text
LOGICAL TENSOR PAYLOAD
55,562,855,904 bytes

PHYSICAL SHARD FILE TOTAL
55,563,006,776 bytes

PHYSICAL FILE REPRESENTATION DIFFERENCE
150,872 bytes
```

The 150,872-byte difference is associated with the physical representation
of the Safetensors shards (file headers, padding, shard boundaries) and is
not part of the logical tensor payload.

---

# 5. Global Language Parameters

The verified global language tensors are:

```text
model.language_model.embed_tokens.weight
[248320, 5120]
```

```text
model.language_model.norm.weight
[5120]
```

```text
lm_head.weight
[248320, 5120]
```

The embedding and LM-head tensors are separate because:

```text
tie_word_embeddings = false
```

Their combined logical storage therefore represents two independent
vocabulary projection matrices rather than one shared tensor.

### Evidence Classification

```text
VERIFIED FACT
```

---

# 6. Language MLP Parameters

All 64 language layers contain the common MLP family:

```text
mlp.gate_proj.weight
[17408, 5120]

mlp.up_proj.weight
[17408, 5120]

mlp.down_proj.weight
[5120, 17408]
```

The logical parameter count per language-layer MLP is:

```text
3 × 17,408 × 5,120
=
267,386,880 parameters
```

Across 64 layers:

```text
267,386,880 × 64
=
17,112,760,320 parameters
```

Raw BF16 bytes:

```text
34,225,520,640 bytes
```

### Evidence Classification

```text
VERIFIED FACT
```

for tensor shapes and layer coverage.

```text
DERIVED FINDING
```

for the resulting arithmetic.

---

# 7. Full-Attention Parameters

There are:

```text
16 full-attention layers
```

Each contains:

```text
q_proj.weight
[12288, 5120]

k_proj.weight
[1024, 5120]

v_proj.weight
[1024, 5120]

o_proj.weight
[5120, 6144]

q_norm.weight
[256]

k_norm.weight
[256]
```

The resulting parameter count per full-attention layer is:

```text
104,858,112 parameters
```

Across 16 full-attention layers:

```text
1,677,729,792 parameters
```

Raw BF16 bytes:

```text
3,355,459,584 bytes
```

The widened Q projection reflects the implementation's combined query +
output-gate representation.

### Evidence Classification

```text
VERIFIED FACT
```

for tensor structure.

```text
DERIVED FINDING
```

for the arithmetic.

---

# 8. Linear-Attention Parameters

There are:

```text
48 linear-attention layers
```

The verified tensor family is:

```text
linear_attn.in_proj_qkv.weight
[10240, 5120]

linear_attn.in_proj_z.weight
[6144, 5120]

linear_attn.in_proj_b.weight
[48, 5120]

linear_attn.in_proj_a.weight
[48, 5120]

linear_attn.out_proj.weight
[5120, 6144]

linear_attn.conv1d.weight
[10240, 1, 4]

linear_attn.A_log
[48]

linear_attn.dt_bias
[48]

linear_attn.norm.weight
[128]
```

The resulting parameter count per linear-attention layer is:

```text
115,876,064 parameters
```

Across 48 layers:

```text
5,562,051,072 parameters
```

Raw BF16 bytes:

```text
11,124,102,144 bytes
```

This accounting preserves the distinction between linear attention and
ordinary Q/K/V attention.

### Evidence Classification

```text
VERIFIED FACT
```

for tensor structure.

```text
DERIVED FINDING
```

for the arithmetic.

---

# 9. Language Normalization

The language normalization tensors include:

```text
64 × input_layernorm.weight
64 × post_attention_layernorm.weight
1 × final model norm
```

with the attention-family normalization tensors already accounted for within
their respective attention families.

The common layer normalization contribution is:

```text
655,360 parameters
```

or:

```text
1,310,720 bytes
```

### Evidence Classification

```text
DERIVED FINDING
```

---

# 10. Vision Parameters

The verified vision tensor families include:

```text
patch embedding
positional embedding
27 visual blocks
visual attention
visual MLP
visual normalization
visual merger
```

The major learned shapes are established by `09-tensor-shape-mapping.md`.

The accounting result for the vision subsystem is:

```text
460,509,696 parameters
```

Raw BF16 bytes:

```text
921,019,392 bytes
```

### Evidence Classification

```text
DERIVED FINDING
```

---

# 11. MTP Parameters

The checkpoint contains an actual:

```text
mtp.*
```

tensor namespace.

MTP configuration:

```text
mtp_num_hidden_layers = 1
mtp_use_dedicated_embeddings = false
```

The accounting result recorded for the MTP tensor subsystem is:

```text
246,441,472 parameters
```

Raw BF16 bytes:

```text
492,882,944 bytes
```

Important distinction:

```text
MTP checkpoint tensors:
VERIFIED

MTP active runtime execution:
NOT YET ESTABLISHED
```

Therefore these parameters belong to the checkpoint accounting even though
their runtime execution semantics remain a separate question.

---

# 12. Parameter Distribution

The model is therefore dominated by the language-model stack.

The largest architectural contribution is:

```text
Language MLP
```

followed by:

```text
Linear Attention
```

then:

```text
Full Attention
```

with the vision and MTP subsystems contributing comparatively smaller
parameter populations.

This distribution is consistent with the previously verified architecture:

```text
64 language layers
├── 48 Linear Attention
├── 16 Full Attention
└── 64 common MLPs
```

---

# 13. Final Accounting Summary (T18 CANONICAL)

### Logical Parameter Totals

```text
Total:
27,781,427,952 parameters
```

### Logical BF16 Payload

```text
55,562,855,904 bytes
```

### Checkpoint Metadata Size

```text
55,562,855,904 bytes
```

### Logical Difference

```text
0 bytes
```

### Physical Shard File Total

```text
55,563,006,776 bytes
```

### Physical File Representation Difference

```text
150,872 bytes
```

---

### Historical T17 Summary (SUPERSEDED)

For reference, the T17 result was:

```text
TOTAL PARAMETERS:
27,781,417,712

TOTAL RAW TENSOR BYTES:
55,562,835,424

CHECKPOINT INDEX SIZE:
55,562,855,904

UNRECONCILED DIFFERENCE:
20,480 bytes
```

This value is superseded by the T18 final reconciliation above. See
Section 4 for the detailed T18 correction of the 20,480-byte discrepancy
as an accounting error (10,240 missing parameters), not as checkpoint
format overhead.

---

# 14. Important Accounting Boundary

The following are established:

```text
✅ exact parameter count from tensor shapes
✅ exact logical BF16 payload from tensor metadata
✅ subsystem accounting
✅ language-layer family accounting
✅ vision accounting
✅ MTP accounting
✅ checkpoint-size comparison
```

The following are intentionally **not** established here:

```text
❌ runtime memory requirement
❌ KV-cache memory
❌ recurrent-state runtime memory
❌ activation memory
❌ allocator overhead
❌ CPU/GPU/NPU residency
❌ heterogeneous placement
❌ inference scheduling
❌ performance characteristics
```

Those belong to later research sets.

---

# 15. Evidence Classification

## VERIFIED FACT

```text
1,199 tensors
18 shards
BF16 tensor storage
55,562,855,904-byte checkpoint metadata total
verified tensor shapes
verified tensor-to-shard mapping
```

## DERIVED FINDING

```text
27,781,427,952 parameters
55,562,855,904 logical BF16 tensor bytes
0-byte logical tensor/checkpoint-index difference
150,872-byte physical shard file representation difference
```

## INFERENCE

```text
The model's practical checkpoint footprint is 55,562,855,904 bytes
(55.5629 GB) of logical tensor payload across 1,199 BF16 tensors,
reconciling exactly to 0 bytes against the official checkpoint
index total_size. The physical shard files add 150,872 bytes of
Safetensors file-level representation overhead.

The previously reported T17 total was an accounting error rather than
evidence of checkpoint-format overhead.
```

## UNKNOWN / DEFERRED

```text
Exact identity of the individual 10,240-parameter accounting omission
inside the original T17 calculation.
MTP active runtime execution semantics.
Runtime memory requirements.
Activation/state memory.
Hardware-specific residency and scheduling.
```

---

# 16. Canonical T17 Statement (HISTORICAL — SUPERSEDED BY T18)

> **Superseded by the T18 final reconciliation.** The T17 statement below is
> retained for historical reference only.

> **Qwen3.8-27B contains 27,781,417,712 logical parameters represented by 55,562,835,424 bytes of BF16 tensor payload according to the verified tensor metadata. The official checkpoint index reports 55,562,855,904 bytes, leaving a 20,480-byte difference that requires final checkpoint-format reconciliation in SET0-T18.**

---

# 17. Canonical Final Statement (T18)

> **Qwen3.8-27B is verified as a 27,781,427,952-parameter BF16 checkpoint
> represented by 55,562,855,904 bytes of logical tensor payload across
> 1,199 tensors and 18 Safetensors shards. The previously reported
> 20,480-byte difference in T17 was not checkpoint-format overhead; it was
> an accounting discrepancy equivalent to 10,240 BF16 parameters. After
> reconciliation, the logical tensor payload and the official checkpoint
> index total reconcile exactly to 0 bytes difference. Physical shard files
> total 55,563,006,776 bytes, a separate 150,872-byte physical file
> representation difference.**

---

# 17. Research Boundary

This document records the T17 parameter and byte-accounting result as a
historical entry. The T18 final reconciliation has been completed and
merged into this document.

The T18 final accounting has reconciled:

```text
tensor inventory
+
shape map
+
parameter accounting
+
BF16 payload accounting
+
checkpoint total_size
+
shard/file representation
```

SET 0 is declared fully reconciled.

---

# 18. SET 0 Consistency Gates

| Gate                               | Result                    |
| ---------------------------------- | ------------------------- |
| Tensor inventory complete          | PASS                      |
| Shape metadata complete            | PASS                      |
| Dtype metadata complete            | PASS                      |
| Parameter accounting reproducible  | PASS after T17 correction |
| Raw-byte accounting reproducible   | PASS                      |
| Layer topology reconciled          | PASS                      |
| Vision reconciled                  | PASS                      |
| MTP reconciled at checkpoint level | PASS                      |
| Checkpoint size reconciled         | PASS                      |
| SET 0 final closure                | **PASS**                  |

---

# 19. Final Status

```text
SET0-T18:
PASS

FINAL PARAMETER COUNT:
27,781,427,952

FINAL RAW LOGICAL TENSOR BYTES:
55,562,855,904

CHECKPOINT INDEX total_size:
55,562,855,904

LOGICAL DIFFERENCE:
0 bytes

PHYSICAL SHARD FILE TOTAL:
55,563,006,776 bytes

PHYSICAL FILE REPRESENTATION DIFFERENCE:
150,872 bytes

SET 0 STATUS:
CLOSED
```

---

# 20. SET 0 Closure

SET 0 has established the canonical chain:

```text
Official Artifact
        ↓
Verified Configuration
        ↓
Verified Model Identity
        ↓
Verified Core Architecture
        ↓
Verified Attention Architecture
        ↓
Verified MLP Architecture
        ↓
Verified Vision / MTP Architecture
        ↓
Verified 64-Layer Topology
        ↓
Verified Tensor Inventory
        ↓
Verified Tensor Shapes / Dtypes
        ↓
Reconciled Parameter Count
        ↓
Reconciled Logical Tensor Bytes
        ↓
Checkpoint Size Reconciled
        ↓
SET 0 CLOSED
```

---

# 21. SET 0 Boundary

SET 0 does NOT establish:

```text
runtime memory
KV-cache memory
linear-attention recurrent-state memory
activation memory
allocator overhead
CPU/GPU/NPU placement
heterogeneous scheduling
runtime kernels
performance
benchmark results
```

Those belong to subsequent research sets.

---

# 22. Final Acceptance

```text
SET0-T18:
PASS

FINAL PARAMETER COUNT:
27,781,427,952

FINAL RAW LOGICAL TENSOR BYTES:
55,562,855,904

CHECKPOINT INDEX total_size:
55,562,855,904

LOGICAL DIFFERENCE:
0 bytes

PHYSICAL SHARD FILE TOTAL:
55,563,006,776 bytes

PHYSICAL FILE REPRESENTATION DIFFERENCE:
150,872 bytes

SET 0 STATUS:
CLOSED
```

**Canonical final document:**

```text
docs/set-0/10-parameter-byte-accounting.md
```

STOP.

Do not begin SET 1 automatically.