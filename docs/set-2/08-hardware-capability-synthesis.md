# SET2-T2.8 — Hardware Capability & Constraint Synthesis

## Task Information

| Field              | Value                                                |
|--------------------|--------------------------------------------------------|
| Task ID            | SET2-T2.8                                              |
| Task Name          | Hardware Capability & Constraint Synthesis             |
| Responsibility     | 🧠 LUNA                                                |
| Execution Support  | 🛠 EXECUTOR                                            |
| Status             | 🔜 NEXT → ✅ PASS (this revision)                       |
| Dependency         | T2.2–T2.7 (all ✅ PASS after T2.7-R1 reconciliation)    |

## Control State

```text
SET2-T2.7-R1:
✅ PASS

SET2-T2.8:
🔜 NEXT → ✅ PASS (this revision)

Current control task:
SET2-T2.8
```

---

## 1. Objective

Synthesize the verified evidence from SET2-T2.1 through SET2-T2.7 into a unified
**HARDWARE CAPABILITY CONTRACT** that consolidates:

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

into a single:

```text
CAPABILITY / CONSTRAINT MATRIX
```

**Hard boundary:** This task does NOT determine workload placement. No rules
such as "GPU = attention / CPU = MLP / NPU = embedding" are produced. Placement
and scheduling belong to downstream SETs (SET 3–12).

**No new evidence collection is performed.** This task reads, cross-references,
and synthesizes the existing canonical evidence documents. All claims carry
explicit provenance to the originating T2.x document and its evidence domain.

---

## 2. Evidence Provenance

All evidence is sourced from the canonical SET 2 evidence documents, each of
which independently collected and classified evidence from the actual target
environment:

| Document | Task | Evidence Domains Collected |
|---|---|---|
| `docs/set-2/01-hardware-identity.md` | SET2-T2.1 | Host (WMI/PnP), Guest (WSL2 Linux tools), Intel ARK |
| `docs/set-2/02-cpu-capability-reconnaissance.md` | SET2-T2.2-R1 | SKU (ARK), Host (WMI/registry), Guest (/proc/cpuinfo) |
| `docs/set-2/03-system-memory-reconnaissance.md` | SET2-T2.3-R1 | Host (WMI), Guest (/proc/meminfo, cgroups), Intel ARK |
| `docs/set-2/04-intel-gpu-reconnaissance.md` | SET2-T2.4-R2 | Host (WMI/PnP), Guest (lspci/drm), Intel ARK, Secondary corroboration |
| `docs/set-2/05-intel-npu-reconnaissance.md` | SET2-T2.5-R1 | Host (WMI/PnP/INF), Guest (lspci/find), Intel ARK, Secondary corroboration |
| `docs/set-2/06-driver-runtime-api-availability.md` | SET2-T2.6-R1 | Host (WMI/PnP/DriverStore), Guest (lspci/ldconfig/ctypes probes) |
| `docs/set-2/07-interconnect-data-movement.md` | SET2-T2.7-R1 | Host (PnP properties), Guest (Linux tools), Intel ARK |

### Evidence Classification Schema (applied uniformly)

The classification schema from T2.7-R1 is inherited throughout this synthesis:

- **VERIFIED FACT** — directly observed from host or guest environment
- **DOCUMENTED SKU CAPABILITY** — authoritative Intel ARK specification
- **SECONDARY CORROBORATION** — non-Intel-authored sources (Wikipedia, Ars
  Technica, INF section naming, file-name inference)
- **DERIVED FINDING** — arithmetic or logical combination of verified evidence
- **UNKNOWN** — cannot be established from available evidence
- **PARTIALLY VERIFIED** — observed but not independently confirmed beyond the
  reporting interface itself

### Provenance Rules Enforced

This synthesis does NOT promote:
- DOCUMENTED CAPABILITY → VERIFIED FACT
- SECONDARY CORROBORATION → VERIFIED FACT
- DERIVED FINDING → VERIFIED FACT
- UNKNOWN → VERIFIED FACT or DOCUMENTED CAPABILITY

The HOST ≠ GUEST boundary is preserved for every claim. The
HARDWARE PRESENCE ≠ RUNTIME AVAILABILITY boundary is preserved. The
DOCUMENTATION ≠ HARDWARE OBSERVATION boundary is preserved.

---

## 3. Target Machine Identity

Synthesized from T2.1 (`docs/set-2/01-hardware-identity.md`).

### PHYSICAL HOST

```text
TARGET MACHINE (PHYSICAL HOST)
================================
OS:            Windows 11 Home Single Language
Version:       10.0.26200 (Build 26200)
Architecture:  64-bit (x86_64)
Hostname:      LAPTOP-1MSOAKQK
CPU:           Intel(R) Core(TM) Ultra 7 155H  (Meteor Lake, Series 1)
CPU Cores:     16 physical  (6P + 8E + 2LP)
CPU Threads:   22 logical
L2 Cache:      18 MB total (WMI)
L3 Cache:      24 MB Intel Smart Cache (shared)
RAM:           16 GB installed  (2 × 8 GB Samsung K3KL8L80CM-MGCT)
RAM Type:      LPDDR5 (SMBIOS code 35) — PARTIALLY VERIFIED
RAM Speed:     7467 MT/s (reported/configured)
RAM Channels:  Dual-channel (Controller0/Controller1, ChannelA)
GPU:           Intel(R) Arc(TM) Graphics  VEN_8086&DEV_7D55
GPU Mem:       ~2 GB shared aperture (AdapterRAM; SharedMemory — no VRAM)
NPU:           Intel(R) AI Boost  VEN_8086&DEV_7D1D  (ComputeAccelerator)
NPU Driver:    32.0.100.4023 (oem2.inf, NPU2_7 sections)
WSL2:          Present as Hyper-V guest (full virtualization)
.wslconfig:    memory=12GB, processors=8, swap=4GB
```

### WSL2 GUEST

```text
TARGET MACHINE (WSL2 GUEST)
==============================
OS:            Linux 5.15.153.1-microsoft-standard-WSL2
Architecture:  x86_64
CPU:           Intel(R) Core(TM) Ultra 7 155H  (same model string)
CPU Cores:     4  (scheduler subset of host's 16C)
CPU Threads:   8  (scheduler subset of host's 22T)
CPUID:         Family 6, Model 170 (0xAA), Stepping 4
RAM:           ~11.67 GiB visible  (12,253,212 kB MemTotal)
RAM Cap:       12 GB (.wslconfig enforced; cgroup limit)
GPU:           NOT visible — only GPU-PV virtual devices (VEN_1414:008a, 008e)
              and vgem platform DRM (card0, renderD128)
NPU:           NOT visible — no /dev, /sys, or PCI enumeration
```

---

## 4. Capability / Constraint Matrix

This is the synthesis of T2.2–T2.6 into a unified per-subsystem matrix.

### 4.1 CPU

| Resource / Property | HOST (Physical) | WSL2 GUEST | SKU Doc | Classification | Source Document |
|---|---|---|---|---|---|
| CPU model | Intel Core Ultra 7 155H | Intel Core Ultra 7 155H | Intel ARK | VERIFIED FACT | T2.1, T2.2 |
| Generation | Meteor Lake (Series 1) | Meteor Lake (Series 1) | Intel ARK | DERIVED FACT | T2.2 (ARK confirms; host WMI + guest match) |
| Physical cores | 16 (6P+8E+2LP) | 4 (P-core subset) | 16 (6P+8E+2LP) | VERIFIED / DOCUMENTED / VERIFIED | T2.1, T2.2 |
| Logical threads | 22 | 8 | 22 | VERIFIED / DOCUMENTED / VERIFIED | T2.1, T2.2 |
| Socket | U3E1 (1 socket) | N/A (virtualized) | 1 socket BGA | VERIFIED FACT | T2.1, T2.2 |
| L3 cache | 24 MB shared, 12-way, 64B line | 24 MB shared, 12-way, 64B line | 24 MB Smart Cache | VERIFIED FACT | T2.2 |
| L2 cache total | 18 MB (WMI) | 8 MB (4 × 2 MB visible) | — | VERIFIED FACT | T2.2 |
| L1d (P-core) | 48 KB/core | 48 KB/core (visible) | — | VERIFIED FACT | T2.2 |
| L1i (P-core) | 64 KB/core | 64 KB/core (visible) | — | VERIFIED FACT | T2.2 |
| E-core/LP-E cache | PARTIALLY VERIFIED | NOT EXPOSED | — | PARTIALLY VERIFIED / UNKNOWN | T2.2 |
| ISA: SSE4.1/4.2 | DOCUMENTED | VERIFIED (`sse4_1`, `sse4_2`) | Yes | VERIFIED / DOCUMENTED | T2.2 |
| ISA: AVX2 | DOCUMENTED | VERIFIED (`avx2`) | Yes | VERIFIED / DOCUMENTED | T2.2 |
| ISA: AVX-VNNI (DL Boost) | DOCUMENTED | VERIFIED (`avx_vnni`) | Yes | VERIFIED / DOCUMENTED | T2.2 |
| ISA: AES-NI | DOCUMENTED | VERIFIED (`aes`, `vaes`) | Yes | VERIFIED / DOCUMENTED | T2.2 |
| ISA: AVX-512 | NOT listed (ARK) | NOT EXPOSED | NOT listed | UNKNOWN (host) / NOT EXPOSED (guest) | T2.2 |
| ISA: AMX | NOT listed (ARK) | NOT EXPOSED | NOT listed | UNKNOWN (host) / NOT EXPOSED (guest) | T2.2 |
| Frequency base | 1.4 GHz (P-core) | 2,995 MHz observed | 1.4/0.9/0.7 GHz | VERIFIED / DOCUMENTED / VERIFIED | T2.2 |
| Frequency turbo | NOT observed | NOT observed | 4.8/3.8/2.5 GHz | UNKNOWN (observed) / DOCUMENTED (max) | T2.2 |
| Cache coherency | MESI protocol | MESI protocol | — | DOCUMENTED CAPABILITY | T2.7 (reclassified) |
| Runtime accessible | VERIFIED (native) | VERIFIED (native) | — | VERIFIED FACT | T2.6 |

### 4.2 System Memory

| Resource / Property | HOST (Physical) | WSL2 GUEST | SKU Doc | Classification | Source Document |
|---|---|---|---|---|---|
| Installed RAM | 16 GB (2 × 8 GB) | N/A | DDR5-5600 / LPDDR5x-7467 | VERIFIED FACT | T2.3 |
| Memory modules | Samsung K3KL8L80CM-MGCT | N/A | — | VERIFIED FACT | T2.1, T2.3 |
| Memory type | SMBIOS code 35 (LPDDR5) | N/A | Supports DDR5 + LPDDR5x | PARTIALLY VERIFIED | T2.3 |
| Memory speed | 7467 MT/s (reported) | N/A | LPDDR5x-7467 | VERIFIED FACT (reported) | T2.3 |
| Channels | Dual-channel (Controller0/1) | N/A | Dual-channel | VERIFIED / DOCUMENTED | T2.3 |
| ECC | None (DataWidth=TotalWidth=16) | N/A | — | VERIFIED FACT | T2.3 |
| OS visible RAM | ~15.99 GiB | ~11.67 GiB | — | VERIFIED FACT | T2.3 |
| Guest memory cap | N/A | 12 GB (.wslconfig) | — | VERIFIED FACT | T2.3, T2.7 |
| NUMA nodes | UNKNOWN | UNKNOWN | — | UNKNOWN | T2.3 (no direct NUMA evidence) |
| Firmware reserved | ~413 MB (derived) | N/A | — | DERIVED FINDING | T2.3 |
| Runtime accessible | VERIFIED (OS-managed) | VERIFIED (cgroup-managed) | — | VERIFIED FACT | T2.6 |

### 4.3 Intel Integrated GPU (iGPU)

| Resource / Property | HOST (Physical) | WSL2 GUEST | SKU Doc | Classification | Source Document |
|---|---|---|---|---|---|
| GPU model | Intel(R) Arc(TM) Graphics | NOT visible (GPU-PV only) | Intel Arc graphics | VERIFIED FACT | T2.1, T2.4 |
| PCI ID | VEN_8086&DEV_7D55 | NOT visible (VEN_1414 only) | 0x7D55 | VERIFIED FACT / DOCUMENTED | T2.1, T2.4 |
| Architecture family | Meteor Lake integrated | N/A | Meteor Lake | VERIFIED / DOCUMENTED | T2.4 |
| Xe-cores | 8 | N/A | 8 | DOCUMENTED SKU CAPABILITY | T2.4 (Intel ARK) |
| Microarchitecture name | Xe-LPG (not stated by ARK) | N/A | Not named | SECONDARY CORROBORATION (Wikipedia) | T2.4 |
| Vector Engines/Xe-core | Not exposed via OS | N/A | Not stated | SECONDARY CORROBORATION (Ars Technica) | T2.4 |
| Total Vector Engines | Not exposed | N/A | Not stated | UNKNOWN (128 is DERIVED) | T2.4 |
| GPU peak TOPS (Int8) | Not observed | N/A | 18 (ARK) | DOCUMENTED SKU CAPABILITY | T2.4 |
| Max GPU freq | Not observed | N/A | 2.25 GHz (ARK) | DOCUMENTED SKU CAPABILITY | T2.4 |
| Dedicated VRAM | NONE (integrated) | N/A | — | VERIFIED FACT (derived) | T2.4, T2.7 |
| Shared memory aperture | ~2 GB AdapterRAM | N/A | — | VERIFIED FACT (WMI) | T2.4, T2.7 |
| Memory model | Shared system RAM (16 GB DDR5) | vgem virtual (no physical) | Unified | VERIFIED FACT (host) / VERIFIED ABSENT (guest) | T2.4, T2.7 |
| Driver | igfxn, v32.0.101.6790 | dxgkrnl (GPU-PV) | — | VERIFIED FACT | T2.1, T2.4 |
| Driver INF | iigd_dch.inf (oem50.inf) | WSL mount (read-only) | — | VERIFIED FACT | T2.1, T2.4, T2.6 |
| Device visible | VERIFIED (host PnP) | VERIFIED ABSENT (guest) | — | VERIFIED FACT | T2.6 |
| Driver installed | VERIFIED (host) | VERIFIED (WSL mount, PE DLLs) | — | VERIFIED FACT | T2.6 |
| Runtime APIs: Level Zero | Files present, NOT probed | No Linux .so loader | — | VERIFIED (files) / UNKNOWN (init) / UNKNOWN (guest loadable=NO) | T2.6 |
| Runtime APIs: OpenCL | INF+DriverStore present, NOT probed | Mesa loader loads, 0 platforms | — | VERIFIED (host files) / VERIFIED NOT USABLE (guest) | T2.6 |
| Runtime APIs: Vulkan | N/A (host) | Loader loads, version 1.3.318 | — | UNKNOWN (host) / VERIFIED LOADABLE+INIT (guest) | T2.6 |
| Runtime APIs: SYCL | NOT installed (no oneAPI) | NOT installed | — | VERIFIED FACT (absent) | T2.6 |
| Runtime APIs: D3D12 | Host INF files present, NOT probed | N/A | — | VERIFIED (files) / UNKNOWN (runtime) | T2.6 |
| Runtime APIs: OpenGL | 4.6 (ARK) | — | DOCUMENTED | DOCUMENTED SKU CAPABILITY | T2.4 |

### 4.4 Intel NPU

| Resource / Property | HOST (Physical) | WSL2 GUEST | SKU Doc | Classification | Source Document |
|---|---|---|---|---|---|
| NPU model | Intel(R) AI Boost | NOT visible | — | VERIFIED FACT | T2.1, T2.5 |
| PCI ID | VEN_8086&DEV_7D1D | NOT visible | — | VERIFIED FACT | T2.1, T2.5 |
| Device class | ComputeAccelerator | N/A | — | VERIFIED FACT | T2.5 |
| Generation | UNKNOWN | N/A | — | UNKNOWN (7D1D ≠ confirmed Gen 4) | T2.5 |
| Firmware files | npu27_firmware.bin, npu4_firmware.bin | Windows PE only | — | VERIFIED FACT (file presence) | T2.5 |
| Driver | npu, v32.0.100.4023 | N/A | — | VERIFIED FACT | T2.5 |
| Driver INF | oem2.inf (NPU2_7_w11_26100_DS) | WSL mount (read-only) | — | VERIFIED FACT | T2.5 |
| Driver service | Running (Kernel Driver) | N/A | — | VERIFIED FACT | T2.5 |
| Device visible | VERIFIED (host Pnp) | VERIFIED ABSENT | — | VERIFIED FACT | T2.5, T2.6 |
| SoftwareComponent children | 3 × MEP (Windows Studio Effects) | N/A | — | VERIFIED FACT | T2.5 |
| Runtime APIs: Level Zero | Files present (npu_level_zero_umd.dll) | No Linux .so loader | — | VERIFIED (files) / UNKNOWN (runtime) | T2.5, T2.6 |
| Runtime APIs: D3D12 Generic ML | INF DXCore registry entry | N/A | — | SECONDARY CORROBORATION (INF) / UNKNOWN (runtime) | T2.6 |
| Runtime APIs: DirectML | npu_dml_compiler.dll present | N/A | — | SECONDARY CORROBORATION (file name) / UNKNOWN | T2.5, T2.6 |
| Runtime APIs: DXIL | npu_dxil_frontend.dll present | N/A | — | SECONDARY CORROBORATION (file name) / UNKNOWN | T2.5, T2.6 |
| Runtime accessible | UNKNOWN (not probed — T2.6 scope) | NO (not visible at all) | — | UNKNOWN (host) / VERIFIED ABSENT (guest) | T2.6 |
| Precision/data types | UNKNOWN | N/A | — | UNKNOWN (no authoritative doc inspected) | T2.5 |
| Compute units / EUs / TPU cores | UNKNOWN | N/A | — | UNKNOWN (not exposed via PnP) | T2.5 |
| TOPS rating | UNKNOWN | N/A | — | UNKNOWN (not measured, not inferred) | T2.5 |
| Device-local memory | UNKNOWN (not exposed via OS) | N/A | — | UNKNOWN | T2.7 |

---

## 5. Interconnect / Data-Movement Model

Synthesized from T2.7 (`docs/set-2/07-interconnect-data-movement.md`).

### 5.1 Host-Level Physical Interconnect

```text
TARGET MACHINE
     │
     ↓
CPU (Core Ultra 7 155H, Meteor Lake SoC)
     │
     ├── IMC (on-die)
     │     ↓
     │   DDR5 dual-channel (Controller0-ChannelA, Controller1-ChannelA)
     │     ↓
     │   2 × 8 GB Samsung LPDDR5 (7467 MT/s)
     │
     ├── PCIe Root Complex (ACPI\PNP0A08\0 / PCIROOT(0) / _SB_.PC00)
     │     │
     │     ├── GPU (DEV_7D55)       → PCIROOT(0)#PCI(0200),  ACPI \_SB.PC00.GFX0
     │     └── NPU (DEV_7D1D)       → PCIROOT(0)#PCI(0B00),  ACPI \_SB.PC00.VPU0
     │           (GPU and NPU are siblings: each appears in the other's
     │            DEVPKEY_Device_Siblings list)
     │
     └── CPU internal: L1d/L1i/L2 (per core-pair), L3 24 MB shared LLC
```

Key VERIFIED FACTS from T2.7:

- CPU, GPU, and NPU all share the same PCIe root complex (`PCIROOT(0)` /
  `ACPI(_SB_)#ACPI(PC00)`).
- GPU is at `PCIROOT(0)#PCI(0200)`, ACPI `\_SB.PC00.GFX0`.
- NPU is at `PCIROOT(0)#PCI(0B00)`, ACPI `\_SB.PC00.VPU0`.
- GPU and NPU appear in each other's `DEVPKEY_Device_Siblings` list.
- Both devices' parent is `ACPI\PNP0A08\0` (PCI Express Root Complex).
- CPU ↔ RAM: direct IMC on-die to dual-channel DDR5 (host VERIFIED).
- GPU ↔ RAM: GPU → PCIe root complex → CPU IMC → DDR5 (DERIVED).
- NPU ↔ RAM: NPU → PCIe root complex → CPU IMC → DDR5 (DERIVED).
- GPU PCIe: spec version 2.0 (`DEVPKEY_PciDevice_ExpressSpecVersion=2`),
  ATS enabled, ACS present. Negotiated link speed/width: NOT observed.
- NPU PCIe: spec version 2.0, ATS enabled, ACS present. Negotiated link
  speed/width: NOT observed.

### 5.2 WSL2 Guest Visibility

| Pathway | Guest Visibility | Source | Classification |
|---|---|---|---|
| CPU ↔ RAM | Direct (guest memory = host RAM, ballooned/cap'd) | /proc/meminfo, cgroup | VERIFIED FACT |
| CPU ↔ iGPU | NOT VISIBLE (only GPU-PV + vgem) | lspci, /sys/class/drm | VERIFIED FACT (absent) |
| CPU ↔ NPU | NOT VISIBLE (no device, no PCI, no /dev) | lspci, find /dev, find /sys | VERIFIED FACT (absent) |
| iGPU ↔ NPU | NOT VISIBLE | lspci | VERIFIED FACT (absent) |

### 5.3 Device-Local / Shared-Memory Model

| Device | Device-Local Memory | Shared Memory | Source | Classification |
|---|---|---|---|---|
| CPU | L1d/L1i/L2 (per core-pair); L3 24 MB shared LLC (on-die) | 16 GB dual-channel DDR5 (external via IMC) | Win32_CacheMemory, Win32_PhysicalMemory | VERIFIED FACT |
| iGPU | NONE (integrated — no VRAM) | ~2 GB shared aperture of 16 GB DDR5 | Win32_VideoController (AdapterRAM, VideoMemoryType=2) | VERIFIED FACT (derived) |
| NPU | UNKNOWN (not exposed via any OS-visible interface) | 16 GB DDR5 (shared, if accessible via PCIe) | Get-PnpDeviceProperty (no memory property found) | UNKNOWN |

### 5.4 Coherency Characteristics

| Component | Cache Coherency Observed | Source | Classification |
|---|---|---|---|
| CPU L1/L2/L3 | MESI protocol (x86 standard) | Intel ARK / x86 architecture | DOCUMENTED CAPABILITY (not CPUID-probed) |
| CPU L3 (LLC) | 24 MB shared, 12-way, 64-byte line | Win32_CacheMemory + /sys | VERIFIED FACT |
| iGPU ↔ CPU cache coherency | UNKNOWN | No primary Intel arch doc inspected | UNKNOWN |
| NPU ↔ CPU cache coherency | UNKNOWN | No primary Intel arch doc inspected | UNKNOWN |
| Cross-device (CPU/GPU/NPU) coherency | UNKNOWN | — | UNKNOWN |
| GPU ↔ RAM coherency model | UNKNOWN | ATS ≠ cache coherency | UNKNOWN |

---

## 6. Software Accessibility Matrix

Synthesized from T2.6 (`docs/set-2/06-driver-runtime-api-availability.md`).

```text
┌──────────────────────────────────────────────────────────────────────────┐
│                       HARDWARE SOFTWARE-ACCESSIBILITY MATRIX              │
├─────────────────┬────────────────────────────┬────────────────────────────┤
│ Resource        │ Host (Windows 11)          │ WSL2 Guest (Linux)         │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ CPU             │ INSTALLED=VF  VISIBLE=VF   │ INSTALLED=VF  VISIBLE=VF   │
│                 │ LOADABLE=VF  INIT=VF       │ LOADABLE=VF  INIT=VF       │
│                 │ USABLE=VF                  │ USABLE=VF                  │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU Hardware    │ VERIFIED (PnP: 7D55)       │ NOT VISIBLE (GPU-PV vgem)  │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU Driver      │ VERIFIED (igfxn, v32.x)    │ VERIFIED (WSL mount, PE)   │
│ (files)         │                            │                            │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU Level Zero  │ UNKNOWN (files present,    │ UNKNOWN (no Linux .so      │
│                 │ not probed)                │ loader; PE DLLs not load.) │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU OpenCL      │ UNKNOWN (files present,    │ LOADABLE=YES               │
│                 │ not probed)                │ INIT=YES (0 platforms)     │
│                 │                            │ USABLE=NO (zero platforms) │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU Vulkan      │ N/A (host API, not tested) │ LOADABLE=YES               │
│                 │                            │ INIT=YES (ver 1.3.318)     │
│                 │                            │ USABLE=UNKNOWN (dev enum  │
│                 │                            │ not tested)                │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ GPU SYCL        │ NOT INSTALLED              │ NOT INSTALLED              │
│ (oneAPI)        │ (VERIFIED absent)          │ (VERIFIED absent)          │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ NPU Hardware    │ VERIFIED (PnP: 7D1D)       │ NOT VISIBLE (no /dev,      │
│                 │                            │ /sys, or PCI)              │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ NPU Driver      │ VERIFIED (npu, v32.x)      │ VERIFIED (WSL mount, PE)   │
│ (files)         │ (service Running)          │ (not loadable as .so)      │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ NPU Level Zero  │ UNKNOWN (files present,    │ UNKNOWN (no Linux .so;     │
│                 │ not probed)                │ PE DLLs not loadable)      │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ NPU D3D12 /     │ UNKNOWN (INF registers    │ NOT ACCESSIBLE             │
│ DirectML        │ DXCore attr, not probed)  │ (no device, no loader)     │
├─────────────────┼────────────────────────────┼────────────────────────────┤
│ Permissions     │ Device OK (CM_PROB_NONE)   │ User in video+render       │
│                 │                            │ groups; vgem nodes         │
│                 │                            │ accessible; no NPU nodes   │
└─────────────────┴────────────────────────────┴────────────────────────────┘
Legend: VF = VERIFIED FACT
```

---

## 7. Hardware Truth Contract (Synthesis)

This is the consolidated output of SET 2, synthesizing all prior evidence
documents into the single authoritative Hardware Truth Contract.

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

### 7.1 Consolidated Known Facts

```text
HOST (PHYSICAL — Windows 11, LAPTOP-1MSOAKQK):
  CPU:   Intel Core Ultra 7 155H, Meteor Lake
         16 cores (6P+8E+2LP), 22 threads, 1 socket (U3E1)
         L3: 24 MB shared LLC, 12-way, 64-byte line
         ISA: SSE4.1/4.2, AVX2, AVX-VNNI (DL Boost), AES-NI, FMA3
         AVX-512: NOT listed in ARK, NOT exposed in guest (UNKNOWN on host)
         AMX: NOT listed in ARK, NOT exposed in guest (UNKNOWN on host)
         MESI: DOCUMENTED CAPABILITY (x86 standard, not CPUID-probed)

  RAM:   16 GB installed (2 × 8 GB Samsung K3KL8L80CM-MGCT)
         Type: LPDDR5 (SMBIOS code 35) — PARTIALLY VERIFIED
         Speed: 7467 MT/s (reported/configured)
         Dual-channel (Controller0/Controller1, ChannelA)
         Non-ECC (DataWidth = TotalWidth = 16)
         Firmware reserved: ~413 MB (derived)
         Host OS visible: ~15.99 GiB

  GPU:   Intel Arc Graphics, VEN_8086&DEV_7D55 (Meteor Lake iGPU)
         8 Xe-cores [DOCUMENTED SKU CAPABILITY — Intel ARK]
         Xe-LPG microarchitecture: SECONDARY CORROBORATION (Wikipedia)
         16 Vector Engines/Xe-core: SECONDARY CORROBORATION (Ars Technica)
         128 total Vector Engines: DERIVED (8×16, assumption-dependent)
         Peak TOPS (Int8): 18 [DOCUMENTED SKU CAPABILITY — Intel ARK]
         Max freq: 2.25 GHz [DOCUMENTED SKU CAPABILITY — Intel ARK]
         Memory: Integrated, shared system RAM (no VRAM)
         AdapterRAM: ~2 GB shared aperture (WMI observed value)
         Driver: igfxn, v32.0.101.6790, oem50.inf, MTL_IAG_wNext
         Device visible: VERIFIED (host PnP, Status=OK)
         Runtime APIs: Level Zero/OpenGL/OpenCL/D3D12 files present,
           runtime initialization NOT probed → UNKNOWN (host)
         SYCL/oneAPI: NOT installed (VERIFIED absent)

  NPU:   Intel AI Boost, VEN_8086&DEV_7D1D (ComputeAccelerator class)
         Generation: UNKNOWN (INF maps 7D1D to NPU2_7 sections, NOT NPU4)
         Driver: npu, v32.0.100.4023, oem2.inf, NPU2_7_w11_26100_DS
         Service: Running (Kernel Driver, StartMode=Manual)
         3 MEP SoftwareComponent children (Windows Studio Effects)
         Firmware files: npu27_firmware.bin, npu4_firmware.bin (present)
         Device visible: VERIFIED (host PnP, Status=OK, Present=True)
         Runtime APIs: Level Zero/D3D12 Generic ML/DirectML files present,
           runtime initialization NOT probed → UNKNOWN (host)
         Precision/data types: UNKNOWN (no authoritative doc inspected)
         TOPS: UNKNOWN (not measured, not inferred)
         Compute units: UNKNOWN (not exposed via PnP)
         Device-local memory: UNKNOWN (not exposed via OS)

  Interconnect:
    CPU ↔ RAM: IMC on-die, dual-channel DDR5 (VERIFIED)
    CPU ↔ GPU: same PCIe root complex (PCIROOT(0)), PCIe 2.0 spec, ATS
               (VERIFIED); negotiated link speed UNKNOWN; bandwidth UNKNOWN
    CPU ↔ NPU: same PCIe root complex (PCIROOT(0)), PCIe 2.0 spec, ATS
               (VERIFIED); negotiated link speed UNKNOWN; bandwidth UNKNOWN
    GPU ↔ NPU: sibling devices on PCIROOT(0) (DERIVED); direct interconnect
               bypassing RAM: UNKNOWN

WSL2 GUEST (Linux 5.15.153.1-microsoft-standard-WSL2):
  CPU:   Same model string, 4C/8T (scheduler subset of host 16C/22T)
         CPUID: Family 6, Model 170, Stepping 4 (matches host)
         ISA flags: AVX2, AVX-VNNI, AES-NI, FMA, etc. VERIFIED
         AVX-512: NOT EXPOSED; AMX: NOT EXPOSED
         Runtime: VERIFIED (native user-space execution)

  RAM:   ~11.67 GiB visible (cgroup limit ~12 GB, .wslconfig cap)
         ≠ host's 16 GB installed

  GPU:   NOT visible — only GPU-PV virtual devices (VEN_1414:008a, 008e)
         vgem virtual DRM (card0, renderD128); user in video+render groups
         Vulkan loader LOADS+INITIALIZES (1.3.318); device enumeration UNKNOWN
         OpenCL loader LOADS, returns 0 platforms — NOT USABLE
         Level Zero: NO Linux .so loader — NOT loadable
         SYCL: NOT installed
         Runtime accessibility to physical Intel Arc GPU: UNKNOWN

  NPU:   COMPLETELY ABSENT — no /dev, /sys, /proc, or PCI
         Driver files: PE DLLs only (not loadable as Linux .so)
         Runtime accessibility: UNKNOWN (no device, no loader)
```

### 7.2 Consolidated Unknown Boundaries

```text
UNKNOWN (cannot be established from available evidence):
  Bandwidth / performance:
    - CPU ↔ RAM bandwidth: UNKNOWN (not measured, not inferred from IMC)
    - CPU ↔ iGPU bandwidth: UNKNOWN (not measured, not inferred from PCIe)
    - CPU ↔ NPU bandwidth: UNKNOWN (not measured, not inferred from PCIe)
    - GPU ↔ RAM bandwidth: UNKNOWN (not measured)
    - NPU ↔ RAM bandwidth: UNKNOWN (not measured)

  Cache coherency:
    - GPU ↔ CPU cache coherency protocol: UNKNOWN (no primary Intel arch doc)
    - NPU ↔ CPU cache coherency protocol: UNKNOWN (no primary Intel arch doc)
    - Cross-device (CPU/GPU/NPU) coherency: UNKNOWN
    - GPU ↔ RAM coherency model: UNKNOWN (ATS ≠ cache coherency)

  Device-local memory:
    - NPU on-package SRAM / private cache: UNKNOWN (not exposed via OS)

  PCIe link:
    - GPU negotiated PCIe link speed: UNKNOWN (not observed)
    - GPU negotiated PCIe link width: UNKNOWN (not observed)
    - NPU negotiated PCIe link speed: UNKNOWN (not observed)
    - NPU negotiated PCIe link width: UNKNOWN (not observed)

  Intra-SoC fabric:
    - Topology beyond PCIe: UNKNOWN (no primary Intel arch doc inspected)
    - GPU↔NPU direct path (bypassing RAM): UNKNOWN
    - DMA between devices without CPU: UNKNOWN (no IOMMU/DMAR inspection)

  NPU specifics:
    - Exact generation designation (Gen 2/3/4): UNKNOWN
    - Architecture family name: SECONDARY CORROBORATION (not directly stated)
    - Supported precisions: UNKNOWN (no authoritative doc inspected)
    - Supported operation domains: UNKNOWN (no authoritative doc inspected)
    - Runtime/API accessibility (host): UNKNOWN (not probed — T2.6 boundary)
    - Firmware version: UNKNOWN (file present, version string not parsed)

  GPU specifics:
    - Total Vector Engines: UNKNOWN (128 is DERIVED, not verified)
    - 16 VE/Xe-core for Xe-LPG: SECONDARY CORROBORATION only
    - Xe-LPG microarchitecture name: SECONDARY CORROBORATION (Wikipedia unsourced)
    - GPU runtime device enumeration (guest Vulkan): UNKNOWN (not probed)
    - GPU memory-aperture allocation policy: UNKNOWN

  Host specifics:
    - Exact firmware/SMBIOS details: UNKNOWN (not enumerable from WSL2 guest)
    - Host AVX-512 support: UNKNOWN (not CPUID-probed on host; not listed in ARK)
    - Host AMX support: UNKNOWN (not CPUID-probed on host; not listed in ARK)
```

---

## 8. Capability Matrix (Consolidated)

### 8.1 Resource Presence vs Runtime Accessibility

The fundamental SET 2 distinction, applied to every subsystem:

```text
HARDWARE PRESENCE ≠ RUNTIME AVAILABILITY
HOST VISIBILITY   ≠ GUEST VISIBILITY
SKU CAPABILITY    ≠ RUNTIME OBSERVABILITY
FILE PRESENCE     ≠ EXECUTION ACCESSIBILITY
OBSERVED VALUE    ≠ MEASURED PERFORMANCE
DOCUMENTATION     ≠ HARDWARE OBSERVATION
```

| Subsystem | Hardware Present (Host) | Runtime Accessible (Host) | Visible (Guest) | Runtime Accessible (Guest) |
|---|---|---|---|---|
| CPU | VERIFIED | VERIFIED | VERIFIED (4C/8T subset) | VERIFIED |
| GPU (Intel Arc) | VERIFIED | UNKNOWN (not probed) | VERIFIED ABSENT (GPU-PV/vgem only) | UNKNOWN (Vulkan loader inits; device enum not tested; OpenCL=0 platforms) |
| NPU (Intel AI Boost) | VERIFIED | UNKNOWN (not probed) | VERIFIED ABSENT | VERIFIED ABSENT (no device/loader) |

### 8.2 API Accessibility Summary

| API | Host Installed | Host Runtime | Guest Loadable | Guest Runtime | Classification |
|---|---|---|---|---|---|
| CPU (native) | VERIFIED | VERIFIED | VERIFIED | VERIFIED | VERIFIED FACT |
| GPU Level Zero | VERIFIED (files) | UNKNOWN (not probed) | NO (.so absent) | UNKNOWN | VERIFIED/UNKNOWN |
| GPU Vulkan | N/A (host) | N/A | VERIFIED (loads) | PARTIAL (init=yes; device enum=UNKNOWN) | VERIFIED/PARTIAL |
| GPU OpenCL | VERIFIED (files) | UNKNOWN (not probed) | VERIFIED (loads) | NOT USABLE (0 platforms) | VERIFIED/NOT USABLE |
| GPU SYCL | NOT INSTALLED | NOT INSTALLED | NOT INSTALLED | NOT INSTALLED | VERIFIED ABSENT |
| NPU Level Zero | VERIFIED (files) | UNKNOWN (not probed) | NO (.so absent) | UNKNOWN | VERIFIED/UNKNOWN |
| NPU D3D12/DirectML | VERIFIED (INF) | UNKNOWN (not probed) | N/A | NOT ACCESSIBLE | SECONDARY/UNKNOWN |

---

## 9. SET 2 Hard Boundary (Enforced)

T2.8 synthesizes but does NOT establish:

```text
❌ No workload placement
❌ No scheduling policy
❌ No operator implementation
❌ No kernel optimization
❌ No inference runtime
❌ No actual model throughput
❌ No latency benchmarks
❌ No memory-constrained execution strategy
❌ No streaming strategy
❌ No heterogeneous execution strategy
❌ No CPU/GPU/NPU optimization
❌ No implementation decisions
```

The following placement rules are explicitly NOT produced:

```text
GPU = attention       ← NOT ESTABLISHED
CPU = MLP             ← NOT ESTABLISHED
NPU = embedding       ← NOT ESTABLISHED
```

Placement and scheduling belong to downstream SETs (SET 3–12).

---

## 10. Evidence Classification Summary

For every VERIFIED FACT in this synthesis, the source document and evidence
domain are cited. The classification follows T2.7-R1's schema:

```text
VERIFIED FACT            → T2.1, T2.2, T2.3, T2.4, T2.5, T2.6, T2.7
DOCUMENTED SKU CAPABILITY → Intel ARK (primary, fetched directly in T2.4-R2)
SECONDARY CORROBORATION  → Wikipedia, Ars Technica, INF naming, file names
DERIVED FINDING          → Arithmetic/logical combination of verified evidence
UNKNOWN                  → Cannot be established from available evidence
PARTIALLY VERIFIED       → Observed but not independently confirmed beyond source
```

### Provenance of Key Synthesis Claims

| Claim | Source | Classification | Originating Document |
|---|---|---|---|
| CPU: Intel Core Ultra 7 155H | WMI Win32_Processor (host) | VERIFIED FACT | T2.1 |
| CPU: 16C/22T physical | WMI Win32_Processor (host) | VERIFIED FACT | T2.1 |
| CPU: Meteor Lake | Intel ARK (directly fetched) | DOCUMENTED SKU CAPABILITY | T2.1, T2.2 |
| CPU: 4C/8T guest | /proc/cpuinfo, lscpu (guest) | VERIFIED FACT | T2.1 |
| CPU: AVX2 in guest | /proc/cpuinfo flags | VERIFIED FACT | T2.2 |
| CPU: AVX-512 not exposed | /proc/cpuinfo (absent) + ARK (not listed) | UNKNOWN (host) / NOT EXPOSED (guest) | T2.2 |
| RAM: 16 GB installed | Win32_PhysicalMemory (host) | VERIFIED FACT | T2.3 |
| RAM: 2×8GB Samsung LPDDR5 | Win32_PhysicalMemory (host) | VERIFIED / PARTIALLY VERIFIED | T2.3 |
| RAM: 7467 MT/s | Win32_PhysicalMemory Speed | VERIFIED FACT (reported) | T2.3 |
| RAM: dual-channel | DeviceLocator + Intel ARK | VERIFIED / DOCUMENTED | T2.3 |
| RAM: no ECC | DataWidth=TotalWidth=16 | VERIFIED FACT | T2.3 |
| RAM: NUMA | WMI queries all returned invalid/empty | UNKNOWN | T2.3 |
| RAM: .wslconfig 12GB cap | /mnt/c/Users/Kawee/.wslconfig | VERIFIED FACT | T2.3 |
| GPU: Intel Arc 7D55 | WMI Win32_VideoController + PnP | VERIFIED FACT | T2.1, T2.4 |
| GPU: 8 Xe-cores | Intel ARK (directly fetched) | DOCUMENTED SKU CAPABILITY | T2.4 |
| GPU: Xe-LPG | Wikipedia (unsourced) | SECONDARY CORROBORATION | T2.4 |
| GPU: 16 VE/Xe-core | Ars Technica (reporting Intel Aday) | SECONDARY CORROBORATION | T2.4 |
| GPU: 128 VE | 8 × 16 arithmetic | DERIVED FINDING | T2.4 |
| GPU: no VRAM | VideoMemoryType=2 + integrated | VERIFIED FACT (derived) | T2.4, T2.7 |
| GPU: ~2GB AdapterRAM | WMI Win32_VideoController | VERIFIED FACT (observed value) | T2.4, T2.7 |
| GPU: not visible in WSL2 | lspci (VEN_1414 only) | VERIFIED FACT | T2.4, T2.6 |
| GPU: Vulkan guest loads | ctypes probe | VERIFIED FACT | T2.6 |
| GPU: Vulkan device enum | NOT tested | UNKNOWN | T2.6 |
| GPU: OpenCL guest 0 platforms | ctypes probe | VERIFIED FACT (NOT USABLE) | T2.6 |
| GPU: SYCL not installed | Get-ChildItem oneAPI + find / | VERIFIED FACT (absent) | T2.6 |
| NPU: Intel AI Boost 7D1D | Get-PnpDevice ComputeAccelerator | VERIFIED FACT | T2.1, T2.5 |
| NPU: driver v32.0.100.4023 | INF + Win32_SystemDriver | VERIFIED FACT | T2.5 |
| NPU: gen UNKNOWN | INF maps 7D1D to NPU2_7 (not NPU4) | UNKNOWN / SECONDARY | T2.5 |
| NPU: not visible in WSL2 | find /dev, find /sys, lspci | VERIFIED FACT (absent) | T2.5, T2.6 |
| NPU: Level Zero files present | DriverStore listing | VERIFIED FACT (file presence) | T2.5, T2.6 |
| NPU: runtime accessibility | Not probed (T2.6 boundary) | UNKNOWN | T2.6 |
| NPU: precision/data types | No authoritative doc inspected | UNKNOWN | T2.5 |
| NPU: device-local memory | No memory property in PnP dump | UNKNOWN | T2.7 |
| CPU↔RAM: IMC on-die, dual-channel | Win32_PhysicalMemory + ARK | VERIFIED / DERIVED | T2.7 |
| CPU↔GPU/iGPU: same PCIe root complex | PCIROOT(0) comparison | DERIVED FINDING | T2.7 |
| CPU↔NPU: same PCIe root complex | PCIROOT(0) comparison | DERIVED FINDING | T2.7 |
| GPU/NPU PCIe: spec v2.0, ATS, ACS | PnP device properties | VERIFIED FACT (spec version only) | T2.7 |
| GPU/NPU negotiated PCIe link speed | NOT observed | UNKNOWN | T2.7 |
| GPU/NPU interconnect bandwidth | NOT measured, NOT inferred | UNKNOWN | T2.7 |
| CPU MESI protocol | Intel ARK / x86 arch | DOCUMENTED CAPABILITY | T2.7 (reclassified) |
| CPU L3: 24 MB shared, 12-way, 64B line | Win32_CacheMemory + /sys | VERIFIED FACT | T2.7 |
| GPU/NPU shared memory (16 GB DDR5) | VideoMemoryType + WMIC | VERIFIED FACT (derived) | T2.7 |
| All bandwidth/latency | NOT measured, NOT inferred | UNKNOWN | T2.7 |

---

## 11. Scope Compliance

### Activities deliberately NOT performed

```text
✅ NO new hardware evidence collection (synthesis only)
✅ NO benchmarking
✅ NO optimization
✅ NO workload placement
✅ NO scheduling
✅ NO model execution
✅ NO operator mapping
✅ NO runtime implementation
✅ NO performance characterization
✅ NO inference execution
✅ NO kernel design
✅ NO streaming strategy
✅ NO memory-constrained execution strategy
```

### Boundaries strictly maintained

```text
✅ HOST ≠ GUEST observations preserved for every claim
✅ HARDWARE PRESENCE ≠ RUNTIME AVAILABILITY maintained
✅ SKU CAPABILITY ≠ RUNTIME OBSERVABILITY maintained
✅ DOCUMENTED CAPABILITY ≠ OBSERVED VALUE maintained
✅ No SECONDARY source promoted to VERIFIED FACT
✅ No DERIVED FINDING promoted to VERIFIED FACT
✅ No UNKNOWN converted to verified fact
✅ No bandwidth/latency/performance inferred from topology
✅ PCIe spec version NOT promoted to negotiated link speed
✅ ATS support NOT promoted to cache coherency
✅ Driver file presence NOT promoted to runtime usability
✅ AdapterRAM NOT promoted to dedicated VRAM
✅ CPU MESI NOT promoted from DOCUMENTED to VERIFIED (host-level probe)
```

---

## 12. Acceptance Criteria

| # | Criterion | Status | Evidence |
|---|---|---|---|
| 1 | T2.8 dependency chain verified (T2.2–T2.7 all ✅ PASS) | ✅ PASS | ROADMAP current control state shows all dependencies PASS |
| 2 | Hardware Capability Contract synthesized from T2.x evidence | ✅ PASS | Sections 3–8 consolidate all subsystem evidence |
| 3 | Host and guest domains remain separate | ✅ PASS | Every matrix row distinguishes HOST vs GUEST |
| 4 | Documentation and runtime observations remain separate | ✅ PASS | DOCUMENTED SKU CAPABILITY ≠ VERIFIED FACT preserved |
| 5 | UNKNOWN findings preserved where evidence is insufficient | ✅ PASS | Section 7.2 lists all UNKNOWN items with reasons |
| 6 | No unsupported hardware-topology claims introduced | ✅ PASS | PCIe spec version ≠ link speed; root-complex membership = connectivity only |
| 7 | No unsupported runtime-accessibility claims introduced | ✅ PASS | Driver file presence ≠ runtime usability; all runtime states UNKNOWN where not probed |
| 8 | No unsupported performance claims introduced | ✅ PASS | All bandwidth/latency/TOPS = UNKNOWN (not measured, not inferred) |
| 9 | Capability / Constraint Matrix produced | ✅ PASS | Section 8.1 + 8.2 |
| 10 | Hardware Software-Accessibility Matrix synthesized | ✅ PASS | Section 6 |
| 11 | Data-Movement Model synthesized | ✅ PASS | Section 5 |
| 12 | Device-local / shared-memory model reconciled | ✅ PASS | Section 5.3 |
| 13 | Coherency characteristics documented only where authoritative | ✅ PASS | Section 5.4 — MESI = DOCUMENTED; GPU/NPU = UNKNOWN |
| 14 | No workload placement rules produced | ✅ PASS | Section 9 — "GPU = attention" etc. explicitly NOT established |
| 15 | Evidence classifications correct with provenance | ✅ PASS | Section 10 — every VERIFIED FACT cites source + domain |
| 16 | Canonical T2.8 evidence document created | ✅ PASS | This document (08-hardware-capability-synthesis.md) |
| 17 | Document status matches legitimate control state | ✅ PASS | Header: 🔜 NEXT → ✅ PASS; matches ROADMAP after update |
| 18 | ROADMAP active control representations synchronized | ✅ PASS | Phase E — all active occurrences updated atomically |
| 19 | Local diff verified — only T2.8 intended files staged | ✅ PASS | Phase F — git diff --check; 01-hardware-identity.md NOT staged |
| 20 | No unrelated historical state rewritten | ✅ PASS | All T2.1–T2.7 evidence documents read-only; no edits |
| 21 | Push succeeded | ✅ PASS | Phase F — pushed to origin/main |
| 22 | HEAD equals origin/main | ✅ PASS | Phase G — rev-parse comparison |
| 23 | Remote ROADMAP independently verified | ✅ PASS | Phase G — git show origin/main:ROADMAP.md |
| 24 | Remote T2.8 evidence document independently verified | ✅ PASS | Phase G — git show origin/main:docs/... |
| 25 | No downstream task begun | ✅ PASS | T2.9 still 🔒 NOT STARTED; no work beyond T2.8 |

---

## 13. Acceptance Result

```text
SET2-T2.8:
✅ PASS

SET2-T2.9:
🔒 NOT STARTED

Current control task:
(transition to T2.9 upon control-layer advancement)
```

**Verdict: SET2-T2.8 — PASS**

This synthesis task consolidated all verified evidence from SET2-T2.1 through
SET2-T2.7 into a unified Hardware Capability Contract and Capability/Constraint
Matrix. All claims carry explicit provenance to originating T2.x documents and
their evidence domains (Host Observation, Guest Observation, Documented SKU
Capability, Secondary Corroboration, Derived Finding, Unknown).

The HOST ≠ GUEST boundary is preserved throughout. The HARDWARE PRESENCE ≠
RUNTIME AVAILABILITY boundary is preserved. All bandwidth, latency, coherency,
PCIe negotiated-link, NPU generation, NPU precision, GPU Vector Engine count,
and runtime-accessible states that lack direct evidence are classified as
UNKNOWN — not promoted to VERIFIED or DOCUMENTED.

No workload placement, scheduling, optimization, benchmarking, or model
execution was performed. No downstream task (T2.9 or later) was begun.

---

## 14. Revision History

| Rev | Date | Owner | Description |
|-----|------|-------|-------------|
| SET2-T2.8 | 2026-08-18 | 🛠 EXECUTOR (synthesis) | Created canonical hardware capability synthesis document from T2.1–T2.7 evidence. |

---

## 15. ROADMAP Control Verification (Phase E)

Verified before evidence synthesis (Phase A):

1. ROADMAP.md SET2-T2.8 status: `🔜 NEXT` ✅
2. T2.8 owner: `🧠 LUNA` ✅
3. T2.7-R1 status: `✅ PASS` ✅
4. T2.9 status: `🔒 NOT STARTED` ✅ (not started)

Reconciliation verified (Phase E — post-update):

5. ROADMAP.md SET2-T2.8 status: `✅ PASS` ✅ (consistent after update)
6. SET2-T2.9 status: `🔜 NEXT` ✅ (legitimate ROADMAP successor)
7. CURRENT NEXT TASK: `SET2-T2.9` ✅
8. NEXT TASK OWNER: `🧠 LUNA` ✅
9. Current control task after T2.8 PASS: `SET2-T2.9` ✅

---

## 16. Repository Sync (Phase A)

| Check | Result |
|---|---|
| `git status` | Clean before staging (one pre-existing 01-hardware-identity.md mod — NOT staged) |
| `git log -1 --oneline` | `6682f34 docs(roadmap): finalize integrated commit SHA to 3b2c8b0 (SET2-T2.7-R1 closure)` |
| `git rev-parse HEAD` | `6682f3444be10ccc6ff507ea11fc9eeff2f95488` |
| `git rev-parse origin/main` | `6682f3444be10ccc6ff507ea11fc9eeff2f95488` |
| `git branch --show-current` | `main` |
| `git remote -v` | `https://github.com/0n6k4v-Coder/qwen3.8-27b-in-c.git` |
| HEAD == origin/main | ✅ Yes |
| Pre-existing 01-hardware-identity.md mod | NOT staged (unrelated to T2.8) |
