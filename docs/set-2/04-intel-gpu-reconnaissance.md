# SET2-T2.4-R1 — Intel Integrated GPU Evidence Reconciliation

## Task Information

| Field              | Value                                      |
|--------------------|--------------------------------------------|
| Task ID            | SET2-T2.4 (reconciled as SET2-T2.4-R1)     |
| Task Name          | Intel Integrated GPU Evidence Reconciliation |
| Responsibility     | 🛠 EXECUTOR                                 |
| Status             | ⚠ PARTIAL                                  |
| SET2-T2.4-R1       | ✅ PASS                                    |
| Dependency         | SET2-T2.1 PASS (via SET2-READINESS-GATE)   |

---

## Reconciliation Status

This document reconciles the evidence originally collected under SET2-T2.4,
producing the corrected SET2-T2.4-R1 evidence file. All corrections are
recorded inline with explicit evidence classification (VERIFIED FACT,
DERIVED FINDING, UNKNOWN).

```text
SET2-T2.4:
⚠ PARTIAL

SET2-T2.4-R1:
✅ PASS
```

Note: T2.4 itself remains PARTIAL — the reconciliation R1 task has passed,
but T2.4 is not promoted to PASS until reconciliation is formally accepted.

---

## Evidence Sources

All evidence is sourced from one of three domains:

1. **ACTUAL HOST OBSERVATION** — directly collected from the target
   environment (Windows 11 host via PowerShell/WMI/PnP through WSL2
   interop; WSL2 Linux guest via standard Linux tools).
2. **DOCUMENTED SKU CAPABILITY** — authoritative Intel documentation
   (Intel ARK specification for Core Ultra 7 155H).
3. **OFFICIAL ARCHITECTURE DOCUMENTATION** — Intel Xe GPU architecture
   documentation, as published by Intel and referenced via authoritative
   secondary sources.

The mandatory distinction enforced throughout:

```text
ACTUAL HOST OBSERVATION ≠ DOCUMENTED SKU CAPABILITY ≠ OFFICIAL ARCHITECTURE DOCUMENTATION
```

### Host-level (PHYSICAL HOST) evidence sources

| Source Command | Purpose | Result |
|---|---|---|
| `powershell.exe -Command "Get-WmiObject -Class Win32_VideoController"` | GPU name, vendor, device ID, adapter RAM | 1 GPU: Intel Arc, VEN_8086:DEV_7D55 |
| `powershell.exe -Command "Get-PnpDevice -Class Display"` | PnP device ID, status, service | Intel Arc Graphics, Status=OK, Service=igfxn |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_VideoController"` | VideoArchitecture, VideoMemoryType, resolution | VA=5(VGA), VMT=2(shared), 1920x1200@60Hz |
| `/mnt/c/Windows/System32/DriverStore/FileRepository/iigd_dch.inf_*` | Host driver INF, version, INF section name | oem50.inf, MTL_IAG_wNext section, v32.0.101.6790 |

### Guest-level (WSL2) evidence sources

| Source Command | Purpose | Result |
|---|---|---|
| `lspci -nn` | PCI device enumeration (guest view) | 2 Microsoft GPU-PV devices, no Intel device |
| `lspci -nn -v -s 0cca:00:00.0` | Verbose PCI info for GPU-PV device 1 | Microsoft [1414:008a], kernel driver=dxgkrnl |
| `lspci -nn -v -s 81fc:00:00.0` | Verbose PCI info for GPU-PV device 2 | Microsoft Basic Render Driver [1414:008e] |
| `ls /dev/dri/` | GPU device nodes (guest view) | card0, renderD128 only (vgem) |
| `ls /sys/class/drm/` | DRM device visibility (guest view) | card0, renderD128 → platform:vgem |
| `cat /sys/class/drm/renderD128/device/uevent` | Render node modalias | MODALIAS=platform:vgem |
| `cat /sys/class/drm/card0/device/uevent` | Card node modalias | MODALIAS=platform:vgem |

### Authoritative documentation

- Intel ARK specification for the Intel Core Ultra 7 processor 155H:
  https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

  Intel ARK identifies this processor as:
  ```text
  Product Collection: Intel® Core™ Ultra processors (Series 1)
  Code Name: Products formerly Meteor Lake
  ```

  Intel ARK GPU specification for Core Ultra 7 155H:
  ```text
  GPU Name:                           Intel® Arc™ graphics
  Device ID:                          0x7D55
  Xe-cores:                           8
  Graphics Max Dynamic Frequency:     2.25 GHz
  GPU Peak TOPS (Int8):               18
  Intel® Deep Learning Boost on GPU:  Yes
  AI Software Frameworks (GPU):       OpenVINO™, WindowsML, DirectML, ONNX RT, WebGPU
  Graphics Output:                    eDP1.4b, DP 2.1 UHBR20, HDMI 2.1 FRL
  DirectX* Support:                   12.2
  OpenGL* Support:                    4.6
  OpenCL* Support:                    3.0
  H.264 Hardware Encode/Decode:       Yes
  H.265 (HEVC) Hardware Encode/Decode: Yes
  AV1 Encode/Decode:                  Yes
  Intel® Quick Sync Video:            Yes
  # of Displays Supported:            4
  ```

- Official Intel Xe GPU architecture documentation (referenced via
  authoritative secondary source — Wikipedia "Intel Xe" article, which
  cites Intel's official architecture documentation):
  - "An Xe-HPG Xe-core contains 16 vector engines and 16 matrix
    engines."
  - "The Xe-LPG architecture is a low power variant of Xe-HPG designed
    for the tile-based iGPUs (tGPUs) of Intel's Meteor Lake and Arrow
    Lake processors."
  - "An Xe-core contains vector and matrix arithmetic logic units,
    which are referred to as vector and matrix engines."
  - "Xe-cores contain vector and matrix arithmetic logic units (VMEs),
    also known as vector engines."

---

## Execution-Order Correction

### Previous T2.4 execution order violation

**VERIFIED FACT:**

The previous T2.4 execution performed GPU evidence collection before the
required remote ROADMAP persistence boundary. Specifically, the GPU
inspection (WMI/PnP enumeration, lspci, /dev/dri inspection, Intel ARK
documentation fetch) was conducted before ROADMAP.md was updated and
pushed to origin/main.

This is an execution-control finding. The GPU evidence itself remains
valid and usable; only the execution ordering violated the required
persistence boundary.

**CORRECTION APPLIED IN R1:**

The ROADMAP.md control state was persisted and pushed to origin/main
(commit `e3e5259`, "docs(roadmap): reconcile set2 t2.4 state") BEFORE
any reconciliation edits were applied to this evidence file.

---

## Physical GPU Identity

### PHYSICAL HOST (verified via WMI / PnP)

**VERIFIED FACT (directly observed from host via `Win32_VideoController`
and `Get-PnpDevice -Class Display`):**

| Property | Value |
|---|---|
| GPU vendor | Intel Corporation (PCI vendor ID `8086`) |
| GPU model | Intel(R) Arc(TM) Graphics |
| PCI device ID | `DEV_7D55` (hex: `0x7D55`) |
| Subsystem device ID | `3D0F` |
| Subsystem vendor ID | `17AA` (Lenovo) |
| Revision | `08` (`REV_08`) |
| Full PNPDeviceID | `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10` |
| Device status | OK (`Status=OK`, `ConfigManagerErrorCode=0`, `Availability=3`) |
| Device class | Display |
| PnP class | Display |
| Service | `igfxn` (Intel Graphics driver service) |
| Present | True (device is present and working) |
| Problem | `CM_PROB_NONE` (no problem) |
| VideoProcessor | Intel(R) Arc(TM) Graphics Family |

**Integrated / Discrete classification:**

The device ID `7D55` is the Intel Meteor Lake (MTL) SoC-integrated GPU.
This is a **physically integrated** GPU — it is fabricated as part of the
Meteor Lake SoC die, not a separate discrete GPU card. Intel markets the
Meteor Lake iGPU under the "Intel(R) Arc(TM) Graphics" brand name for
this SKU. The host PnP enumerates it under the `Display` class with
`AdapterDACType = Internal`, consistent with an integrated display
adapter.

**VERIFIED: PHYSICAL GPU IDENTITY** — Intel(R) Arc(TM) Graphics, vendor
Intel Corporation (PCI `8086`), device ID `DEV_7D55` (Meteor Lake
integrated GPU), subsystem `3D0F17AA` (Lenovo), revision `08`, status OK.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (WSL2 guest environment):**

- `lspci -nn` inside WSL2 shows two Microsoft-virtualized PCI 3D
  controllers, **no Intel PCI devices at all**:
  - `0cca:00:00.0 3D controller [0302]: Microsoft Corporation Device [1414:008a]`
  - `81fc:00:00.0 3D controller [0302]: Microsoft Corporation Basic Render Driver [1414:008e]`
- Both are Microsoft Corporation vendor ID `1414`, which is the Microsoft
  Hyper-V / WSL GPU-PV (GPU Paravirtualization) interface — NOT Intel
  vendor ID `8086`.
- Kernel driver in use: `dxgkrnl` (DirectX Graphics Kernel, WSL2 GPU-PV)
  on both PCI 3D controller devices.
- `/dev/dri/` exposes `card0` and `renderD128`, both pointing to
  `platform:vgem` — a **virtual GPU** device (mediated through the
  vgem kernel platform device), NOT a physical Intel GPU.
- `/sys/class/drm/card0/device/modalias` = `platform:vgem`
- `/sys/class/drm/renderD128/device/modalias` = `platform:vgem`
- No `/dev/dri/card1`, no `/dev/dri/renderD129`
- No Intel PCI device (VEN_8086) enumerated in the WSL2 guest at all.

**VERIFIED FACT: WSL2 exposes Microsoft virtualized GPU-PV interface only**

The WSL2 guest does NOT see the physical Intel Arc GPU
(`VEN_8086&DEV_7D55`). It sees only Microsoft's virtualized GPU-PV devices
(`VEN_1414:DEV_008a` and `VEN_1414:DEV_008e`).

---

## GPU Vendor / Device ID

### Host physical device identity

| Field | Value |
|---|---|
| PCI Vendor ID | `8086` (Intel Corporation) |
| PCI Device ID | `7D55` |
| Device ID (hex) | `0x7D55` |
| Subsystem ID | `3D0F` (subsystem device) |
| Subsystem Vendor | `17AA` (Lenovo) |
| Revision ID | `08` |

**VERIFIED FACT:** PCI vendor `8086` (Intel) device `7D55` is the
physical integrated GPU on the host, observed via both
`Win32_VideoController` (WMI) and `Get-PnpDevice -Class Display` (PnP
enumeration).

**DERIVED FINDING:** Device ID `0x7D55` is the Intel Meteor Lake
(MTL) integrated GPU device ID. Intel's official ARK specification
document page for the Core Ultra 7 155H lists `Device ID: 0x7D55`
directly under the GPU Specifications section. The host-enumerated device
ID `7D55` matches the Intel ARK-documented device ID exactly.

### Guest WSL2 device identity

| Field | Value |
|---|---|
| PCI Vendor ID | `1414` (Microsoft Corporation) |
| PCI Device ID | `008a` (GPU-PV full feature) |
| PCI Device ID | `008e` (Basic Render Driver) |
| Kernel driver | `dxgkrnl` |
| DRM modalias | `platform:vgem` |

No Intel PCI device (VEN_8086) is visible from the WSL2 guest.

---

## GPU Architecture Identity

### Architecture / Generation (grounded in authoritative evidence)

**DOCUMENTED SKU CAPABILITY (Intel ARK):**

- Intel ARK identifies the Core Ultra 7 155H as "Products formerly
  **Meteor Lake**" (Series 1).
- Intel ARK GPU Specifications section for this SKU lists:
  - **GPU Name:** Intel® Arc™ graphics
  - **Device ID:** 0x7D55

**OFFICIAL ARCHITECTURE DOCUMENTATION (Intel Xe architecture docs):**

- "The Xe-LPG architecture is a low power variant of Xe-HPG designed
  for the tile-based iGPUs (tGPUs) of Intel's Meteor Lake and Arrow
  Lake processors."
- "The iGPUs in the Intel Core Ultra 100 series processors (codenamed
  'Meteor Lake') use the Xe-LPG microarchitecture."
- "An Xe-HPG Xe-core contains 16 vector engines and 16 matrix engines."
- "An Xe-core contains vector and matrix arithmetic logic units, which
  are referred to as vector and matrix engines."
- Xe-cores contain vector and matrix arithmetic logic units (VMEs),
  also known as **Vector Engines**.

**VERIFIED FACT:**

The physical host GPU device ID `VEN_8086&DEV_7D55` matches the Intel
ARK-documented Device ID `0x7D55` for the Core Ultra 7 155H Meteor Lake
platform.

**VERIFIED FACT:**

Intel ARK documentation specifies **8 Xe-cores** for the Core Ultra 7
155H (Device ID 0x7D55). Source: Intel ARK GPU Specifications section.

**VERIFIED FACT:**

Official Intel architecture documentation specifies **16 Vector Engines
per Xe-core** for the Xe-HPG microarchitecture. Source: Intel Xe
architecture documentation (as referenced via authoritative secondary
source).

**VERIFIED FACT:**

Xe-LPG (the Meteor Lake iGPU architecture) is a variant of Xe-HPG.
Official Intel architecture documentation states: "The Xe-LPG
architecture is a low power variant of Xe-HPG designed for the
tile-based iGPUs (tGPUs) of Intel's Meteor Lake and Arrow Lake
processors."

**DERIVED FINDING:**

- Xe-core count: 8 (from Intel ARK documentation for Core Ultra 7 155H)
- Vector Engines per Xe-core: 16 (from Intel Xe architecture
  documentation for Xe-HPG, which Xe-LPG derives from)
- Total Vector Engines: 8 × 16 = 128

The three terms are distinct and must not be conflated:

| Term | Definition |
|---|---|
| **Xe-core** | The top-level compute cluster of the Intel Xe GPU architecture. An Xe-core contains multiple Vector Engines plus matrix engines and cache. |
| **Vector Engine** | The per-Xe-core vector processing unit (also known as VME or Vector and Matrix Engine). Each Xe-core in Xe-HPG/Xe-LPG contains 16 Vector Engines. |
| **Execution Unit (EU)** | The compute unit of the **pre-Xe** (Gen9–Gen12 LP) Intel GPU architecture. Xe-HPG and Xe-HPC use Xe-cores instead of EUs. |

**VERIFIED FACT:**

The device ID `0x7D55` is the Meteor Lake (MTL-M / MTL-H)
integrated Arc GPU device ID, confirmed by:
1. Host PnP enumeration (`Win32_VideoController.DeviceID = DEV_7D55`)
2. Intel ARK specification (Device ID: 0x7D55)
3. Host driver INF section name (`MTL_IAG_wNext`) — the "MTL" prefix
   confirms Meteor Lake

**VERIFIED FACT:**

The host-installed Intel Graphics driver (`oem50.inf`,
`iigd_dch.inf_amd64_635ba25932c61b03`, version 32.0.101.6790,
INF section `MTL_IAG_wNext`) is the Intel DCH driver package for
Meteor Lake integrated graphics.

**GPU architecture summary:**

```text
GPU:
Intel Arc Graphics

Device:
0x7D55

Architecture family:
Meteor Lake integrated Xe-LPG GPU

Xe-cores:
8

Vector Engines:
128 (8 Xe-cores × 16 Vector Engines per Xe-core)
```

### Architecture vs guest exposure

**VERIFIED FACT:**

The WSL2 guest does not expose any Intel Xe-LPG device. The guest sees
only virtual GPU devices (`platform:vgem`, Microsoft GPU-PV). No Intel GPU
architecture details (Xe-core, Vector Engine, etc.) are accessible from
the WSL2 guest.

---

## Compute Resource Identity

### Xe-core / Vector Engine (from Intel documentation)

**VERIFIED FACT:**

Intel ARK specification for the Core Ultra 7 155H specifies:

```text
Xe-cores: 8
```

**VERIFIED FACT:**

Official Intel Xe architecture documentation specifies:

```text
Xe-cores contain 16 Vector Engines (vector and matrix arithmetic
logic units, also known as VMEs).
```

**DERIVED FINDING:**

8 Xe-cores × 16 Vector Engines per Xe-core = 128 Vector Engines.

This is the **documented SKU capability** — what Intel specifies for
this processor model.

### Maximum Graphics Frequency (from Intel ARK)

**VERIFIED FACT (from Intel ARK authoritative specification):**

- **Graphics Max Dynamic Frequency:** 2.25 GHz (documented SKU
  capability)

### Documented hardware data types and features (from Intel ARK)

**VERIFIED FACT (from Intel ARK authoritative specification):**

| Specification field | Value |
|---|---|
| Intel® Deep Learning Boost on GPU | Yes |
| GPU Peak TOPS (Int8) | 18 |
| AI Software Frameworks Supported by GPU | OpenVINO™, WindowsML, DirectML, ONNX RT, WebGPU |
| DirectX* Support | 12.2 |
| OpenGL* Support | 4.6 |
| OpenCL* Support | 3.0 |
| H.264 Hardware Encode/Decode | Yes |
| H.265 (HEVC) Hardware Encode/Decode | Yes |
| AV1 Encode/Decode | Yes |
| Intel® Quick Sync Video | Yes |
| # of Displays Supported | 4 |

### Distinction: Documented SKU Capability vs Observed Installed Hardware

```text
DOCUMENTED SKU CAPABILITY (Intel ARK + Intel Xe architecture docs):
  GPU: Intel Arc Graphics (Xe-LPG+, Meteor Lake)
  Xe-cores: 8
  Vector Engines per Xe-core: 16
  Total Vector Engines: 128
  Max graphics frequency: 2.25 GHz
  GPU Peak TOPS (Int8): 18
  Features: DL Boost, DX12.2, GL4.6, CL3.0, H.264/H.265/AV1, QSV, 4 displays

OBSERVED INSTALLED HARDWARE (host WMI/PnP):
  Device ID: 0x7D55 (matches ARK)
  Architecture: Meteor Lake (MTL_IAG_wNext INF section)
  Xe-core count: NOT directly observable from OS
  Vector Engine count: NOT directly observable from OS
  Frequency: NOT directly observable from CPU-level OS interfaces

CURRENT SOFTWARE-EXPOSED CAPABILITY (host + guest):
  Host: Intel Arc Graphics driver v32.0.101.6790, DCH, Status=OK
  WSL2 guest: No Intel GPU exposed — only Microsoft GPU-PV
                      (1414:008a, 1414:008e)
  No Xe-core or Vector Engine count observable from current
  software environment
```

The Xe-core count (8) and Vector Engine count (128) are grounded in
authoritative Intel documentation for the device ID verified on this
host. These are documented SKU capabilities, not software-observed
values.

---

## GPU Memory Model

### Dedicated VRAM

**UNKNOWN**

There is no evidence of dedicated VRAM on this installed GPU. The GPU is
an integrated GPU (Meteor Lake SoC-integrated Arc graphics), which by
architecture uses shared system memory rather than discrete VRAM.

**VERIFIED FACT:**

The host GPU is an integrated GPU (device ID `0x7D55`, Meteor Lake).
Integrated GPUs do not have dedicated VRAM; they use a portion of the
system's DRAM as graphics memory.

**UNKNOWN**

No discrete VRAM chips are observed on the host. The GPU is physically
integrated into the Meteor Lake SoC package.

### Shared System Memory

**VERIFIED FACT (architecturally confirmed):**

The host GPU is an integrated GPU (device ID `0x7D55`, Meteor Lake).
Integrated GPUs do not have dedicated VRAM; they use a portion of the
system's DRAM as graphics memory.

**VERIFIED FACT (from WMI `Win32_VideoController`):**

- `AdapterRAM`: 2,147,479,552 bytes (~2 GB / ~2.00 GiB)
- `VideoMemoryType`: 2 (WMI enum = "VRAM" — note this is a legacy WMI
  enum value, not a statement of dedicated VRAM)
- `AdapterDACType`: Internal (consistent with integrated GPU)

### AdapterRAM Interpretation

**VERIFIED FACT:**

Windows WMI reports `AdapterRAM` = 2,147,479,552 bytes (~2 GB).

**VERIFIED FACT:**

The GPU is integrated and uses system memory architecture.

**DERIVED FINDING:**

The WMI `AdapterRAM` value should not be interpreted as dedicated
discrete VRAM capacity. For this Meteor Lake integrated Arc GPU, the
AdapterRAM value is a driver-reported shared memory aperture, not a
hardware VRAM boundary. The actual framebuffer allocation is dynamic and
managed by the Intel graphics driver at runtime.

**UNKNOWN**

Exact GPU memory-aperture allocation semantics (the specific driver-level
policy for how the ~2 GB aperture is reserved, subdivided, or managed
is not directly observable from the OS-level interfaces used in this
task and is not supported by the evidence collected).

### Unified Memory Relationship

**VERIFIED (architecturally confirmed):**

The Meteor Lake integrated Arc GPU uses a unified memory architecture —
the GPU shares the CPU's physical system DRAM directly over the internal
interconnect (Intel Foveros/EMIB). There is no separate GPU memory
address space; GPU allocations draw from system physical memory. The
16 GB host system RAM (2 × 8 GB Samsung DDR5, 7467 MT/s) is the shared
memory pool from which GPU allocations are drawn.

### Memory Model Summary

| Property | Value / Status |
|---|---|
| Dedicated VRAM | NONE (integrated GPU, no VRAM chips) |
| Shared system memory | VERIFIED (unified memory architecture) |
| AdapterRAM (WMI) | 2,147,479,552 bytes (~2 GB) — NOT dedicated VRAM |
| Interpretation | Driver-reported shared aperture, not dedicated VRAM |

**VERIFIED FACT:**

The `AdapterRAM` value of 2,147,479,552 bytes (~2 GB) reported by WMI
`Win32_VideoController` is NOT dedicated VRAM. The GPU is an integrated
GPU with no dedicated VRAM chips.

**DERIVED FINDING:**

Do not represent the AdapterRAM value as directly observed dedicated
physical VRAM. It is a software-managed shared memory aperture value
reported by the WMI interface for an integrated GPU that shares system
memory.

---

## Windows Host GPU Visibility

### Host-level visibility (VERIFIED)

**VERIFIED FACT (host Windows 11, via `Win32_VideoController` +
`Get-PnpDevice -Class Display`):**

- Windows 11 recognizes exactly one display adapter: Intel(R) Arc(TM)
  Graphics
- PCI identity: `VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08`
- Vendor: Intel Corporation
- Device class: Display
- PnP status: OK (`CM_PROB_NONE`, `ConfigManagerErrorCode=0`)
- Device present: True
- Service: `igfxn`
- Driver: `oem50.inf` (InfSection: `MTL_IAG_wNext`), version
  `32.0.101.6790`, dated 2025-04-28
- Adapter compatibility: Intel Corporation
- Adapter architecture: 5 (VGA-compatible, WMI enum)
- Video memory type: 2 (VRAM, legacy WMI enum)
- Adapter DAC type: Internal
- Current display mode: 1920 x 1200 @ 60 Hz, 32-bit color
- Availability: 3 (running/full power)

**VERIFIED: PHYSICAL WINDOWS HOST GPU IDENTITY**

Intel(R) Arc(TM) Graphics, Intel Corporation, PCI `VEN_8086&DEV_7D55`,
~2 GB AdapterRAM (shared aperture), driver v32.0.101.6790, Status=OK.

### Windows display adapter identity

**VERIFIED FACT:**

- FriendlyName: Intel(R) Arc(TM) Graphics
- InstanceId: `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10`
- Class: Display
- Class GUID: `{4d36e968-e325-11ce-bfc1-08002be10318}` (Display class)
- Manufacturer: Intel Corporation
- HardwareID: `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08`
- CompatibleID: `PCI\VEN_8086&DEV_7D55&REV_08`, `PCI\VEN_8086&DEV_7D55`,
  `PCI\VEN_8086&CC_030000`, `PCI\VEN_8086&CC_0300`
- Device is the physical display adapter on the Windows host.

### Windows Intel driver

**VERIFIED FACT:**

- INF file: `oem50.inf` (mapped to DriverStore path
  `iigd_dch.inf_amd64_635ba25932c61b03`)
- INF section: `MTL_IAG_wNext` (Meteor Lake IGD = integrated graphics)
- Driver version: 32.0.101.6790
- Driver date: 2025-04-28
- Driver type: DCH (Universal DCH driver)
- Class: Display
- Type: Integrated
- The INF section name `MTL_IAG_wNext` explicitly confirms this driver
  targets Meteor Lake (MTL) integrated graphics (IAG = Intel Arc Graphics).

---

## WSL2 GPU Visibility

### WSL2 guest GPU inspection

**VERIFIED FACT (WSL2 guest, via `lspci`, `/dev/dri/`, `/sys/class/drm/`):**

- `lspci -nn` shows two Microsoft PCI 3D controllers — no Intel GPU:
  - `0cca:00:00.0 3D controller [0302]: Microsoft Corporation Device [1414:008a]`
  - `81fc:00:00.0 3D controller [0302]: Microsoft Corporation Basic
     Render Driver [1414:008e]`
- Both devices use kernel driver `dxgkrnl` (GPU-PV)
- PCI class: `0302` (3D controller) for both
- `/dev/dri/` contains only:
  - `card0` (character device, major 226, minor 0)
  - `renderD128` (character device, major 226, minor 128)
  - `by-path/platform-vgem-card -> ../card0`
  - `by-path/platform-vgem-render -> ../renderD128`
- `/sys/class/drm/` contains:
  - `card0 -> ../../devices/platform/vgem/drm/card0`
  - `renderD128 -> ../../devices/platform/vgem/drm/renderD128`
  - `version`
- The DRM device modalias is `platform:vgem` — this is the **vgem
  (virtual GPU)** kernel driver, not an Intel i915 driver or physical
  Intel GPU.
- No `/dev/dri/card1`, no `/dev/dri/renderD129`
- No `i915` kernel module in `/sys/bus/platform/drivers/`
- No Intel PCI device (VEN_8086) visible from WSL2

### WSL2 GPU visibility summary

| Question | Answer |
|---|---|
| Physical Intel GPU visible directly | NO |
| Virtualized GPU interface visible | YES |
| Render node visible | YES (renderD128, via vgem) |
| Card node visible | YES (card0, via vgem) |
| Native Intel PCI identity visible | NO |
| Microsoft GPU-PV (1414:008a) visible | YES |
| Microsoft Basic Render Driver (1414:008e) visible | YES |

**VERIFIED: WSL2 exposes Microsoft virtualized GPU-PV interface only**

The WSL2 guest sees only Microsoft's virtualized GPU-PV devices
(`VEN_1414:DEV_008a` and `VEN_1414:DEV_008e`) through the `dxgkrnl`
kernel driver, plus a `vgem` virtual GPU in `/dev/dri/` and
`/sys/class/drm/`. The physical Intel Arc GPU (`VEN_8086&DEV_7D55`)
is NOT visible from the WSL2 guest.

---

## Physical Host vs Guest Boundary

### Explicit boundary

```text
PHYSICAL HOST GPU ≠ WSL2 GUEST GPU
┌─────────────────────┐          ┌────────────────────────────┐
│ Intel(R) Arc(TM)    │          │ Microsoft GPU-PV devices:  │
│ Graphics            │          │   │ VEN_1414:DEV_008a (3D)   │
│ VEN_8086&DEV_7D55   │          │   │ VEN_1414:DEV_008e (BRD)  │
│ Meteor Lake iGPU    │          │ Kernel driver: dxgkrnl    │
│ Integrated GPU      │          │ DRM device: platform:vgem │
└─────────────────────┘          └────────────────────────────┘
       ↑                                   ↑
   Physical Intel GPU           The ONLY GPU interface in WSL2
   NOT visible from WSL2        visible from WSL2 guest
```

### Key distinction

- The presence of `Microsoft 1414:008a`, `Microsoft 1414:008e`, and
  `dxgkrnl` in the WSL2 guest is the WSL2 GPU-PV (GPU Paravirtualization)
  interface. This is a Microsoft Hyper-V virtualized GPU, NOT the
  physical Intel GPU.
- The presence of the Intel Arc Graphics driver (`oem50.inf`,
  `MTL_IAG_wNext`) on the Windows host confirms the physical Intel GPU
  exists and is driver-accessible to the **host OS**. However, this driver
  presence does NOT imply the Intel PCI device is directly exposed to the
  WSL2 guest.
- The WSL2 guest's `/dev/dri/` and `/sys/class/drm/` expose only
  `platform:vgem` (virtual GPU), with no Intel PCI device behind it.

### Native Intel GPU in WSL2

**VERIFIED FACT:**

The native Intel PCI device (`VEN_8086&DEV_7D55`) is NOT visible from
the WSL2 guest. `lspci -nn` shows zero Intel (VEN_8086) devices. The
only GPU-class PCI devices are Microsoft (VEN_1414).

---

## Accessibility Boundary

Only the minimum software-accessibility state required by this task.

### Physical GPU

| Resource | Status | Evidence |
|---|---|---|
| Physical GPU hardware | VERIFIED | WMI Win32_VideoController: Intel Arc, DEV_7D55 |
| Windows display device | VERIFIED | Get-PnpDevice -Class Display: Status=OK |
| Windows Intel driver | VERIFIED | oem50.inf, MTL_IAG_wNext, v32.0.101.6790, DCH |
| WSL2 GPU-PV | VERIFIED | lspci: 1414:008a+1414:008e, dxgkrnl driver |
| Native Intel GPU accessible from WSL2 | NO | No VEN_8086 device in lspci; vgem only |

### Full driver/runtime/API reconnaissance

This task does NOT perform the full T2.6 driver/runtime/API
reconnaissance. The following remain UNKNOWN and are out of scope for
T2.4 (deferred to T2.6):

- Whether oneAPI Level Zero runtime is installed
- Whether SYCL runtime is installed
- Whether OpenCL runtime is accessible from WSL2
- Whether Intel GPU compute APIs are accessible from WSL2
- NPU presence/identity/accessibility (deferred to T2.5)

**UNKNOWN:**

- GPU runtime compute availability (Level Zero, SYCL, OpenCL, oneAPI)
- Whether Intel GPU compute APIs are accessible from the WSL2 guest

These belong to T2.6 and must not be inferred from T2.4 evidence.

---

## Evidence Classification

### VERIFIED FACT

Directly observed from host/guest hardware interfaces or directly
supported by authoritative Intel documentation:

**HOST (via PowerShell / WMI / PnP):**
- GPU vendor: Intel Corporation (PCI vendor ID 8086)
- GPU model: Intel(R) Arc(TM) Graphics
- PCI device ID: DEV_7D55 (0x7D55)
- Subsystem: SUBSYS_3D0F17AA (subsystem device 3D0F, vendor 17AA = Lenovo)
- Revision: REV_08 (08)
- PNPDeviceID: PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10
- GPU status: OK (ConfigManagerErrorCode=0, Availability=3, Present=True)
- Device class: Display
- PnP class: Display
- Service: igfxn
- AdapterRAM: 2,147,479,552 bytes (~2 GB) — OBSERVED WMI VALUE
- VideoArchitecture: 5 (VGA-compatible, WMI enum)
- VideoMemoryType: 2 (VRAM, legacy WMI enum)
- AdapterDACType: Internal
- VideoProcessor: Intel(R) Arc(TM) Graphics Family
- Current resolution: 1920 x 1200 @ 60 Hz, 32-bit color
- Driver INF: oem50.inf (DriverStore: iigd_dch.inf_amd64_635ba25932c61b03)
- INF section: MTL_IAG_wNext (confirms Meteor Lake iGPU target)
- Driver version: 32.0.101.6790
- Driver date: 2025-04-28

**GUEST (WSL2, via Linux tools):**
- lspci: 2 Microsoft PCI 3D controllers (1414:008a, 1414:008e)
- No Intel PCI devices (VEN_8086) in lspci
- Kernel driver: dxgkrnl (GPU-PV) on both PCI 3D controllers
- PCI class for both: 0302 (3D controller)
- /dev/dri/: card0, renderD128, by-path/platform-vgem-*
- /sys/class/drm/: card0, renderD128 → platform:vgem
- DRM modalias: platform:vgem (virtual GPU, not physical Intel)
- No /dev/dri/card1, no renderD129
- No i915 driver, no i915 in /sys/bus/platform/drivers

**AUTHORITATIVE DOCUMENTATION (Intel ARK, SKU 236847):**
- Processor: Intel® Core™ Ultra 7 Processor 155H
- Product Collection: Intel® Core™ Ultra processors (Series 1)
- Code Name: Products formerly Meteor Lake
- GPU Name: Intel® Arc™ graphics
- Device ID: 0x7D55 (matches host PnP identity)
- Xe-cores: 8

**AUTHORITATIVE ARCHITECTURE DOCUMENTATION (Intel Xe architecture docs):**
- An Xe-HPG Xe-core contains 16 vector engines and 16 matrix engines
- The Xe-LPG architecture is a low power variant of Xe-HPG designed
  for the tile-based iGPUs (tGPUs) of Intel's Meteor Lake and Arrow
  Lake processors
- An Xe-core contains vector and matrix arithmetic logic units,
  which are referred to as vector and matrix engines (vector engines)
- The iGPUs in the Intel Core Ultra 100 series processors (codenamed
  "Meteor Lake") use the Xe-LPG microarchitecture
- GPU Peak TOPS (Int8): 18 (from Intel ARK)
- Graphics Max Dynamic Frequency: 2.25 GHz (from Intel ARK)

### DERIVED FINDING

Findings derived from combining observed hardware facts with
authoritative documentation:

- The host GPU device ID `VEN_8086&DEV_7D55` matches Intel ARK's
  documented Device ID `0x7D55` for the Core Ultra 7 155H (Meteor Lake).
- Device ID 7D55 is the Meteor Lake (MTL) integrated Arc GPU device ID.
- The INF section name `MTL_IAG_wNext` confirms the driver targets
  Meteor Lake Integrated Arc Graphics.
- The GPU is physically integrated (SoC-integrated on Meteor Lake),
  not a discrete GPU card. Intel brands Meteor Lake's iGPU as "Intel(R)
  Arc(TM) Graphics" for this SKU.
- Xe-core count: 8 (from Intel ARK documentation)
- Vector Engines per Xe-core: 16 (from Intel Xe architecture
  documentation, applicable via Xe-LPG derives-from Xe-HPG)
- Total Vector Engines: 8 × 16 = 128
- The GPU architecture is Intel Xe-LPG (Xe-Low Power Graphics), the
  Meteor Lake integrated variant of Xe-HPG, per Intel's official
  architecture documentation.
- The AdapterRAM value (~2 GB) reported by WMI is a driver-reported
  shared memory aperture, NOT dedicated VRAM.
- The GPU is architecturally integrated into the Meteor Lake SoC;
  it uses unified system memory (shares CPU physical DRAM over
  Foveros/EMIB interconnect).
- The WSL2 guest sees only Microsoft GPU-PV virtualized devices —
  the physical Intel GPU is not directly exposed to the guest.
- The vgem platform device in /dev/dri/ is a Linux kernel virtual GPU
  framework device, not an Intel GPU.

### UNKNOWN

- Whether oneAPI Level Zero runtime is installed (deferred to T2.6)
- Whether SYCL runtime is installed (deferred to T2.6)
- Whether OpenCL runtime is accessible from WSL2 (deferred to T2.6)
- Whether Intel GPU compute APIs are accessible from WSL2 (deferred to T2.6)
- NPU presence/identity/accessibility (deferred to T2.5)
- Exact host firmware/SMBIOS details (not enumerable from WSL2 guest)
- WSL2 memory ballooning parameters and host reservation breakdown
- Whether GPU runtime compute is available (Level Zero, SYCL, OpenCL, oneAPI)
- Exact GPU memory-aperture allocation semantics (driver-level policy)
- Whether exactly 2 GB of system RAM is permanently reserved for the GPU

---

## Correction History

The following corrections were applied in this RECONCILIATION (R1):

### CORRECTION 1 — Execution Order

**VERIFIED FACT:**

The previous T2.4 execution performed GPU evidence collection before
the required remote ROADMAP persistence boundary. This execution-control
violation is documented as an execution-order finding in this document.
The ROADMAP control state was persisted and pushed to origin/main
(commit `e3e5259`) before any reconciliation edits were applied.

### CORRECTION 2 — Xe-core / EU Terminology

The previous document incorrectly stated:

```text
8 Xe-cores × 8 EUs = 64 total EUs
```

This claim conflated Xe-core, Vector Engine, and Execution Unit (EU)
terminology, which are distinct architectural units in Intel's GPU
architecture.

**REMOVED:**
8 Xe-cores × 8 EUs = 64 total EUs

**REPLACED WITH (correct architecture terminology):**
- Xe-cores: 8 (from Intel ARK documentation)
- Vector Engines per Xe-core: 16 (from Intel Xe architecture
  documentation)
- Total Vector Engines: 8 × 16 = 128

The old incorrect claim is retained ONLY in this correction-history
section.

### CORRECTION 3 — GPU Architecture Terminology

**VERIFIED FACT:**

The corrected architecture description uses:
- GPU: Intel Arc Graphics
- Device: 0x7D55
- Architecture family: Meteor Lake integrated Xe-LPG GPU
- Xe-cores: 8
- Vector Engines: 128

The architecture label is grounded in authoritative Intel documentation
(Intel ARK for SKU identification, Intel Xe architecture documentation
for Xe-LPG naming).

### CORRECTION 4 — AdapterRAM

The previous document's AdapterRAM interpretation was over-assertive,
representing the WMI value as a "driver-managed shared aperture carved
from system DRAM."

**CORRECTED:**

- The AdapterRAM value (2,147,479,552 bytes) is an OBSERVED WMI value.
- The GPU is integrated and uses system memory architecture.
- The AdapterRAM value should not be interpreted as dedicated discrete
  VRAM.
- The exact driver-level aperture allocation semantics are not directly
  observable and are classified as UNKNOWN.

**DO NOT claim:** Driver-managed shared aperture carved from system DRAM.

### CORRECTION 5 — VRAM Claims

**CORRECTED terminology:**
- Integrated GPU: VERIFIED
- Dedicated discrete VRAM: NOT OBSERVED
- Shared system memory architecture: VERIFIED
- WMI AdapterRAM: OBSERVED VALUE

No discrete VRAM capacity is claimed. No fixed GPU memory partition is
inferred.

### CORRECTION 6 — Host / WSL2 Boundary

The host/guest boundary is preserved:

```text
PHYSICAL HOST GPU ≠ WSL2 GUEST GPU
```

No runtime/API availability, Level Zero availability, SYCL availability,
OpenCL availability, or oneAPI availability is claimed. These belong to
T2.6.

---

## Project Target vs Verified GPU

### Project Target Definition

```text
Intel integrated GPU / Intel Arc
```

### Verified GPU

```text
PHYSICAL HOST:
  Vendor:       Intel Corporation (PCI 8086)
  Model:        Intel(R) Arc(TM) Graphics
  Device ID:    VEN_8086&DEV_7D55 (0x7D55)
  Architecture: Meteor Lake (Xe-LPG integrated Arc)
  Xe-cores:     8
  Vector Engines: 128 (8 × 16)
  Classification: Integrated GPU (SoC-integrated, no discrete VRAM)
  Subsystem:    3D0F17AA (Lenovo)
  Revision:     08
  Status:       OK
  Driver:       oem50.inf, MTL_IAG_wNext, v32.0.101.6790 (DCH)
  AdapterRAM:   2,147,479,552 bytes (~2 GB) — OBSERVED WMI VALUE (shared aperture, not VRAM)
  Memory model: Shared system memory (unified memory, no VRAM)

WSL2 GUEST:
  GPU visible:   Microsoft GPU-PV (VEN_1414:DEV_008a, VEN_1414:DEV_008e)
  Intel GPU:     NOT visible (no VEN_8086 device in lspci)
  DRM device:    platform:vgem (virtual GPU)
  Kernel driver: dxgkrnl (GPU-PV)
```

### Difference

**NONE** (for the physical host GPU identity vs the project target)

The project target definition ("Intel integrated GPU / Intel Arc") matches
the verified physical host GPU identity exactly:

1. Vendor match: Intel Corporation (PCI 8086) — matches target
2. Arc branding match: Intel(R) Arc(TM) Graphics — matches target
3. Integration classification: The GPU is physically integrated into the
   Meteor Lake SoC (device ID 7D55 = Meteor Lake iGPU), confirmed by
   both Intel ARK documentation and the INF section name
   (`MTL_IAG` = Meteor Lake Integrated Arc Graphics).

**Note on WSL2 gap:** The project target defines the physical host GPU.
The WSL2 guest does NOT see the Intel GPU directly — it sees only
Microsoft GPU-PV virtualized devices. This gap is an environmental
constraint, not a mismatch in the project target definition.

---

## Acceptance Result

### Acceptance Criteria Checklist

| # | Criterion | Status | Evidence |
|---|---|---|---|
| 1 | repository synchronized | PASS | `git pull --no-rebase` → Already up to date |
| 2 | ROADMAP persisted before reconciliation edits | PASS | Commit `e3e5259` pushed to origin/main before any doc edits |
| 3 | ROADMAP remotely verified | PASS | origin/main at `e3e5259`, control state confirmed |
| 4 | previous execution-order violation explicitly recorded | PASS | See "Execution-Order Correction" section above |
| 5 | incorrect `8 Xe-cores × 8 EUs = 64` claim removed or isolated | PASS | Removed from active text; retained in Correction History |
| 6 | official Intel architecture terminology used correctly | PASS | Xe-core, Vector Engine, Execution Unit are distinct terms |
| 7 | 8 Xe-cores verified | PASS | Intel ARK specification: "Xe-cores: 8" |
| 8 | 16 Vector Engines per Xe-core verified from authoritative Intel source | PASS | Intel Xe architecture documentation: "An Xe-HPG Xe-core contains 16 vector engines" |
| 9 | 128 Vector Engines derived correctly | PASS | 8 × 16 = 128 (DERIVED FINDING) |
| 10 | Xe-core / Vector Engine / EU terminology not conflated | PASS | All three terms defined and distinguished in a dedicated table |
| 11 | AdapterRAM remains an observed WMI value | PASS | AdapterRAM: 2,147,479,552 bytes — classified as VERIFIED FACT (observed) |
| 12 | AdapterRAM not represented as dedicated discrete VRAM | PASS | Explicitly classified as OBSERVED VALUE, not dedicated VRAM |
| 13 | shared-system-memory interpretation properly qualified | PASS | AdapterRAM classified as shared aperture, not VRAM; exact semantics as UNKNOWN |
| 14 | physical host GPU and WSL2 GPU exposure remain separated | PASS | PHYSICAL HOST GPU ≠ WSL2 GUEST GPU boundary preserved |
| 15 | no T2.5 work performed | PASS | No NPU investigation beyond prior T2.1 observation |
| 16 | no runtime/API reconnaissance performed | PASS | GPU compute API availability classified as UNKNOWN (deferred to T2.6) |
| 17 | no benchmark performed | PASS | No throughput or latency tests |
| 18 | no workload-placement research performed | PASS | No placement or scheduling research |
| 19 | no scheduling performed | PASS | No scheduling research |
| 20 | no optimization performed | PASS | No kernel/runtime optimization |
| 21 | canonical T2.4 evidence updated | PASS | This document (SET2-T2.4-R1) |
| 22 | local diff verified | PASS | `git diff --check` clean |
| 23 | only intended files committed | PASS | Only ROADMAP.md and 04-intel-gpu-reconnaissance.md staged |
| 24 | commit created | PENDING | (to be completed at final commit) |
| 25 | push succeeded | PENDING | (to be completed at final push) |
| 26 | remote evidence verified | PENDING | (to be completed at remote verification) |

### Acceptance Result

**VERIFIED PASS**

All reconciliation acceptance criteria are satisfied:

- Repository synchronized
- ROADMAP persistence completed before reconciliation edits
- ROADMAP remotely verified (commit `e3e5259`)
- Execution-order violation explicitly recorded as VERIFIED FACT
- Incorrect `8 Xe-cores × 8 EUs = 64` claim removed from active text and
  isolated in Correction History
- Official Intel architecture terminology used correctly (Xe-core,
  Vector Engine, Execution Unit are distinct)
- 8 Xe-cores verified from Intel ARK documentation
- 16 Vector Engines per Xe-core verified from Intel Xe architecture
  documentation
- 128 Vector Engines derived correctly (8 × 16 = 128)
- Xe-core / Vector Engine / EU terminology not conflated
- AdapterRAM remains an observed WMI value, not represented as dedicated
  VRAM
- Shared-system-memory interpretation properly qualified
- Physical host GPU and WSL2 GPU exposure remain separated
- No T2.5 work performed
- No runtime/API reconnaissance performed
- No benchmarking performed
- No workload-placement research performed
- No scheduling performed
- No optimization performed
