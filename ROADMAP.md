# Qwen3.8-27B Inference Engine — Project Roadmap

## Document Status

* Document: `ROADMAP.md`
* Role: Project Control / Roadmap State
* Project Source of Truth: `https://github.com/0n6k4v-Coder/qwen3.8-27b-in-c`
* Last control update: `2026-08-19`
* Current integrated branch: `main`
* Current integrated commit: `8b5b7d8235a86c552e01698f7f40dbe809b954f0`
* Current integrated commit semantics: The repository HEAD immediately preceding the most recent commit that modified ROADMAP.md (i.e., the parent of the ROADMAP-persistence commit). This is non-self-referential: the field references the parent commit, not the commit containing the field. It is technically stable because the parent SHA is immutable. (Established by SET2-T2.9-R3.)
* Current project phase: **SET 0 formally closed; SET 1 formally closed; SET 2 formally closed; SET 3 formally closed**
* SET 1 execution: **CLOSED**
* SET 2 execution: **CLOSED**
* Current control task: **SET3-CLOSE**

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
│   ✅ TECHNICAL COMPLETE
│   ✅ EVIDENCE COMPLETE
│   ✅ FORMAL CONTROL CLOSURE
│
├── SET 2 — Hardware Reconnaissance
│   ✅ TECHNICAL COMPLETE
│   ✅ EVIDENCE COMPLETE
│   ✅ FORMAL CONTROL CLOSURE
│
├── SET 3 — Operator / Computation Model
│   ✅ TECHNICAL COMPLETE
│   ✅ EVIDENCE COMPLETE
│   ✅ FORMAL CONTROL CLOSURE
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

| SET    | Objective                                                                                            | Status                   | Responsibility        |
| ------ | ---------------------------------------------------------------------------------------------------- | ------------------------ | --------------------- |
| SET 0  | Establish verified model/source-of-truth foundation                                                  | ✅ CLOSED                 | 🧠 LUNA + 🛠 EXECUTOR |
| SET 1  | Establish verified tensor, parameter, logical-byte, and checkpoint-storage truth                     | ✅ CLOSED                 | 🧠 LUNA               |
|| SET 2  | Establish verified hardware capability, constraints, software accessibility, and data-movement truth | ✅ CLOSED                 | 🧠 LUNA + 🛠 EXECUTOR |
|| SET 3  | Define operator and computation model                                                                | ✅ CLOSED                 | 🧠 LUNA               |
| SET 4  | Define runtime memory model                                                                          | 🔒 NOT STARTED           | 🧠 LUNA               |
| SET 5  | Build correctness-first reference inference engine                                                   | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 6  | Validate numerical/correctness behavior                                                              | 🔒 NOT STARTED           | 🧠 LUNA + 🛠 EXECUTOR |
| SET 7  | Enable execution under memory constraints                                                            | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 8  | Introduce streaming/data movement strategy                                                           | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 9  | Optimize CPU execution                                                                               | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 10 | Intel Arc GPU execution                                                                              | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 11 | Intel NPU execution                                                                                  | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 12 | Coordinate heterogeneous execution                                                                   | 🔒 NOT STARTED           | 🧠 LUNA               |
| SET 13 | Scheduling/workload placement                                                                        | 🔒 NOT STARTED           | 🧠 LUNA               |
| SET 14 | End-to-end system optimization                                                                       | 🔒 NOT STARTED           | 🧠 LUNA → 🛠 EXECUTOR |
| SET 15 | Final benchmark and validation                                                                       | 🔒 NOT STARTED           | 🧠 LUNA               |

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
✅ FORMALLY CLOSED
```

### Responsibility

```text
Primary owner: 🧠 LUNA
Execution dependency: 🧠→🛠 LUNA → EXECUTOR
```

### Atomic Task State

```text
SET1-READINESS-GATE
✅ PASS
🧠 LUNA
Dependency: SET 0 technical closure + ROADMAP persistence

SET1-T1.1 — Raw Metadata Acquisition
✅ PASS
🛠 EXECUTOR
Dependency: SET1-READINESS-GATE PASS

SET1-T1.2 — Raw Metadata Verification
✅ PASS
🧠 LUNA
Dependency: SET1-T1.1 PASS

SET1-T1.4 — Tensor Shape / Dtype / Offset Audit
✅ PASS
🧠 LUNA
Dependency: SET1-T1.2 PASS

SET1-T1.5-R1 — Tensor Parameter Reconstruction
✅ PASS
🧠 LUNA
Dependency: SET1-T1.4 PASS

SET1-T1.6 — Tensor Byte Accounting
✅ PASS
🧠 LUNA
Dependency: SET1-T1.5-R1 PASS

SET1-T1.7 — Final Evidence Reconciliation
✅ PASS
🧠 LUNA
Dependency: SET1-T1.6 PASS

SET1-T1.8 — Checkpoint Storage / Physical Layout Reconciliation
✅ PASS
🧠 LUNA → 🛠 EXECUTOR
Dependency: SET1-T1.7 PASS

SET1-T1.9 — SET 1 Boundary / Completeness Audit
✅ PASS / COMPLETE
🧠 LUNA
Dependency: SET1-T1.8 PASS

SET1-CLOSE — Formal SET 1 Acceptance
✅ CLOSED
🧠 LUNA
Dependency: SET1-T1.9 COMPLETE
```

### SET 1 Technical Acceptance

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

### SET 1 Boundary

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
❌ Loader implementation
❌ Memory-mapping implementation
❌ Streaming strategy
❌ Runtime optimization
```

These belong to downstream SETs.

### SET 1 Evidence Files

```text
docs/set-1/01-raw-metadata-verification.md
docs/set-1/02-parameter-reconstruction.md
docs/set-1/03-tensor-byte-accounting.md
docs/set-1/04-checkpoint-storage-layout-reconciliation.md
docs/set-1/05-set1-boundary-completeness-audit.md
```

### SET 1 Final Control State

```text
Technical work:                    ✅ COMPLETE
Technical evidence:                ✅ COMPLETE
Technical completeness:            ✅ VERIFIED
Formal acceptance:                 ✅ CLOSED
Downstream input contract:         ✅ ESTABLISHED
SET 2 prerequisite:                ✅ SATISFIED
```

---

## SET 2 — Hardware Reconnaissance

**Objective:** Establish a verified Hardware Truth Layer for the actual target machine and authoritative hardware/software sources, identifying hardware resources, compute capabilities, system-memory characteristics, accelerator availability, driver/runtime accessibility, and data-movement constraints without making workload-placement, scheduling, optimization, or runtime-implementation decisions.

### Status

```text
✅ PASS

SET 2 execution:
✅ CLOSED

SET2-READINESS-GATE:
✅ PASS

SET2-T2.1:
✅ PASS

SET2-T2.1-R1:
✅ PASS

SET2-T2.2:
✅ PASS

SET2-T2.2-R1:
✅ PASS

SET2-T2.3:
✅ PASS

SET2-T2.3-R1:
✅ PASS

SET2-T2.4:
✅ PASS

SET2-T2.4-R1:
✅ PASS

SET2-T2.4-R2:
✅ PASS

SET2-T2.5:
✅ PASS

SET2-T2.5-R1:
✅ PASS

SET2-T2.6:
✅ PASS

SET2-T2.6-R1:
✅ PASS

SET2-T2.7:
✅ PASS

SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
✅ PASS

SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
✅ CLOSED

SET3-READINESS-GATE:
🔜 NEXT

SET 3:
🔜 NEXT — READINESS GATE
```

CURRENT NEXT TASK:
SET3-READINESS-GATE

### Dependency

```text
SET 1:
✅ FORMALLY CLOSED

SET 1 downstream checkpoint contract:
✅ AVAILABLE
```

### Current Control

```text
SET 2:
✅ CLOSED

SET2-READINESS-GATE:
✅ PASS

SET2-T2.1:
✅ PASS

SET2-T2.1-R1:
✅ PASS

SET2-T2.2:
✅ PASS

SET2-T2.3-R1:
✅ PASS

SET2-T2.4:
✅ PASS

SET2-T2.4-R1:
✅ PASS

SET2-T2.4-R2:
✅ PASS

SET2-T2.5:
✅ PASS

SET2-T2.5-R1:
✅ PASS

SET2-T2.6:
✅ PASS

SET2-T2.6-R1:
✅ PASS

SET2-T2.7:
✅ PASS

SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
✅ PASS

SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
✅ CLOSED

SET3-READINESS-GATE:
🔜 NEXT

NEXT TASK OWNER:
🧠 LUNA
```

### Responsibility

```text
Primary responsibility model:

🧠 LUNA
Research
Interpretation
Capability modeling
Constraint synthesis
Sequencing
Acceptance

🛠 EXECUTOR
Hardware inspection
Terminal execution
OS inspection
Device enumeration
Driver inspection
Environment inspection
Measurements where explicitly delegated
Evidence persistence

🔄 ORCHESTRATOR
Coordination / control enforcement only
```

---

### SET 2 Scope

SET 2 establishes:

```text
Hardware Identity
      ↓
CPU Capability
      ↓
System Memory
      ↓
Integrated GPU Capability
      ↓
NPU Capability
      ↓
Driver / Runtime / API Availability
      ↓
Interconnect / Data-Movement Characteristics
      ↓
Capability Matrix
      ↓
Constraint Matrix
      ↓
Hardware Truth Contract
```

SET 2 must not establish:

```text
❌ workload placement
❌ scheduling policy
❌ operator implementation
❌ kernel optimization
❌ inference runtime
❌ actual model throughput
❌ latency benchmarks
❌ memory-constrained execution strategy
❌ streaming strategy
❌ heterogeneous execution strategy
❌ CPU optimization
❌ GPU optimization
❌ NPU optimization
```

---

### SET2-READINESS-GATE — SET 2 Readiness

**Responsibility:** 🧠 LUNA

**Status:** 🔜 NEXT

**Dependency:** `SET1-CLOSE PASS`

**Objective:**

Confirm that SET 2 can begin without unsupported hardware assumptions.

Required conditions:

```text
[ ] SET 1 formally closed
[ ] SET 1 output contract available
[ ] target hardware sufficiently identifiable
[ ] actual target environment can be accessed
[ ] Executor can inspect the target environment
[ ] hardware inspection scope is defined
[ ] authoritative hardware sources are available
[ ] evidence requirements are defined
[ ] responsibility is assignable
[ ] no blocking prerequisite remains
```

Result:

```text
READY FOR SET 2
```

or:

```text
NOT READY
```

The readiness gate does not perform actual hardware inspection.

---

### SET2-T2.1 — Target Hardware Identity

**Responsibility:** 🛠 EXECUTOR

**Status:** ⚠ PARTIAL — REQUIRES CORRECTION

**Dependency:** `SET2-READINESS-GATE PASS`

**Objective:**

Verify that the inspected environment is the actual target machine and establish its hardware/software identity.

Required evidence:

```text
CPU model
CPU family / stepping where observable
physical CPU topology
logical processor topology
installed RAM
GPU identity
NPU presence / identity
platform / motherboard identity where relevant
OS
kernel
architecture
virtualization / container environment
```

Output:

```text
TARGET HARDWARE IDENTITY
```

Evidence:
```text
docs/set-2/01-hardware-identity.md
```

```text
⚠ PARTIAL (REQUIRES CORRECTION)
📌 PERSISTED
☁ PUSHED
🔎 REMOTE VERIFIED
```

---

### SET2-T2.2 — CPU Capability Reconnaissance

**Responsibility:** 🛠 EXECUTOR

**Interpretation:** 🧠 LUNA

**Status:** ⚠ PARTIAL

**Dependency:** `SET2-T2.1 PASS`

**Objective:**

Establish the actual CPU capability profile.

Required scope:

```text
core topology
P-core / E-core topology where applicable
logical processors
hardware threading
ISA features
vector extensions
SIMD capabilities
cache hierarchy
relevant instruction-set support
frequency information where reliably observable
```

Relevant examples:

```text
AVX2
AVX-512
AMX
FMA
VNNI
other verified instruction extensions
```

Do not infer ISA support solely from CPU model assumptions.

Output:

```text
CPU CAPABILITY MATRIX
```

---

### SET2-T2.2-R1 — CPU Capability Evidence Reconciliation

**Responsibility:** 🛠 EXECUTOR

**Status:** 🔜 NEXT

**Dependency:** `SET2-T2.2 ⚠ PARTIAL`

**Objective:**

Reconcile T2.2 evidence to strictly distinguish SKU capability, host-observed
capability, and WSL2 guest-exposed capability. Correct overstatements in host
vs guest vs SKU evidence, remove host-equivalence claims, fix AVX-512/AMX
classification, and reclassify cache per-core claims.

Evidence:

```text
docs/set-2/02-cpu-capability-reconnaissance.md
```

```text
⚠ PARTIAL
```

---

### SET2-T2.3 — System Memory Reconnaissance

**Responsibility:** 🛠 EXECUTOR

**Status:** 🔜 NEXT

**Dependency:** `SET2-T2.2-R1 PASS`

**Objective:**

Establish actual system-memory state.

Required scope:

```text
installed RAM
OS-visible RAM
usable RAM
available RAM
memory configuration
memory type where observable
memory channels where observable
frequency / data rate where observable
bandwidth information where authoritative
NUMA characteristics where relevant
reserved memory where observable
```

Required distinction:

```text
Installed Memory
≠
Usable Memory
≠
Currently Available Memory
```

Output:

```text
SYSTEM MEMORY CAPABILITY / CONSTRAINT PROFILE
```

---

### SET2-T2.4 — Intel Integrated GPU Reconnaissance

**Responsibility:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** `SET2-T2.1 PASS`

**Objective:**

Establish the actual integrated Intel GPU capability and software visibility.

Required scope:

```text
GPU identity
architecture / generation
EU / compute-unit information where exposed
GPU memory model
shared-system-memory relationship
driver
device visibility
supported APIs
hardware acceleration interfaces
supported precision / data types where authoritative
relevant verified execution features
```

Maintain these distinctions:

```text
GPU hardware exists

≠

GPU is visible to current environment

≠

software stack exposes usable capability
```

Output:

```text
INTEL GPU CAPABILITY MATRIX
```

---

### SET2-T2.5 — Intel NPU Reconnaissance

**Responsibility:** 🛠 EXECUTOR (execution), 🧠 LUNA (acceptance)

**Status:** ✅ PASS

**Dependency:** `SET2-T2.4-R2 PASS`

**Objective:**

Establish the actual NPU presence, capability, and accessibility.

Required scope:

```text
NPU presence
NPU identity
generation
driver
runtime/API availability
device visibility
supported operation domains where authoritative
supported data types / precision where documented
available system access path
```

Maintain:

```text
NPU hardware capability
≠
NPU software accessibility
```

Output:

```text
NPU CAPABILITY MATRIX
```

---

### SET2-T2.6 — Driver / Runtime / API Availability

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** T2.2 + T2.4 + T2.5 evidence

**Objective:**

Determine which verified hardware capabilities are actually accessible through the current software environment.

CPU scope:

```text
OS support
runtime access
```

GPU scope:

```text
driver
oneAPI where relevant
Level Zero
SYCL
OpenCL where relevant
other authoritative accelerator interfaces
```

NPU scope:

```text
driver
NPU runtime/API
device access
```

OS/environment scope:

```text
kernel/device interfaces
permissions
device visibility
container visibility
```

Do not execute the model.

Output:

```text
HARDWARE SOFTWARE-ACCESSIBILITY MATRIX
```

Example structure:

| Resource | Hardware | Driver   | Runtime/API | Visible  | Usable             |
| -------- | -------- | -------- | ----------- | -------- | ------------------ |
| CPU      | VERIFIED | VERIFIED | VERIFIED    | VERIFIED | VERIFIED           |
| GPU      | VERIFIED | VERIFIED | VERIFIED    | VERIFIED | VERIFIED / UNKNOWN |
| NPU      | VERIFIED | VERIFIED | VERIFIED    | VERIFIED | VERIFIED / UNKNOWN |

---

### SET2-T2.6-R1 — Driver / Runtime / API Availability — Control & Evidence Reconciliation

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** SET2-T2.6 (RECONCILIATION REQUIRED)

**Objective:**

Reconcile SET2-T2.6 after the original execution produced technically useful
runtime evidence but violated the required ROADMAP-first execution-order
boundary.

This revision must:

1. reconcile the execution-order violation;
2. audit every active ROADMAP control representation;
3. preserve valid T2.6 runtime evidence;
4. qualify unsupported Vulkan interpretation;
5. verify the semantic remote ROADMAP state;
6. ensure T2.6 is legitimately closed;
7. keep T2.7 blocked until T2.6-R1 passes.

**Scope:**

```text
✅ Audit the existing T2.6 evidence document
✅ Audit every ACTIVE ROADMAP control representation
✅ Verify SET2-T2.6 state
✅ Verify SET2-T2.6-R1 state
✅ Verify SET2-T2.7 dependency/state
✅ Verify Current Control Task
✅ Verify CURRENT NEXT TASK
✅ Verify Current next task
✅ Verify NEXT TASK OWNER
✅ Verify Stop Condition
✅ Verify detailed T2.6 task status
✅ Verify detailed T2.7 task status
✅ Verify SET-level state
✅ Verify remote ROADMAP state
✅ Correct unsupported or over-broad Vulkan wording
✅ Preserve the existing runtime evidence unless directly contradicted
✅ Reconcile the original execution-order violation explicitly
```

**Roadmap-first boundary:** This revision exists because the original T2.6
execution occurred before the required ROADMAP control-state persistence
boundary. ROADMAP MUST be updated to the R1 reconciliation state and remotely
verified before the canonical evidence document is modified.

**Do-not-run:**

```text
❌ Do not begin SET2-T2.7
❌ Do not perform new benchmark work
❌ Do not perform optimization
❌ Do not perform workload placement
❌ Do not perform scheduling
❌ Do not perform model execution
❌ Do not perform operator mapping
❌ Do not benchmark Vulkan, OpenCL, Level Zero, SYCL, GPU, NPU, or CPU performance
❌ Do not interpret driver-file presence as runtime usability
❌ Do not interpret host visibility as guest visibility
❌ Do not interpret Vulkan loader initialization as proof of physical Intel Arc execution
```

**Stop Condition:**

```text
SET2-T2.6-R1:
✅ PASS

SET2-T2.6:
✅ PASS (after reconciliation)

SET2-T2.7:
✅ PASS

SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
🔜 NEXT

Current control task:
SET2-T2.8
```

Evidence:

```text
ROADMAP.md
docs/set-2/06-driver-runtime-api-availability.md
```

---

### SET2-T2.7 — Interconnect / Data-Movement Reconnaissance

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS (R1 reconciled)

**Dependency:** T2.3 + T2.4 + T2.5 + T2.6 + T2.6-R1

**Objective:**
Establish how system resources are connected and what data-movement pathways are supported by evidence.

Required scope:

```text
CPU ↔ RAM
CPU ↔ iGPU
CPU ↔ NPU
GPU ↔ shared memory
NPU ↔ shared/system memory
device-local / shared-memory model
coherency characteristics where authoritative
memory transfer pathways
```

This is not a performance benchmark.

Do not infer bandwidth without authoritative evidence or explicit measurement.

Unknown bandwidth remains:

```text
UNKNOWN
```

Output:

```text
HARDWARE DATA-MOVEMENT / INTERCONNECT MODEL
```

---

### SET2-T2.7-R1 — Interconnect / Data-Movement Evidence Reconciliation

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** SET2-T2.7 (✅ PASS after reconciliation)

**Objective:**

Reconcile SET2-T2.7 evidence after independent review identified unsupported
promotion of `DEVPKEY_PciDevice_ExpressSpecVersion=2` into `PCIe Gen2 x16` link
claims, over-interpretation of PnP / PCI hierarchy as proof of exact physical
silicon-level interconnect topology, over-broad claims regarding NPU shared/system
memory and absence of device-local or near-compute memory, inadequate provenance
for the CPU MESI classification, and stale ROADMAP integrated-commit metadata.

This revision must:

1. Audit `ROADMAP.md` ACTIVE control-state representations.
2. Establish the correct T2.7-R1 control state.
3. Preserve valid T2.7 host and guest observations.
4. Audit `docs/set-2/07-interconnect-data-movement.md`.
5. Correct unsupported PCIe-link claims.
6. Correct physical-topology over-interpretation.
7. Correct NPU shared-memory / private-memory classification.
8. Correct CPU MESI provenance.
9. Preserve known/unknown boundaries.
10. Reconcile ROADMAP integrated-commit metadata with the actual final commit.
11. Perform final ACTIVE control-state synchronization.
12. Commit, push, and remotely verify the reconciled state.

**Roadmap-first boundary:** This revision MUST exist before relying on it as an
active control state. ROADMAP MUST be updated to the R1 reconciliation state and
remotely verified before the canonical evidence document is modified.

**Do-not-run:**

```text
❌ Do NOT begin SET2-T2.8
❌ Do NOT perform benchmark work
❌ Do NOT perform GPU/NPU bandwidth testing
❌ Do NOT perform latency testing
❌ Do NOT perform workload placement
❌ Do NOT perform scheduling
❌ Do NOT perform optimization
❌ Do NOT perform operator mapping
❌ Do NOT perform model execution
❌ Do NOT perform performance characterization
❌ Do NOT redesign SET2
❌ Do NOT modify unrelated historical ROADMAP content
❌ Do NOT modify unrelated evidence documents
❌ Do NOT stage unrelated working-tree modifications
```

**Do-NOT-RUN boundaries:**

```text
Do NOT treat DEVPKEY_PciDevice_ExpressSpecVersion=2 as proof of negotiated/current PCIe Gen2 x16
Do NOT treat PCIROOT(0) as proof of a complete physical silicon interconnect topology
Do NOT treat absence of an OS-visible NPU memory property as proof that the NPU has no private or near-compute memory
Do NOT treat PCIe ATS support as proof of cache coherency
Do NOT infer bandwidth, latency, throughput, performance, DMA behavior, cache-coherency protocol, or exact internal SoC fabric topology unless directly established by authoritative evidence
```

**Stop Condition:**

```text
SET2-T2.7-R1:
✅ PASS

SET2-T2.7:
✅ PASS (after reconciliation)

SET2-T2.8:
🔜 NEXT

Current control task:
SET2-T2.8
```

Evidence:

```text
ROADMAP.md
docs/set-2/07-interconnect-data-movement.md
```

---

### SET2-T2.8 — Hardware Capability & Constraint Synthesis

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** T2.2–T2.7 (all ✅ PASS after T2.7-R1 reconciliation)

**Objective:**

Synthesize:

```text
CPU
RAM
GPU
NPU
Drivers
Runtime / APIs
Interconnect
Data movement
```

into:

```text
HARDWARE CAPABILITY CONTRACT
```

Target structure:

```text
                       TARGET MACHINE
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
         CPU                 GPU                 NPU
          │                   │                   │
      ISA / Cache        Compute / Memory     Compute / API
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                       System Memory
                              ↓
                    Data-Movement Model
                              ↓
                  Software Accessibility
                              ↓
                 Capability / Constraint Matrix
```

Hard boundary:

Do not determine workload placement.

Do not produce rules such as:

```text
GPU = attention
CPU = MLP
NPU = embedding
```

Placement and scheduling belong to downstream SETs.

---

### SET2-T2.9 — SET 2 Boundary / Completeness Audit

**Responsibility:** 🧠 LUNA

**Status:** ✅ PASS

**Dependency:** `SET2-T2.8 PASS`

**Objective:**

Determine whether SET 2 has established all hardware truth required by its defined scope and whether its downstream contract is complete.

Coverage:

```text
hardware identity
CPU
system memory
GPU
NPU
drivers
runtime/API access
interconnect
data movement
capability matrix
constraint matrix
provenance
known / unknown boundary
```

Negative boundary:

```text
❌ no workload placement
❌ no scheduling
❌ no optimization
❌ no benchmarking
❌ no inference
❌ no operator mapping
❌ no runtime memory execution model
❌ no kernel design
```

Output:

```text
SET 2 TECHNICAL EVIDENCE:
COMPLETE / INCOMPLETE
```

Stop Condition:

```text
SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
🔜 NEXT

Current control task:
SET2-CLOSE
```

Evidence:

```text
ROADMAP.md
docs/set-2/09-set2-boundary-completeness-audit.md
docs/set-2/09-set2-boundary-completeness-audit-r1-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r2-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r3-reconciliation.md
```

---

### SET2-T2.9-R1 — T2.9 Control-Plane Reconciliation

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** `SET2-T2.9 COMPLETE`

**Objective:**

Reconcile the active ROADMAP control-plane defect left by the SET2-T2.9
substantive audit commit (`573c821`). The substantive T2.9 evidence remains valid
and is preserved. This R1 task corrects:

1. Stale `Current integrated commit` metadata (`d10a3ec` — the T2.8 parent commit)
   that does not match the actual HEAD (`573c821`). Corrected to
   `573c8211643218fef7fd30dde0bc18826a95caea` per project convention.
   **Note:** The R1 commit advanced HEAD to `afe6acf`, making `573c821` stale
   (R1's parent, not the actual HEAD). SET2-T2.9-R2 corrected this to
   `afe6acf` — the actual HEAD. SET2-T2.9-R3 subsequently established the
   authoritative non-self-referential semantics for the `integrated-commit`
   field (parent of the ROADMAP-persistence commit, not the commit itself),
   making the parent-SHA pattern intentional and stable rather than a defect.
2. The premature control-state transition to SET2-CLOSE that skipped the
   intermediate R1 reconciliation task. The `SET2-T2.9-R1` task section is
   established as the intermediate atomic task between SET2-T2.9 and SET2-CLOSE,
   following the pattern of T2.1-R1, T2.2-R1, T2.3-R1, T2.4-R2, T2.5-R1,
   T2.6-R1, and T2.7-R1.
3. All ACTIVE ROADMAP control representations synchronized to reflect
   T2.9 = ✅ PASS, T2.9-R1 = ✅ PASS, SET2-CLOSE = 🔜 NEXT.

**Scope:**

```text
✅ Audit existing T2.9 evidence (preserved, not re-collected)
✅ Audit all ACTIVE ROADMAP control representations
✅ Verify HEAD == origin/main
✅ Verify commit ancestry (d10a3ec is ancestor of 573c821)
✅ Correct stale integrated-commit metadata to actual HEAD
✅ Establish SET2-T2.9-R1 task section as intermediate control step
✅ Synchronize all ACTIVE control representations
✅ Preserve historical stop-condition snapshots
✅ Commit, push, and remotely verify reconciled state
```

**Roadmap-first boundary:** ROADMAP.md MUST be updated to the R1 reconciliation
state and remotely verified. The T2.9 evidence document is NOT modified.

**Do-not-run:**

```text
❌ Do not begin SET2-CLOSE
❌ Do not begin SET 3
❌ Do not perform new hardware reconnaissance
❌ Do not perform new benchmark work
❌ Do not perform performance characterization
❌ Do not perform workload placement
❌ Do not perform scheduling
❌ Do not perform optimization
❌ Do not perform model execution
❌ Do not rewrite historical task states
❌ Do not modify unrelated files
❌ Do not stage pre-existing 01-hardware-identity.md modification
```

**Stop Condition:**

```text
SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
🔜 NEXT

Current control task:
SET2-CLOSE
```

Evidence:

```text
ROADMAP.md
docs/set-2/09-set2-boundary-completeness-audit.md
docs/set-2/09-set2-boundary-completeness-audit-r1-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r2-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r3-reconciliation.md
```

---

### SET2-T2.9-R2 — Final Control-Plane Reconciliation

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** `SET2-T2.9-R1 COMPLETE`

**Objective:**

Reconcile the active ROADMAP integrated-commit metadata defect left unresolved
by the SET2-T2.9-R1 commit (`afe6acf`). The R1 commit wrote `573c821` (its own
parent) into the `Current integrated commit` field, replicating the exact same
stale-parent defect that T2.9 and T2.9-R1 exhibited. This R2 task corrects the
active integrated-commit value to the actual final repository state (`afe6acf`)
and synchronizes all remaining active control representations.

1. Stale `Current integrated commit` metadata (`573c821` — the R1 commit's parent)
   that does not match the actual HEAD (`afe6acf`). Corrected to
   `afe6acfdceb991bbe1a316f600a2b296ed32a525` per project convention.
2. All ACTIVE ROADMAP control representations synchronized to include
   SET2-T2.9-R2 = ✅ PASS alongside SET2-T2.9 = ✅ PASS, SET2-T2.9-R1 = ✅ PASS,
   SET2-CLOSE = 🔜 NEXT.
3. T2.9-R2 task section established as the final reconciliation step between
   T2.9-R1 and SET2-CLOSE, following the pattern of T2.1-R1, T2.2-R1,
   T2.3-R1, T2.4-R2, T2.5-R1, T2.6-R1, and T2.7-R1.
   **Note:** R2 correctly identified the self-referential SHA problem but
   declared PASS under the implicit "field must equal HEAD" invariant, which is
   technically impossible. SET2-T2.9-R3 resolved this by establishing the
   authoritative non-self-referential semantics: the field records the parent
   of the ROADMAP-persistence commit, not the commit containing the field.
   R2's parent-SHA correction to `afe6acf` is valid under the new contract —
   `afe6acf` was the HEAD before the R2 follow-up commit `49fd937`.

**Scope:**

```text
✅ Audit existing T2.9 evidence (preserved, not re-collected)
✅ Audit all ACTIVE ROADMAP control representations
✅ Verify HEAD == origin/main
✅ Verify commit ancestry (573c821 is ancestor of afe6acf)
✅ Correct stale integrated-commit metadata to actual current HEAD
✅ Establish SET2-T2.9-R2 task section as final control step
✅ Synchronize all ACTIVE control representations
✅ Preserve historical stop-condition snapshots
✅ Commit, push, and remotely verify reconciled state
```

**Roadmap-first boundary:** ROADMAP.md MUST be updated to the R2 reconciliation
state and remotely verified. The T2.9 evidence document and R1 reconciliation
document are NOT modified.

**Do-not-run:**

```text
❌ Do not begin SET2-CLOSE
❌ Do not begin SET 3
❌ Do not perform new hardware reconnaissance
❌ Do not perform new benchmark work
❌ Do not perform performance characterization
❌ Do not perform workload placement
❌ Do not perform scheduling
❌ Do not perform optimization
❌ Do not perform model execution
❌ Do not rewrite historical task states
❌ Do not modify unrelated files
❌ Do not stage pre-existing 01-hardware-identity.md modification
```

**Stop Condition:**

```text
SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
🔜 NEXT

Current control task:
SET2-CLOSE
```

**Evidence:**

```text
ROADMAP.md
docs/set-2/09-set2-boundary-completeness-audit.md
docs/set-2/09-set2-boundary-completeness-audit-r1-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r2-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r3-reconciliation.md
```

---

### SET2-T2.9-R3 — Metadata Contract Reconciliation

**Responsibility:** 🧠 LUNA

**Execution Support:** 🛠 EXECUTOR

**Status:** ✅ PASS

**Dependency:** `SET2-T2.9-R2 COMPLETE`

**Objective:**

Resolve the unresolved control-plane contradiction identified by SET2-T2.9-R2:
the `Current integrated commit` field in ROADMAP.md cannot contain the SHA of
the same Git commit that contains that field content. R2 correctly identified
that every reconciliation attempt wrote the parent commit's SHA rather than the
commit's own SHA — but R2 still declared PASS under the implicit assumption
that the field should equal HEAD, which is an impossible self-referential
invariant.

R3 establishes and documents the authoritative, non-self-referential semantics
for the `Current integrated commit` field:

1. **Definition:** The field records the repository HEAD immediately preceding
   the ROADMAP-persistence commit — i.e., the parent commit of the most recent
   commit that modified ROADMAP.md. This represents the base repository state
   from which the current ROADMAP content was authored.

2. **Technical proof of satisfiability:** Git commit SHAs are cryptographic
   hashes of commit content. Since ROADMAP.md is part of the commit content,
   embedding the commit's own SHA in the file creates an unsolvable fixed-point
   equation `SHA(content_containing_SHA) == SHA`. Writing the parent commit's
   SHA avoids this entirely: the parent is a distinct, already-finalized Git
   object whose SHA is immutable and independent of the current commit's
   content.

3. **Historical validation:** Every commit that has ever updated this field —
   across T2.6-R1 finalization (6682f34), T2.7-R1 finalization (6682f34),
   T2.9 (573c821), R1 (afe6acf), R2 (77bd8dd), and R2 finalization (49fd937) —
   has written the parent commit's SHA. This is not a bug to be corrected; it
   is the only technically possible behavior. R3 formalizes this as the
   intended, stable semantics.

4. **Stability:** After a ROADMAP-persistence commit P (with parent B), the field
   contains B. B is immutable. The field remains correct for the lifetime of P.
   When the next ROADMAP-persistence commit P' is created, the field is updated
   to P (the new parent). No "stale" condition can arise — the field is always
   a valid provenance reference.

5. **Distinguishing the field from HEAD:**
   - Current repository HEAD = the latest commit on main (e.g., `49fd937` at
     R3 authoring time, then the R3 commit itself after persistence).
   - Latest substantive integration commit = the most recent non-metadata
     commit (e.g., `77bd8dd` — the R2 reconciliation).
   - Parent/base commit = HEAD before the ROADMAP-persistence commit (the value
     stored in the field).
   - Historical integration commits = all prior commits (3b2c8b0, 6682f34,
     d10a3ec, 573c821, afe6acf, 77bd8dd) preserved as provenance references.

**Scope:**

```text
✅ Audit existing T2.9 evidence (preserved, not re-collected)
✅ Audit all active ROADMAP control representations
✅ Verify HEAD == origin/main
✅ Verify full ancestry chain: d10a3ec → 573c821 → afe6acf → 77bd8dd → 49fd937
✅ Establish non-self-referential semantics for integrated-commit field
✅ Add semantic definition to ROADMAP.md Document Status
✅ Update integrated-commit field to current HEAD (49fd937) at ROADMAP persistence
✅ Synchronize all active ROADMAP control representations (add T2.9-R3 = PASS)
✅ Preserve historical stop-condition snapshots
✅ Commit, push, and remotely verify reconciled state
```

**Roadmap-first boundary:** ROADMAP.md MUST be updated to the R3 reconciliation
state and remotely verified. The T2.9 evidence document, R1 reconciliation
document, and R2 reconciliation document are NOT modified.

**Do-not-run:**

```text
❌ Do not begin SET2-CLOSE
❌ Do not begin SET 3
❌ Do not perform new hardware reconnaissance
❌ Do not perform new benchmark work
❌ Do not perform performance characterization
❌ Do not perform workload placement
❌ Do not perform scheduling
❌ Do not perform optimization
❌ Do not perform model execution
❌ Do not rewrite historical task states
❌ Do not modify unrelated files
❌ Do not stage pre-existing 01-hardware-identity.md modification
```

**Stop Condition:**

```text
SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
🔜 NEXT

Current control task:
SET2-CLOSE
```

**Evidence:**

```text
ROADMAP.md
docs/set-2/09-set2-boundary-completeness-audit.md
docs/set-2/09-set2-boundary-completeness-audit-r1-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r2-reconciliation.md
docs/set-2/09-set2-boundary-completeness-audit-r3-reconciliation.md
```

---

### SET2-CLOSE — Formal SET 2 Acceptance

**Responsibility:** 🧠 LUNA

**Status:** ✅ CLOSED

**Dependency:** `SET2-T2.9-R3 PASS`

`T2.9 COMPLETE` does not automatically close SET 2.

Formal acceptance requires:

```text
T2.1
T2.2
T2.3
T2.4
T2.5
T2.6
T2.7
T2.8
T2.9
   ↓
SET2-CLOSE
```

---

### SET 2 Evidence Track

Canonical technical evidence:

```text
docs/set-2/
├── 01-hardware-identity.md
├── 02-cpu-capability-reconnaissance.md
├── 03-system-memory-reconnaissance.md
├── 04-intel-gpu-reconnaissance.md
├── 05-intel-npu-reconnaissance.md
├── 06-driver-runtime-api-availability.md
├── 07-interconnect-data-movement.md
├── 08-hardware-capability-synthesis.md
├── 09-set2-boundary-completeness-audit.md
├── 09-set2-boundary-completeness-audit-r1-reconciliation.md
├── 09-set2-boundary-completeness-audit-r2-reconciliation.md
├── 09-set2-boundary-completeness-audit-r3-reconciliation.md
├── 10-set2-close-acceptance.md
```

Formal acceptance remains a separate control task:

```text
SET2-CLOSE
```

Technical evidence and formal acceptance remain separate:

```text
Technical Evidence
        ≠
Formal Acceptance
```

---

### SET 2 Output Contract

When formally closed, SET 2 should provide downstream sets with:

```text
┌────────────────────────────────────────────┐
│          SET 2 HARDWARE TRUTH              │
├────────────────────────────────────────────┤
│ Target Hardware Identity                   │
│ CPU Capability                             │
│ System Memory                              │
│ Intel GPU Capability                       │
│ Intel NPU Capability                       │
│ Driver / Runtime / API Availability        │
│ Interconnect / Data-Movement Constraints   │
│ Capability Matrix                          │
│ Constraint Matrix                          │
└────────────────────────────────────────────┘
                     ↓
              DOWNSTREAM CONTRACT
                     ↓
             SET 3 / SET 4 / SET 5+
```

Evidence must preserve:

```text
VERIFIED FACT
DERIVED FINDING
UNKNOWN
```

Unknowns must not be silently converted into assumptions.

---

### SET 2 Hard Boundary

```text
SET 2 STOP
        │
        ├── ❌ No inference
        ├── ❌ No benchmark
        ├── ❌ No throughput
        ├── ❌ No latency
        ├── ❌ No workload placement
        ├── ❌ No scheduling
        ├── ❌ No operator mapping
        ├── ❌ No kernel design
        ├── ❌ No optimization
        ├── ❌ No streaming
        ├── ❌ No runtime memory model
        └── ❌ No implementation
```

---

## SET 3 — Operator / Computation Model

**Objective:** Translate verified model structure into an explicit operator/computation model without prematurely assuming runtime behavior.

**Status:** ✅ PASS

**Responsibility:** 🧠 LUNA

**Dependency:** SET 1 + SET 2 verified evidence

**Readiness dependency:** `SET2-CLOSE PASS`

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
SET 3 — Operator / Computation Model

SET 0:
✅ FORMALLY CLOSED

SET 1:
✅ FORMALLY CLOSED

SET 1 TECHNICAL EVIDENCE:
✅ COMPLETE

SET 1 FORMAL ACCEPTANCE:
✅ CLOSED

SET 2:
✅ CLOSED

SET2-READINESS-GATE:
✅ PASS

SET2-CLOSE:
✅ CLOSED

CURRENT NEXT TASK:
SET3-CLOSE

NEXT TASK OWNER:
🧠 LUNA

SET2-T2.1:
✅ PASS

SET2-T2.2:
✅ PASS

SET2-T2.2-R1:
✅ PASS

SET2-T2.3:
✅ PASS

SET2-T2.3-R1:
✅ PASS

SET2-T2.4:
✅ PASS

SET2-T2.4-R1:
✅ PASS

SET2-T2.4-R2:
✅ PASS

SET2-T2.5:
✅ PASS

SET2-T2.5-R1:
✅ PASS

SET2-T2.6:
✅ PASS

SET2-T2.6-R1:
✅ PASS

SET2-T2.7:
✅ PASS

SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
✅ PASS

SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET3-READINESS-GATE:
✅ PASS

SET3-CLOSE:
✅ CLOSED

SET 3:
✅ CLOSED
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
9. SET 0 and SET 1 are formally closed.
10. SET 2 must not execute before `SET2-READINESS-GATE` passes.
11. Every Atomic Task has exactly one primary responsibility owner.
12. ORCHESTRATOR may coordinate and enforce control flow but must not replace LUNA's technical research, interpretation, or acceptance authority.
13. SET 2 must not make workload-placement, scheduling, optimization, runtime, or implementation decisions.

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

For hardware facts:

```text
ACTUAL TARGET ENVIRONMENT
        +
AUTHORITATIVE HARDWARE DOCUMENTATION
        ↓
PERSISTED HARDWARE EVIDENCE
        ↓
LUNA INTERPRETATION
        ↓
HARDWARE CAPABILITY / CONSTRAINT CONTRACT
```

`ROADMAP.md` persistence is intentionally batched.

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
✅ PASS            completed and accepted
❌ FAIL            failed acceptance criteria
⚠ PARTIAL         partially completed / unresolved
⏸ BLOCKED         cannot proceed because dependency is unresolved
🔒 NOT STARTED     no execution has begun
🔜 NEXT            next atomic task selected by control layer
📌 PERSISTED       durable artifact committed to repository
☁ PUSHED          commit pushed to remote
🔎 REMOTE VERIFIED remote state independently checked
```

---

# 7. Current Stop Condition

```text
SET 0:
✅ FORMALLY CLOSED

SET 1:
✅ FORMALLY CLOSED

SET 2:
✅ CLOSED

Current next task:
SET3-CLOSE

SET2-T2.1:
✅ PASS

SET2-T2.2:
✅ PASS

SET2-T2.2-R1:
✅ PASS

SET2-T2.3:
✅ PASS

SET2-T2.3-R1:
✅ PASS

SET2-T2.4:
✅ PASS

SET2-T2.4-R1:
✅ PASS

SET2-T2.4-R2:
✅ PASS

SET2-T2.5:
✅ PASS

SET2-T2.5-R1:
✅ PASS

SET2-T2.6:
✅ PASS

SET2-T2.6-R1:
✅ PASS

SET2-T2.7:
✅ PASS

SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
✅ PASS

SET2-T2.9:
✅ PASS

SET2-T2.9-R1:
✅ PASS

SET2-T2.9-R2:
✅ PASS

SET2-T2.9-R3:
✅ PASS

SET2-CLOSE:
✅ CLOSED

SET3-READINESS-GATE:
✅ PASS

SET3-CLOSE:
✅ CLOSED

SET 3:
✅ CLOSED

SET 3 operator/computation model evidence persisted and verified.

ROADMAP.md:
PERSISTED

docs/set-2/06-driver-runtime-api-availability.md:
PERSISTED

docs/set-2/07-interconnect-data-movement.md:
PERSISTED

docs/set-2/08-hardware-capability-synthesis.md:
PERSISTED

docs/set-2/09-set2-boundary-completeness-audit.md:
🔎 REMOTE VERIFIED

docs/set-2/09-set2-boundary-completeness-audit-r1-reconciliation.md:
🔎 REMOTE VERIFIED

docs/set-2/09-set2-boundary-completeness-audit-r2-reconciliation.md:
🔎 REMOTE VERIFIED

docs/set-2/09-set2-boundary-completeness-audit-r3-reconciliation.md:
🔎 REMOTE VERIFIED

docs/set-3/01-operator-computation-model.md:
📌 PERSISTED
☁ PUSHED
🔎 REMOTE VERIFIED
```
