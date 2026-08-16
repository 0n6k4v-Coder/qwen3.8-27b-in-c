# Qwen3.8-27B Inference Engine — Project Roadmap

## Document Status

* Document: `ROADMAP.md`
* Role: Project Control / Roadmap State
* Project Source of Truth: `https://github.com/0n6k4v-Coder/qwen3.8-27b-in-c`
* Last control update: `2026-08-16`
* Current integrated branch: `main`
* Current integrated commit: `3547193`
* Current project phase: **SET 0 formally closed; SET 1 active and executing**
* SET 1 execution: **ACTIVE**

---

# 1. Overall Project Roadmap

```text
PROJECT
│
├── SET 0 — Model Reconnaissance
│   ✅ TECHNICAL COMPLETE
│   ✅ MERGED TO MAIN
│   ✅ ROADMAP PERSISTED
│   ✅ FORMAL CONTROL CLOSURE
│
├── SET 1 — Tensor / Byte-Level Audit
│   🟢 ACTIVE
│
├── SET 2 — Hardware Reconnaissance
│   🔒 NOT STARTED
│
├── SET 3 — Operator / Computation Model
│   🔒 NOT STARTED
│
├── SET 4 — Runtime Memory Model
│   🔒 NOT STARTED
│
├── SET 5 — Reference Inference Engine
│   🔒 NOT STARTED
│
├── SET 6 — Correctness / Validation
│   🔒 NOT STARTED
│
├── SET 7 — Memory-Constrained Execution
│   🔒 NOT STARTED
│
├── SET 8 — Streaming
│   🔒 NOT STARTED
│
├── SET 9 — CPU Optimization
│   🔒 NOT STARTED
│
├── SET 10 — Intel Arc GPU
│   🔒 NOT STARTED
│
├── SET 11 — Intel NPU
│   🔒 NOT STARTED
│
├── SET 12 — Heterogeneous Execution
│   🔒 NOT STARTED
│
├── SET 13 — Scheduling / Workload Placement
│   🔒 NOT STARTED
│
├── SET 14 — End-to-End Optimization
│   🔒 NOT STARTED
│
└── SET 15 — Final Benchmark / Validation
    🔒 NOT STARTED
```

## Overall Responsibility Model

| SET    | Objective                                                                        | Status               | Responsibility        |
| ------ | -------------------------------------------------------------------------------- | -------------------- | --------------------- |
| SET 0  | Establish verified model/source-of-truth foundation                              | ✅ TECHNICAL COMPLETE | 🧠 LUNA + 🛠 EXECUTOR |
| SET 1  | Establish verified tensor, parameter, logical-byte, and checkpoint-storage truth | 🟢 ACTIVE            | 🧠 LUNA               |
| SET 2  | Characterize target hardware capabilities/constraints                            | 🔒 NOT STARTED       | 🧠 LUNA               |
| SET 3  | Define operator and computation model                                            | 🔒 NOT STARTED       | 🧠 LUNA               |
| SET 4  | Define runtime memory model                                                      | 🔒 NOT STARTED       | 🧠 LUNA               |
| SET 5  | Build correctness-first reference inference engine                               | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 6  | Validate numerical/correctness behavior                                          | 🔒 NOT STARTED       | 🧠 LUNA + 🛠 EXECUTOR |
| SET 7  | Enable execution under memory constraints                                        | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 8  | Introduce streaming/data movement strategy                                       | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 9  | Optimize CPU execution                                                           | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 10 | Intel Arc GPU execution                                                          | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 11 | Intel NPU execution                                                              | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 12 | Coordinate heterogeneous execution                                               | 🔒 NOT STARTED       | 🧠 LUNA               |
| SET 13 | Scheduling/workload placement                                                    | 🔒 NOT STARTED       | 🧠 LUNA               |
| SET 14 | End-to-end system optimization                                                   | 🔒 NOT STARTED       | 🧠 LUNA → 🛠 EXECUTOR |
| SET 15 | Final benchmark and validation                                                   | 🔒 NOT STARTED       | 🧠 LUNA               |

---

# 2. Current SET Roadmaps

## SET 0 — Model Reconnaissance

**Objective:** Establish a verified, artifact-grounded model description before implementation work begins.

### Atomic Task State

```text
SET 0
│
├── SET0-T16 — Tensor Metadata
│   🛠 EXECUTOR
│   ✅ PASS
│   📌 PERSISTED
│   ☁ PUSHED
│   🔎 REMOTE VERIFIED
│
├── SET0-T17 — Parameter / Byte Accounting
│   🧠 LUNA
│   ⚠ SUPERSEDED
│
├── SET0-T18 — Final Reconciliation
│   🧠 LUNA
│   ✅ PASS
│   📌 PERSISTED
│
├── SET0-CLOSE-FIX-FINALIZE — MTP Accounting Reconciliation
│   🧠 LUNA
│   ✅ PASS
│   📌 PERSISTED
│   ☁ PUSHED
│   🔎 REMOTE VERIFIED
│
└── SET0-MERGE-TO-MAIN — Integrate SET 0 Closure
    🛠 EXECUTOR
    ✅ PASS
    ☁ MERGED
    🔎 REMOTE VERIFIED
    PR: #1
    Merge commit: `c0c6244abcea30255ead52fdfdea65c559f8eed1`
```

### SET 0 Technical Acceptance

```text
MTP tensor count:
15

MTP parameters:
424,699,392

MTP logical BF16 bytes:
849,398,784

Global parameters:
27,781,427,952

Global logical BF16 bytes:
55,562,855,904
```

### MTP Root Cause

The previous MTP subtotal omitted exactly two verified tensors:

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
178,257,920 parameters
```

Historical subtotal:

```text
246,441,472 parameters
492,882,944 bytes
```

Corrected subtotal:

```text
424,699,392 parameters
849,398,784 bytes
```

### SET 0 Evidence Boundary

The historical SET 0 documents may reference the removed:

```text
model/official/TENSOR-METADATA.md
```

This file is intentionally removed and must not be recreated.

The provenance issue is retained as historical documentation state and does not alter the verified SET 0 accounting result.

### SET 0 Boundary

Verified:

```text
MTP checkpoint tensors             VERIFIED
MTP exact tensor metadata          VERIFIED
MTP accounting                     RECONCILED
Global accounting                  VERIFIED
```

Still intentionally unknown:

```text
MTP active runtime execution
MTP runtime scheduling
MTP runtime memory behavior
Hardware-specific placement
```

### Current SET 0 Control State

```text
Technical work:                    ✅ COMPLETE
Branch integration:                ✅ MERGED TO MAIN
Technical closure evidence:        ✅ VERIFIED
Roadmap persistence:               ✅ VERIFIED
Formal SET 0 closure:              ✅ COMPLETE
```

---

## SET 1 — Tensor / Byte-Level Audit

**Objective:** Establish a verified, reproducible, tensor-level and byte-level representation of the pinned `Qwen/Qwen3.8-27B` checkpoint so that subsequent SETs can treat tensor structure, parameter counts, logical bytes, shard placement, and checkpoint-storage facts as established project evidence without re-inferring them.

### SET 1 Technical Boundary

SET 1 establishes:

```text
Official Checkpoint
        ↓
RAW Metadata
        ↓
Tensor Truth
        ↓
Parameter Truth
        ↓
Logical Byte Truth
        ↓
Shard / Storage Truth
        ↓
Canonical SET 1 Evidence
        ↓
Downstream Input Contract
```

SET 1 does not establish:

```text
❌ Hardware capability
❌ RAM / GPU / NPU capacity
❌ Runtime memory
❌ KV-cache memory
❌ Activation memory
❌ Runtime MTP execution
❌ Operator implementation
❌ Runtime scheduling
❌ CPU / GPU / NPU placement
❌ Performance characteristics
❌ Inference execution
```

These belong to downstream SETs.

### SET 1 Status

```text
🟢 ACTIVE

Technical Evidence Track:
SET1-READINESS-GATE ✅ PASS
SET1-T1.1          ✅ PASS
SET1-T1.2          ✅ PASS
SET1-T1.4          ✅ PASS
SET1-T1.5-R1       ✅ PASS
SET1-T1.6          ✅ PASS
SET1-T1.7          ✅ PASS
SET1-T1.8          ✅ PASS

Current:
🔜 SET1-T1.9

Remaining:
🔒 SET1-CLOSE
```

### SET 1 Responsibility Model

```text
Primary owner:
🧠 LUNA

EXECUTOR:
🛠 owns explicitly delegated environment access,
execution, evidence acquisition, filesystem operations,
and persistence operations.

ORCHESTRATOR:
🔄 coordination/control only.
```

### SET 1 Atomic Tasks

```text
SET1-READINESS-GATE — SET 1 Readiness
🧠 LUNA
✅ PASS
Dependency: SET 0 technical closure + required roadmap state

Objective:
Confirm model identity, pinned revision, source-of-truth,
upstream artifact, evidence path, prerequisites, and blockers.

Result:
READY FOR SET 1
```

```text
SET1-T1.1 — Raw Metadata Acquisition
🛠 EXECUTOR
✅ PASS
Dependency: SET1-READINESS-GATE PASS

Objective:
Acquire and persist raw checkpoint metadata directly from the
pinned official artifact without downloading full tensor payloads.

Required evidence:
config.json
manifest.json
model.safetensors.index.json
18 Safetensors shard headers

Execution boundary:
Evidence acquisition only.
```

```text
SET1-T1.2 — Raw Metadata Verification
🧠 LUNA
✅ PASS
Dependency: SET1-T1.1 PASS

Objective:
Verify provenance, pinned revision, completeness, integrity,
header structure, index presence, and source-of-truth validity.

Result:
RAW evidence = TRUSTWORTHY
```

```text
SET1-T1.4 — Tensor Shape / Dtype / Offset Audit
🧠 LUNA
✅ PASS
Dependency: SET1-T1.2 PASS

Objective:
Establish canonical tensor structural truth.

Verified scope:
tensor names
tensor shapes
tensor dtypes
data_offsets
tensor-to-shard assignments
RAW ↔ official index reconciliation

Verified inventory:
1,199 tensors
18 shards
0 missing
0 duplicate
0 unassigned
```

```text
SET1-T1.5-R1 — Tensor Parameter Reconstruction
🧠 LUNA
✅ PASS
Dependency: SET1-T1.4 PASS

Objective:
Reconstruct parameter counts directly from RAW tensor shapes.

Accounting rule:
parameter_count = product(shape dimensions)

Required reconciliation:
tensor
→ shard
→ subsystem
→ global

Special groups:
MTP
embed_tokens
lm_head

Verified global:
27,781,427,952 parameters
```

```text
SET1-T1.6 — Tensor Logical Byte Accounting
🧠 LUNA
✅ PASS
Dependency: SET1-T1.5-R1 PASS

Objective:
Reconstruct logical tensor bytes from RAW shape + RAW dtype.

Accounting rule:
logical_bytes =
parameter_count × bytes_per_element(dtype)

Verified dtype:
BF16

Verified width:
2 bytes / element

Verified global:
55,562,855,904 logical bytes

Boundary:
logical checkpoint bytes only
not runtime memory
```

```text
SET1-T1.7 — Final Evidence Reconciliation
🧠 LUNA
✅ PASS
Dependency: SET1-T1.6 PASS

Objective:
Reconcile the complete T1.4 → T1.5-R1 → T1.6 evidence chain.

Required checks:
RAW foundation
parameter reconciliation
byte reconciliation
MTP reconciliation
embedding reconciliation
LM-head reconciliation
SET 1 document consistency
SET 0 historical cross-check
provenance consistency

Result:
VERIFIED PASS
READY FOR CLOSURE CONSIDERATION
```

```text
SET1-T1.8 — Checkpoint Storage / Physical Layout Reconciliation
🧠 LUNA → 🛠 EXECUTOR
✅ PASS
Dependency: SET1-T1.7 PASS

Objective:
Establish the verified boundary between logical tensor bytes
and physical checkpoint storage representation.

Scope:
header structure
tensor payload regions
data-offset spans
per-shard storage layout
known storage overhead
logical-versus-physical accounting boundary

Required classification:
KNOWN
DERIVED FINDING
UNKNOWN

Hard rule:
Do not represent logical tensor bytes as physical checkpoint
file size without evidence.

Do not enter runtime memory analysis.
```

```text
SET1-T1.9 — SET 1 Boundary / Completeness Audit
🧠 LUNA
✅ PASS
Technical Result:
COMPLETE
Dependency: SET1-T1.8 PASS

Objective:
Determine whether SET 1 has completed everything required
by its technical objective and boundary.

Coverage:
model identity
pinned revision
tensor inventory
tensor shapes
tensor dtypes
tensor offsets
shard assignment
parameter counts
logical bytes
MTP
embedding
LM head
aggregation
storage boundary
provenance
known / unknown boundary

Negative boundary:
Confirm that runtime, hardware, performance,
scheduling, and implementation claims were not introduced.

Result:
SET 1 TECHNICAL EVIDENCE = COMPLETE / INCOMPLETE
```

```text
SET1-CLOSE — Formal SET 1 Acceptance
🧠 LUNA
🔒 NOT STARTED
Dependency: SET1-T1.9 COMPLETE

Objective:
Perform the formal acceptance decision for SET 1.

Acceptance requires:
all required atomic tasks PASS
no unresolved contradiction
no blocking UNKNOWN
scope satisfied
boundary satisfied

Result:
SET 1 CLOSED / NOT CLOSED
```

### SET 1 Current Verified Facts

```text
Model:
Qwen/Qwen3.8-27B

Pinned revision:
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0

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

MTP:
15 tensors
424,699,392 parameters
849,398,784 logical bytes

embed_tokens:
1,271,398,400 parameters
2,542,796,800 logical bytes

lm_head:
1,271,398,400 parameters
2,542,796,800 logical bytes
```

### SET 1 Output Contract

When SET 1 is formally closed, downstream SETs may treat the following as established project evidence:

```text
Tensor Truth
    │
    ├── tensor inventory
    ├── tensor shapes
    ├── tensor dtypes
    ├── tensor offsets
    └── shard assignment
          │
          ▼
Parameter Truth
    │
    ├── per-tensor parameters
    ├── per-shard parameters
    ├── subsystem parameters
    ├── MTP parameters
    ├── embedding parameters
    ├── LM-head parameters
    └── global parameters
          │
          ▼
Byte Truth
    │
    ├── logical tensor bytes
    ├── per-shard logical bytes
    ├── subsystem logical bytes
    ├── MTP logical bytes
    ├── embedding bytes
    ├── LM-head bytes
    └── global logical bytes
          │
          ▼
Storage Truth
    │
    └── verified checkpoint storage / shard-layout boundary
```

### SET 1 Completion Boundary

SET 1 is not considered complete merely because a numerical
reconciliation task passes.

Formal completion requires:

```text
T1.7
  ↓
T1.8
  ↓
T1.9
  ↓
SET1-CLOSE
  ↓
SET 1 CLOSED
```

The purpose of formal closure is to freeze the verified checkpoint
representation as a downstream input contract.

---

## SET 2 — Hardware Reconnaissance

**Objective:** Establish verified CPU/GPU/NPU capabilities and constraints for the target machine.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 1 formal closure / verified tensor and storage facts available

---

## SET 3 — Operator / Computation Model

**Objective:** Translate verified model structure into an explicit operator/computation model without prematurely assuming runtime behavior.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 0 + SET 1 verified model/tensor evidence

---

## SET 4 — Runtime Memory Model

**Objective:** Build a verified runtime memory model covering weights, activations, state, and relevant execution buffers.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 2 + SET 3

---

## SET 5 — Reference Inference Engine

**Objective:** Implement a correctness-first reference inference path.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 3 + SET 4

---

## SET 6 — Correctness / Validation

**Objective:** Establish correctness against authoritative/reference outputs.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA + 🛠 EXECUTOR

**Dependency:** SET 5

---

## SET 7 — Memory-Constrained Execution

**Objective:** Make inference practical under the target memory budget.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 6

---

## SET 8 — Streaming

**Objective:** Introduce streaming and controlled data movement where required by memory constraints.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 7

---

## SET 9 — CPU Optimization

**Objective:** Optimize CPU execution after correctness and memory behavior are established.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 8 + correctness baseline

---

## SET 10 — Intel Arc GPU

**Objective:** Develop and validate Intel Arc GPU execution paths.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 9 baseline + GPU readiness

---

## SET 11 — Intel NPU

**Objective:** Develop and validate Intel NPU execution paths.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 10 / NPU-specific readiness

---

## SET 12 — Heterogeneous Execution

**Objective:** Coordinate CPU, Intel Arc GPU, and Intel NPU execution as a combined system.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 9 + SET 10 + SET 11

---

## SET 13 — Scheduling / Workload Placement

**Objective:** Determine verified workload placement and scheduling strategies across available compute resources.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 12

---

## SET 14 — End-to-End Optimization

**Objective:** Optimize the complete inference pipeline, including computation, memory, data movement, and placement.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠→🛠 LUNA → EXECUTOR

**Dependency:** SET 13

---

## SET 15 — Final Benchmark / Validation

**Objective:** Produce the final correctness, memory, and performance validation of the complete system.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 14

---

# 3. Current Control State

```text
CURRENT SET:
SET 1 — Tensor / Byte-Level Audit

CURRENT TECHNICAL STATE:
COMPLETE THROUGH SET1-T1.9

CURRENT NEXT TASK:
SET1-CLOSE

NEXT TASK OWNER:
🧠 LUNA

EXECUTION SUPPORT:
🛠 EXECUTOR

SET 1:
🟢 ACTIVE

SET 1 TECHNICAL EVIDENCE:
COMPLETE

SET 1 FORMAL CLOSURE:
🔜 NEXT

SET 2:
🔒 NOT STARTED

ROADMAP.md:
PERSISTED AFTER THIS TASK
```

---

# 4. Control Loop

```text
CURRENT RESULT
      ↓
UPDATE CONVERSATION ROADMAP
      ↓
NEXT ATOMIC TASK
      ↓
RESPONSIBILITY / OWNERSHIP GATE
      ↓
READINESS GATE
      ↓
GENERATE ROLE-SPECIFIC PROMPT
      ↓
EXECUTE
      ↓
PERSIST EVIDENCE WHEN REQUIRED
      ↓
COMMIT
      ↓
PUSH
      ↓
REMOTE CROSS-CHECK
      ↓
LUNA VERIFICATION / INTERPRETATION
      ↓
UPDATE CONVERSATION ROADMAP
      ↓
NEXT ATOMIC TASK
```

## Hard Rules

1. Do not begin a downstream SET while its prerequisite gate is unresolved.
2. A technical task is not accepted solely from an Executor `PASS` claim.
3. Durable Executor evidence must be persisted and remotely cross-checkable when persistence is required.
4. LUNA owns research, analysis, interpretation, architecture reasoning, sequencing, and acceptance decisions.
5. EXECUTOR owns local environment access, file operations, terminal execution, downloads, measurements, and persistence operations explicitly delegated to it.
6. The official upstream model artifact remains the authoritative source for model facts.
7. The GitHub repository is the shared persistent project Source of Truth.
8. `UNKNOWN` must not be silently converted into an assumption.
9. SET 1 remains ACTIVE until formal closure.
10. SET 2 must not begin before `SET1-CLOSE` is accepted.
11. Every Atomic Task has exactly one primary responsibility owner.
12. ORCHESTRATOR may coordinate and enforce control flow but must not replace LUNA's technical research, interpretation, or acceptance authority.

---

# 5. Persistence / Provenance Rules

For every durable Executor artifact:

```text
LOCAL EXECUTION
    ↓
LOCAL VERIFICATION
    ↓
COMMIT
    ↓
PUSH
    ↓
REMOTE VERIFICATION
    ↓
LUNA CROSS-CHECK
```

For model facts:

```text
OFFICIAL QWEN / HUGGING FACE ARTIFACT
        ↓
PROJECT EVIDENCE ACQUISITION
        ↓
PERSISTED RAW EVIDENCE
        ↓
VERIFIED PROJECT DOCUMENT
        ↓
DERIVED ACCOUNTING / ANALYSIS
```

`ROADMAP.md` persistence is intentionally batched.

LUNA-owned tasks normally update the Conversation Roadmap only.

Before a dedicated Executor task begins after accumulated roadmap changes:

```text
PULL / SYNC
    ↓
UPDATE ROADMAP.md
    ↓
REVIEW DIFF
    ↓
COMMIT
    ↓
PUSH
    ↓
REMOTE VERIFY
    ↓
BEGIN DEDICATED EXECUTOR TASK
```

Executor must use the exact roadmap state supplied by LUNA and must not redesign it.

---

# 6. Status Legend

```text
✅ PASS          completed and accepted
❌ FAIL          failed acceptance criteria
⚠ PARTIAL       partially completed / unresolved
⏸ BLOCKED       cannot proceed because dependency is unresolved
🔒 NOT STARTED   no execution has begun
🔜 NEXT          next atomic task selected by control layer
📌 PERSISTED     durable artifact committed to repository
☁ PUSHED        commit pushed to remote
🔎 REMOTE VERIFIED remote state independently checked
```

---

# 7. Current Stop Condition

```text
Current stop condition:

SET 0 technical integration is complete and formally closed.

SET 1 technical evidence is VERIFIED through:
SET1-T1.8 ✅ PASS

The next selected task is:
SET1-T1.9 — SET 1 Boundary / Completeness Audit

SET1-T1.9 has not started.
SET1-CLOSE has not started.
SET 2 has not started.

Do not begin SET 2 until SET 1 formal closure is accepted.

ROADMAP.md persistence:
PERSISTED AFTER THIS TASK
```
