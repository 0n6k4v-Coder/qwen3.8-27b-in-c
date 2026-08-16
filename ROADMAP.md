# Qwen3.8-27B Inference Engine — Project Roadmap

## Document Status

- Document: `ROADMAP.md`
- Role: Project Control / Roadmap State
- Project Source of Truth: `https://github.com/0n6k4v-Coder/qwen3.8-27b-in-c`
- Last control update: 2026-08-16
- Current integrated branch: `main`
- Current integrated commit: `c0c6244abcea30255ead52fdfdea65c559f8eed1`
- Current project phase: **SET 0 closure integrated and formally closed; SET 1 readiness gate pending**
- SET 1 execution: **NOT STARTED**

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
│   🔒 NOT STARTED
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

| SET | Objective | Status | Responsibility |
|---|---|---|---|
| SET 0 | Establish verified model/source-of-truth foundation | ✅ TECHNICAL COMPLETE | 🧠 LUNA + 🛠 EXECUTOR |
| SET 1 | Audit tensor representation and byte-level properties | 🔒 NOT STARTED | 🧠 LUNA |
| SET 2 | Characterize target hardware capabilities/constraints | 🔒 NOT STARTED | 🧠 LUNA |
| SET 3 | Define operator and computation model | 🔒 NOT STARTED | 🧠 LUNA |
| SET 4 | Define runtime memory model | 🔒 NOT STARTED | 🧠 LUNA |
| SET 5 | Build correctness-first reference inference engine | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 6 | Validate numerical/correctness behavior | 🔒 NOT STARTED | 🧠 LUNA + 🛠 EXECUTOR |
| SET 7 | Enable execution under memory constraints | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 8 | Introduce streaming/data movement strategy | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 9 | Optimize CPU execution | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 10 | Intel Arc GPU execution | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 11 | Intel NPU execution | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 12 | Coordinate heterogeneous execution | 🔒 NOT STARTED | 🧠 LUNA |
| SET 13 | Scheduling/workload placement | 🔒 NOT STARTED | 🧠 LUNA |
| SET 14 | End-to-end system optimization | 🔒 NOT STARTED | 🧠 LUNA → 🛠 EXECUTOR |
| SET 15 | Final benchmark and validation | 🔒 NOT STARTED | 🧠 LUNA |

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

### SET 0 Evidence Files

```text
model/official/SOURCE.md
model/official/config.json
model/official/model.safetensors.index.json
model/official/TENSOR-METADATA.md

docs/set-0/01-artifact-provenance.md
docs/set-0/02-model-identity.md
docs/set-0/03-core-architecture.md
docs/set-0/04-attention-architecture.md
docs/set-0/05-mlp-architecture.md
docs/set-0/06-vision-and-mtp.md
docs/set-0/07-layer-topology.md
docs/set-0/08-tensor-shape-mapping.md
docs/set-0/09-parameter-byte-accounting.md
```

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

**Objective:** Produce a raw, reproducible, byte-level tensor audit grounded directly in the pinned official checkpoint metadata, with explicit provenance and cross-checkability.

### Status

```text
🔒 NOT STARTED
```

### Responsibility

```text
Primary owner: 🧠 LUNA
Execution dependency: 🧠→🛠 LUNA → EXECUTOR
```

### Entry Conditions

SET 1 must not begin until:

```text
[x] ROADMAP.md is persisted to the shared repository
[x] ROADMAP.md is remotely verified
[x] SET 0 remains consistent with main
[ ] SET 1 readiness gate passes
```

### Planned Initial Evidence Acquisition

The first SET 1 evidence task is expected to acquire **raw checkpoint metadata** directly from the pinned official artifact without downloading full tensor payloads.

Planned acquisition scope:

```text
config.json
model.safetensors.index.json
Safetensors shard headers
```

The expected method is header-only / ranged acquisition where feasible, followed by independent cross-checking against the index artifact.

This is **planned scope only**. No SET 1 execution has started.

### Current SET 1 Tasks

```text
SET1-READINESS-GATE
🔒 NOT STARTED
🧠 LUNA
Dependency: SET 0 technical closure + ROADMAP persistence

SET1-RAW-METADATA-ACQUISITION
🔒 NOT STARTED
🛠 EXECUTOR under 🧠 LUNA direction
Dependency: SET1-READINESS-GATE PASS
```

---

## SET 2 — Hardware Reconnaissance

**Objective:** Establish verified CPU/GPU/NPU capabilities and constraints for the target machine.

**Status:** 🔒 NOT STARTED

**Responsibility:** 🧠 LUNA

**Dependency:** SET 1 completion / required tensor facts available

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
SET 0 — Model Reconnaissance

CURRENT TECHNICAL STATE:
COMPLETE

INTEGRATION STATE:
MERGED TO MAIN

MAIN HEAD:
c0c6244abcea30255ead52fdfdea65c559f8eed1

CURRENT NEXT TASK:
SET1-READINESS-GATE

NEXT TASK OWNER:
🧠 LUNA

NEXT CONTROL TASK AFTER ROADMAP PERSISTENCE:
(no pending transition — SET 0 formally closed; gate is SET1-READINESS-GATE)

SET 1 EXECUTION:
NOT STARTED

RAW METADATA ACQUISITION:
NOT STARTED
```

---

# 4. Control Loop

```text
CURRENT RESULT
      ↓
ROADMAP UPDATE
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
PERSIST EVIDENCE
      ↓
COMMIT
      ↓
PUSH
      ↓
REMOTE CROSS-CHECK
      ↓
LUNA VERIFICATION / INTERPRETATION
      ↓
ROADMAP UPDATE
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
9. SET 1 execution must remain `NOT STARTED` until its readiness gate passes.

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

---

# 6. Status Legend

```text
✅ PASS        completed and accepted
❌ FAIL        failed acceptance criteria
⚠ PARTIAL     partially completed / unresolved
⏸ BLOCKED     cannot proceed because dependency is unresolved
🔒 NOT STARTED no execution has begun
🔜 NEXT       next atomic task selected by control layer
📌 PERSISTED   durable artifact committed to repository
☁ PUSHED      commit pushed to remote
🔎 REMOTE VERIFIED remote state independently checked
```

---

# 7. Current Stop Condition

```text
SET 0 technical integration is complete.
ROADMAP persistence is the current control task.
SET 1 execution has NOT started.
No raw metadata acquisition has been performed as SET 1 work.
```

**Do not begin SET 1 until `ROADMAP-PERSIST-MAIN` and the subsequent `SET1-READINESS-GATE` both pass.**