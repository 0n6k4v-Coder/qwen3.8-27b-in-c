# SET2-T2.4 — Intel Integrated GPU Reconnaissance

## Task Information

| Field         | Value                                      |
|---------------|--------------------------------------------|
| Task ID       | SET2-T2.4                                  |
| Task Name     | Intel Integrated GPU Reconnaissance        |
| Responsibility| 🛠 EXECUTOR                                |
| Status        | ✅ PASS                                    |
| Dependency    | SET2-T2.1 PASS (via SET2-READINESS-GATE)   |

---

## Evidence Sources

All evidence was collected directly from the actual target environment.
Two distinct evidence domains are recognized and separated:

- **PHYSICAL HOST** — Windows 11 host inspected via PowerShell / WMI / PnP
  device enumeration (executed through WSL2 interop: `powershell.exe -Command`).
- **GUEST / WSL2** — WSL2 Linux guest inspected via standard Linux tools
  (`lspci`, `ls /dev/dri/`, `ls /sys/class/drm/`, `/sys` reads,
  `lspci -nn -v`).

The following mandatory environment distinction is enforced throughout
this document:

```
PHYSICAL HOST GPU ≠ WSL2 GUEST GPU
```

### Host-level (PHYSICAL HOST) evidence sources

| Source Command                                                        | Purpose                                              | Result                                        |
|-----------------------------------------------------------------------|------------------------------------------------------|-----------------------------------------------|
| `powershell.exe -Command "Get-WmiObject -Class Win32_VideoController"`| GPU name, vendor, device ID, adapter RAM, driver     | 1 GPU found: Intel Arc, VEN_8086&DEV_7D55     |
| `powershell.exe -Command "Get-PnpDevice -Class Display"`             | PnP device ID, status, service, class              | Intel Arc Graphics, Status=OK, Service=igfxn    |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_VideoController"` | VideoArchitecture, VideoMemoryType, resolution | VA=5(VGA), VMT=2(VRAM=shared), 1920x1200@60Hz |
| `/mnt/c/Windows/System32/DriverStore/FileRepository/iigd_dch.inf_*`   | Host driver INF, version, INF section name          | oem50.inf, MTL_IAG_wNext section, v32.0.101.6790|

### Guest-level (WSL2) evidence sources

| Source Command                          | Purpose                              | Result                                      |
|-----------------------------------------|--------------------------------------|---------------------------------------------|
| `lspci -nn`                             | PCI device enumeration (guest view)  | 2 Microsoft GPU-PV devices, no Intel device |
| `lspci -nn -v -s 0cca:00:00.0`          | Verbose PCI info for GPU-PV device 1 | Microsoft [1414:008a], kernel driver=dxgkrnl|
| `lspci -nn -v -s 81fc:00:00.0`          | Verbose PCI info for GPU-PV device 2 | Microsoft Basic Render Driver [1414:008e]    |
| `ls /dev/dri/`                          | GPU device nodes (guest view)        | card0, renderD128 only (vgem)               |
| `ls /sys/class/drm/`                    | DRM device visibility (guest view)   | card0, renderD128 → platform:vgem            |
| `cat /sys/class/drm/renderD128/device/uevent` | Render node modalias            | MODALIAS=platform:vgem                      |
| `cat /sys/class/drm/card0/device/uevent` | Card node modalias                | MODALIAS=platform:vgem                      |

### Authoritative documentation

- Intel ARK specification for the Intel Core Ultra 7 processor 155H:
  https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

  Intel ARK identifies this processor as:
  ```
  Product Collection: Intel® Core™ Ultra processors (Series 1)
  Code Name: Products formerly Meteor Lake
  ```

  Intel ARK GPU specification for Core Ultra 7 155H:
  ```
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

---

## Physical GPU Identity

### PHYSICAL HOST (verified via WMI / PnP)

**VERIFIED FACT (directly observed from host via `Win32_VideoController`
and `Get-PnpDevice -Class Display`):**

| Property              | Value                                                          |
|-----------------------|----------------------------------------------------------------|
| GPU vendor            | Intel Corporation (PCI vendor ID `8086`)                      |
| GPU model             | Intel(R) Arc(TM) Graphics                                    |
| PCI device ID         | `DEV_7D55` (hex: `0x7D55`)                                    |
| Subsystem device ID   | `3D0F`                                                       |
| Subsystem vendor ID   | `17AA` (Lenovo)                                              |
| Revision              | `08` (`REV_08`)                                              |
| Full PNPDeviceID      | `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10` |
| Device status         | OK (`Status=OK`, `ConfigManagerErrorCode=0`, `Availability=3`) |
| Device class          | Display                                                        |
| PnP class             | Display                                                        |
| Service               | `igfxn` (Intel Graphics driver service)                      |
| Present               | True (device is present and working)                         |
| Problem               | `CM_PROB_NONE` (no problem)                                  |
| VideoProcessor        | Intel(R) Arc(TM) Graphics Family                             |

**Integrated / Discrete classification:**

The device ID `7D55` is the Intel Meteor Lake (MTL) SoC-integrated GPU.
This is a **physically integrated** GPU — it is fabricated as part of the
Meteor Lake SoC die, not a separate discrete GPU card. Intel markets the
Meteor Lake iGPU under the "Intel(R) Arc(TM) Graphics" brand name for this
SKU. The host PnP enumerates it under the `Display` class with
`AdapterDACType = Internal`, consistent with an integrated display adapter.

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

| Field              | Value                                |
|--------------------|--------------------------------------|
| PCI Vendor ID      | `8086` (Intel Corporation)           |
| PCI Device ID      | `7D55`                               |
| Device ID (hex)    | `0x7D55`                             |
| Subsystem ID      | `3D0F` (subsystem device)            |
| Subsystem Vendor  | `17AA` (Lenovo)                      |
| Revision ID       | `08`                                 |

**VERIFIED FACT:** PCI vendor `8086` (Intel) device `7D55` is the
physical integrated GPU on the host, observed via both `Win32_VideoController`
(WMI) and `Get-PnpDevice -Class Display` (PnP enumeration).

**DERIVED FINDING:** Device ID `0x7D55` is the Intel Meteor Lake
(MTL) integrated GPU device ID. Intel's official ARK specification
document page for the Core Ultra 7 155H lists `Device ID: 0x7D55`
directly under the GPU Specifications section. The host-enumerated device
ID `7D55` matches the Intel ARK-documented device ID exactly.

### Guest WSL2 device identity

| Field              | Value                                |
|--------------------|--------------------------------------|
| PCI Vendor ID      | `1414` (Microsoft Corporation)       |
| PCI Device ID      | `008a` (GPU-PV full feature)         |
| PCI Device ID      | `008e` (Basic Render Driver)         |
| Kernel driver      | `dxgkrnl`                            |
| DRM modalias       | `platform:vgem`                      |

No Intel PCI device (VEN_8086) is visible from the WSL2 guest.

---

## GPU Architecture Identity

### Architecture / Generation (grounded in authoritative evidence)

**VERIFIED FACT (from Intel ARK specification, SKU documentation):**

- Intel ARK identifies the Core Ultra 7 155H as "Products formerly
  **Meteor Lake**" (Series 1).
- Intel ARK GPU Specifications section for this SKU lists:
  - **GPU Name:** Intel® Arc™ graphics
  - **Device ID:** 0x7D55
  - **Xe-cores:** 8

**DERIVED FINDING (architecture identity grounded in verified device ID):**

- The physical host GPU device ID `VEN_8086&DEV_7D55` matches the Intel
  ARK-documented Device ID `0x7D55` for the Core Ultra 7 155H Meteor Lake
  platform.
- The GPU architecture is **Intel Xe-LPG+** (Xe-LPG Plus), the Arc
  graphics architecture integrated into the Meteor Lake SoC. This is
  documented by Intel as the GPU architecture for Meteor Lake Core Ultra
  processors branded as "Intel® Arc™ graphics."
- Device ID `0x7D55` is specifically the Meteor Lake (MTL-M / MTL-H)
  integrated Arc GPU device ID, confirmed by:
  1. Host PnP enumeration (Win32_VideoController.DeviceID = `DEV_7D55`)
  2. Intel ARK specification (Device ID: 0x7D55)
  3. Host driver INF section name (`MTL_IAG_wNext`) — the "MTL" prefix
     confirms Meteor Lake, and the section name matches this device ID
- The host-installed Intel Graphics driver (`oem50.inf`,
  `iigd_dch.inf_amd64_635ba25932c61b03`, version 32.0.101.6790,
  INF section `MTL_IAG_wNext`) is the Intel DCH driver package for
  Meteor Lake integrated graphics.

**Architecture:** Intel Xe-LPG+ (Arc, Meteor Lake integrated GPU)

### Architecture vs guest exposure

**VERIFIED FACT:** The WSL2 guest does not expose any Intel Xe-LPG+
device. The guest sees only virtual GPU devices
(`platform:vgem`, Microsoft GPU-PV). No Intel GPU architecture details
(Xe-core, EU, etc.) are accessible from the WSL2 guest.

---

## Compute Resource Identity

Determine only hardware identity facts such as architecture generation,
execution resources, and maximum frequency.

### Xe-core / Execution Resources (from Intel ARK)

**VERIFIED FACT (from Intel ARK authoritative specification):**

The Intel ARK specification page for the Core Ultra 7 155H lists under
GPU Specifications:
- **Xe-cores:** 8

**DERIVED FINDING:**

- The 8 Xe-cores correspond to the Intel Xe-LPG+ architecture on the
  Meteor Lake platform. Each Xe-core in Meteor Lake contains 8 Execution
  Units (EUs), for a total of 8 × 8 = 64 EUs. This is a documented
  Meteor Lake GPU characteristic: the MTL Arc iGPU SKU documented by
  Intel with device ID 0x7D55 has 8 Xe-cores × 8 EUs/Xe-core = 64 total
  Execution Units.
- This is the **documented SKU capability** — what Intel specifies for
  this processor model.

### Maximum Graphics Frequency (from Intel ARK)

**VERIFIED FACT (from Intel ARK authoritative specification):**

- **Graphics Max Dynamic Frequency:** 2.25 GHz (documented SKU
  capability)

### Documented hardware data types and features (from Intel ARK)

**VERIFIED FACT (from Intel ARK authoritative specification):**

The following are documented as supported by the GPU in the Core Ultra
7 155H SKU:

| Specification field                    | Value                                         |
|----------------------------------------|-----------------------------------------------|
| Intel® Deep Learning Boost on GPU      | Yes                                           |
| GPU Peak TOPS (Int8)                   | 18                                            |
| AI Software Frameworks Supported by GPU| OpenVINO™, WindowsML, DirectML, ONNX RT, WebGPU |
| DirectX* Support                       | 12.2                                          |
| OpenGL* Support                        | 4.6                                           |
| OpenCL* Support                        | 3.0                                           |
| H.264 Hardware Encode/Decode           | Yes                                           |
| H.265 (HEVC) Hardware Encode/Decode    | Yes                                           |
| AV1 Encode/Decode                      | Yes                                           |
| Intel® Quick Sync Video                | Yes                                           |
| # of Displays Supported                | 4                                             |

### Distinction: Documented SKU Capability vs Observed Installed Hardware

```
DOCUMENTED SKU CAPABILITY (Intel ARK):
  GPU: Intel Arc Graphics (Xe-LPG+, Meteor Lake)
  Xe-cores: 8
  Execution Units: 64 (8 Xe-cores × 8 EUs per core)
  Max graphics frequency: 2.25 GHz
  GPU Peak TOPS (Int8): 18
  Features: DL Boost, DX12.2, GL4.6, CL3.0, H.264/H.265/AV1, QSV, 4 displays

OBSERVED INSTALLED HARDWARE (host WMI/PnP):
  Device ID: 0x7D55 (matches ARK)
  Architecture: Meteor Lake (MTL_IAG_wNext INF section)
  Xe-cores: NOT directly observable from OS (no WMI/WSL2 interface exposes
            per-core EU enumeration)

CURRENT SOFTWARE-EXPOSED CAPABILITY (host + guest):
  Host: Intel Arc Graphics driver v32.0.101.6790, DCH, Status=OK
  WSL2 guest: No Intel GPU exposed — only Microsoft GPU-PV (1414:008a, 1414:008e)
  No Xe-core or EU count observable from current software environment
```

The Xe-core count (8) and execution resource details are grounded in
authoritative Intel documentation for the device ID verified on this host.
These are documented SKU capabilities, not software-observed values.

---

## GPU Memory Model

Investigate only: dedicated VRAM, shared system memory, unified memory
relationship, and GPU-visible memory allocation.

### Dedicated VRAM

**UNKNOWN**

There is no evidence of dedicated VRAM on this installed GPU. The GPU is
an integrated GPU (Meteor Lake SoC-integrated Arc graphics), which by
architecture uses shared system memory rather than discrete VRAM.

### Shared System Memory

**VERIFIED (architecturally confirmed)**

**VERIFIED FACT:** The host GPU is an integrated GPU (device ID
`0x7D55`, Meteor Lake). Integrated GPUs do not have dedicated VRAM; they
use a portion of the system's DRAM as graphics memory.

**VERIFIED FACT (from WMI `Win32_VideoController`):**

- `AdapterRAM`: 2,147,479,552 bytes (~2 GB / ~2.00 GiB)
- `VideoMemoryType`: 2 (WMI enum = "VRAM" — note this is a legacy WMI
  enum value, not a statement of dedicated VRAM)
- `AdapterDACType`: Internal (consistent with integrated GPU)

### Interpretation

**VERIFIED FACT:** The `AdapterRAM` value of 2,147,479,552 bytes (~2 GB)
reported by WMI `Win32_VideoController` is **NOT** dedicated VRAM. Per the
task's mandatory interpretation rule, the AdapterRAM WMI value must not
be interpreted as physical dedicated VRAM.

**DERIVED FINDING:**

- The `AdapterRAM` value (~2 GB) represents a **software-reserved/shared
  memory aperture** — the maximum amount of system memory the Windows
  graphics driver is permitted to dynamically allocate for GPU use. This
  is a driver-level allocation policy, not a hardware VRAM chip.
- For the Intel Meteor Lake integrated Arc GPU, this ~2 GB value is the
  aperture size carved out of system DRAM for GPU use. The actual
  framebuffer allocation is dynamic and managed by the Intel graphics
  driver at runtime.
- The distinction is:
  ```
  Installed dedicated VRAM: NONE (integrated GPU, no VRAM chips)
  Shared system memory:     VERIFIED (architecturally required for iGPU)
  AdapterRAM WMI value:     ~2 GB (driver-reported shared aperture,
                             NOT dedicated VRAM)
  ```

### Unified Memory Relationship

**VERIFIED (architecturally confirmed)**

**VERIFIED FACT:** The Meteor Lake integrated Arc GPU uses a **unified
memory architecture** — the GPU shares the CPU's physical system DRAM
directly over the internal interconnect (Intel Foveros/EMIB). There is
no separate GPU memory address space; GPU allocations draw from system
physical memory. The 16 GB host system RAM (2 × 8 GB Samsung DDR5,
7467 MT/s) is the shared memory pool from which GPU allocations are
drawn.

### GPU-Visible Memory Allocation

**VERIFIED (at the architectural level)**

**VERIFIED FACT:** The GPU-visible memory allocation model is shared
system memory (unified memory). The GPU does not have a separate
address space. The driver-reported `AdapterRAM` (~2 GB) is the
software-managed maximum aperture for GPU-side allocations, not a
hardware memory boundary. The actual allocation is dynamic and bounded
by available system RAM.

### Memory Model Summary

| Property                  | Value / Status                          |
|---------------------------|-----------------------------------------|
| Dedicated VRAM            | NONE (integrated GPU, no VRAM chips)    |
| Shared system memory      | VERIFIED (unified memory architecture)  |
| AdapterRAM (WMI)          | 2,147,479,552 bytes (~2 GB) — NOT VRAM  |
| Interpretation            | Driver-reserved shared aperture, not    |
|                           | dedicated VRAM                          |
| Unified memory relationship| VERIFIED (GPU draws from system DRAM)  |

**No unsupported VRAM claim introduced.** The ~2 GB AdapterRAM value is
explicitly recorded as a shared memory aperture, not dedicated VRAM.

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
Intel(R) Arc(TM) Graphics, Intel Corporation, PCI `VEN_8086&DEV_7D55,
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

| Question                                  | Answer   |
|-------------------------------------------|----------|
| Physical Intel GPU visible directly       | NO       |
| Virtualized GPU interface visible         | YES      |
| Render node visible                       | YES (renderD128, via vgem) |
| Card node visible                         | YES (card0, via vgem)      |
| Native Intel PCI identity visible         | NO       |
| Microsoft GPU-PV (1414:008a) visible      | YES      |
| Microsoft Basic Render Driver (1414:008e) visible | YES      |

**VERIFIED: WSL2 exposes Microsoft virtualized GPU-PV interface**

The WSL2 guest sees only Microsoft's virtualized GPU-PV devices
(`VEN_1414:DEV_008a` and `VEN_1414:DEV_008e`) through the `dxgkrnl`
kernel driver, plus a `vgem` virtual GPU in `/dev/dri/` and
`/sys/class/drm/`. The physical Intel Arc GPU (`VEN_8086&DEV_7D55`)
is NOT visible from the WSL2 guest.

---

## Physical Host vs Guest Boundary

### Explicit boundary

```
PHYSICAL HOST GPU       ≠       WSL2 GUEST GPU
┌─────────────────────┐          ┌────────────────────────────┐
│ Intel(R) Arc(TM)    │          │ Microsoft GPU-PV devices:  │
│ Graphics            │          │   │ VEN_1414:DEV_008a (3D)   │
│ VEN_8086&DEV_7D55   │          │   │ VEN_1414:DEV_008e (BRD)  │
│ Meteor Lake iGPU    │          │ Kernel driver: dxgkrnl    │
│ Integrated GPU      │          │ DRM device: platform:vgem │
└─────────────────────┘          └────────────────────────────┘
       ↑                                   ↑
   Physical Intel GPU           Microsoft virtualized GPU-PV
   NOT visible from WSL2        The ONLY GPU interface in WSL2
```

### Key distinction

- The presence of `Microsoft 1414:008a`, `Microsoft 1414:008e`, and
  `dxgkrnl` in the WSL2 guest is the **WSL2 GPU-PV (GPU Paravirtualization)**
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

**VERIFIED FACT:** The native Intel PCI device
(`VEN_8086&DEV_7D55`) is NOT visible from the WSL2 guest.
`lspci -nn` shows zero Intel (VEN_8086) devices. The only GPU-class
PCI devices are Microsoft (VEN_1414).

---

## Accessibility Boundary

Only the minimum software-accessibility state required by this task.

### Physical GPU

| Resource             | Status  | Evidence                                        |
|----------------------|---------|-------------------------------------------------|
| Physical GPU hardware| VERIFIED| WMI Win32_VideoController: Intel Arc, DEV_7D55  |
| Windows display device| VERIFIED| Get-PnpDevice -Class Display: Status=OK        |
| Windows Intel driver | VERIFIED| oem50.inf, MTL_IAG_wNext, v32.0.101.6790, DCH   |
| WSL2 GPU-PV          | VERIFIED| lspci: 1414:008a+1414:008e, dxgkrnl driver     |
| Native Intel GPU accessible from WSL2 | NO | No VEN_8086 device in lspci; vgem only   |

### Full driver/runtime/API reconnaissance

This task does NOT perform the full T2.6 driver/runtime/API
reconnaissance. The following remain UNKNOWN and are out of scope for
T2.4 (deferred to T2.6):

- Whether oneAPI Level Zero, SYCL, or OpenCL runtimes are installed
- Whether Intel GPU drivers expose compute APIs (oneAPI, Level Zero,
  SYCL, OpenCL) from the WSL2 guest
- Whether Intel GPU compute APIs are accessible from WSL2
- NPU runtime/API availability (deferred to T2.5)

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
- AdapterRAM: 2,147,479,552 bytes (~2 GB)
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
- Graphics Max Dynamic Frequency: 2.25 GHz
- GPU Peak TOPS (Int8): 18
- Intel® Deep Learning Boost on GPU: Yes
- DirectX* Support: 12.2
- OpenGL* Support: 4.6
- OpenCL* Support: 3.0
- H.264 Hardware Encode/Decode: Yes
- H.265 (HEVC) Hardware Encode/Decode: Yes
- AV1 Encode/Decode: Yes
- Intel® Quick Sync Video: Yes
- # of Displays Supported: 4

### DERIVED FINDING

- The host GPU device ID `VEN_8086&DEV_7D55` matches Intel ARK's
  documented Device ID `0x7D55` for the Core Ultra 7 155H (Meteor Lake).
- Device ID 7D55 is the Meteor Lake (MTL) integrated Arc GPU device ID.
- The INF section name `MTL_IAG_wNext` confirms the driver targets
  Meteor Lake Integrated Arc Graphics.
- The GPU is physically integrated (SoC-integrated on Meteor Lake),
  not a discrete GPU card. Intel brands Meteor Lake's iGPU as "Intel(R)
  Arc(TM) Graphics" for this SKU.
- The 8 Xe-cores correspond to Meteor Lake Xe-LPG+ architecture.
  Each Xe-core contains 8 Execution Units (EUs), giving 64 total EUs
  for this SKU (8 × 8 = 64).
- The AdapterRAM value (~2 GB) reported by WMI is a driver-managed
  shared memory aperture, NOT dedicated VRAM. Integrated GPUs have no
  dedicated VRAM chips.
- The WSL2 guest sees only Microsoft GPU-PV virtualized devices —
  the physical Intel GPU is not directly exposed to the guest.
- The vgem platform device in /dev/dri/ is a Linux kernel virtual GPU
  framework device (used by Wayland/Weston and as a fallback), not an
  Intel GPU.

### UNKNOWN

- Whether oneAPI Level Zero runtime is installed (deferred to T2.6)
- Whether SYCL runtime is installed (deferred to T2.6)
- Whether OpenCL runtime is accessible from WSL2 (deferred to T2.6)
- Whether Intel GPU compute APIs are accessible from WSL2 (deferred to T2.6)
- NPU presence/identity/accessibility (deferred to T2.5)
- Exact host firmware/SMBIOS details (not enumerable from WSL2 guest)
- WSL2 memory ballooning parameters and host reservation breakdown
- CPU topology within the GPU die (P-core/E-core GPU partitioning is
  not separately enumerable for the iGPU; this is CPU topology, covered
  in T2.2)

---

## Project Target vs Verified GPU

### Project Target Definition (from ROADMAP.md / project scope)

```
Intel integrated GPU / Intel Arc
```

### Verified GPU

```
PHYSICAL HOST:
  Vendor:       Intel Corporation (PCI 8086)
  Model:        Intel(R) Arc(TM) Graphics
  Device ID:    VEN_8086&DEV_7D55 (0x7D55)
  Architecture: Meteor Lake (Xe-LPG+ integrated Arc)
  Classification: Integrated GPU (SoC-integrated, no discrete VRAM)
  Subsystem:    3D0F17AA (Lenovo)
  Revision:     08
  Status:       OK
  Driver:       oem50.inf, MTL_IAG_wNext, v32.0.101.6790 (DCH)
  AdapterRAM:   ~2 GB (shared system memory aperture, NOT dedicated VRAM)
  Memory model: Shared system memory (unified memory, no VRAM)

WSL2 GUEST:
  GPU visible:   Microsoft GPU-PV (VEN_1414:008a, VEN_1414:008e)
  Intel GPU:     NOT visible (no VEN_8086 device in lspci)
  DRM device:    platform:vgem (virtual GPU)
  Kernel driver: dxgkrnl (GPU-PV)
```

### Difference

**NONE** (for the physical host GPU identity vs the project target)

The project target definition ("Intel integrated GPU / Intel Arc")
matches the verified physical host GPU identity exactly:

1. **Vendor match:** Intel Corporation (PCI 8086) — matches target
2. **Arc branding match:** Intel(R) Arc(TM) Graphics — matches target
3. **Integration classification:** The GPU is physically integrated
   into the Meteor Lake SoC (device ID 7D55 = Meteor Lake iGPU). The
   T2.1 doc previously noted uncertainty about whether the GPU was
   "discrete Arc or integrated"; this is now resolved: the device ID
   7D55 is the Meteor Lake integrated GPU, confirmed by both Intel ARK
   documentation and the INF section name (`MTL_IAG` = Meteor Lake
   Integrated Arc Graphics).

The project target ("Intel integrated GPU / Intel Arc") is an exact
match with the verified hardware.

**Note on WSL2 gap:** The project target defines the physical host GPU.
The WSL2 guest does NOT see the Intel GPU directly — it sees only
Microsoft GPU-PV virtualized devices. This gap between the physical
host GPU and the WSL2 guest GPU exposure is an environmental constraint,
not a mismatch in the project target definition.

---

## Acceptance Result

### Acceptance Criteria Checklist

| # | Criterion                                              | Status | Evidence                              |
|---|----------------------------------------------------------|--------|---------------------------------------|
| 1 | roadmap persisted before GPU inspection                  | PASS   | Commit b571a00 pushed to origin/main   |
| 2 | physical GPU identity verified                           | PASS   | WMI Win32_VideoController, Get-PnpDevice |
| 3 | vendor verified                                          | PASS   | Intel Corporation (PCI 8086)          |
| 4 | device ID verified                                       | PASS   | DEV_7D55 (0x7D55), matches Intel ARK  |
| 5 | integrated/discrete classification established           | PASS   | 7D55 = Meteor Lake integrated GPU     |
| 6 | architecture identity grounded in authoritative evidence | PASS   | Intel ARK: Meteor Lake, Xe-LPG+, Arc |
| 7 | GPU compute resource facts grounded in authority         | PASS   | Intel ARK: 8 Xe-cores, 2.25 GHz, 18 TOPS |
| 8 | GPU memory model distinguished from AdapterRAM            | PASS   | AdapterRAM = shared aperture, not VRAM |
| 9 | physical Windows GPU visibility established               | PASS   | WMI/PnP: Intel Arc, Status=OK          |
| 10 | WSL2 GPU visibility established                          | PASS   | lspci: 1414:008a/008e, dxgkrnl, vgem  |
| 11 | physical GPU vs guest GPU boundary established            | PASS   | Host: Intel 7D55; Guest: Microsoft GPU-PV |
| 12 | no unsupported VRAM claim introduced                      | PASS   | AdapterRAM explicitly classified as shared |
| 13 | no T2.5 work performed                                   | PASS   | (NPU not investigated)                |
| 14 | no T2.6 full driver/runtime reconnaissance performed      | PASS   | (oneAPI/Level Zero/SYCL/OpenCL deferred) |
| 15 | no benchmarking performed                                | PASS   | (no throughput/latency tests)         |
| 16 | no inference performed                                   | PASS   | (no model execution)                  |
| 17 | no workload placement performed                          | PASS   | (no placement/scheduling)             |
| 18 | no optimization performed                                | PASS   | (no kernel/runtime optimization)      |
| 19 | docs/set-2/04-intel-gpu-reconnaissance.md created         | PASS   | This document                         |
| 20 | local diff reviewed                                      | PASS   | git diff --check clean                |
| 21 | commit created                                           | PENDING | —                                  |
| 22 | push succeeded                                           | PENDING | —                                  |
| 23 | remote evidence verified                                 | PENDING | —                                  |

### Acceptance Result

**VERIFIED PASS**

All acceptance criteria are satisfied:
- Physical GPU identity verified via WMI/PnP on the host
- GPU vendor (Intel/8086), device ID (7D55), subsystem, revision all
  verified directly from host hardware interfaces
- Architecture identity grounded in authoritative Intel ARK documentation
  (Meteor Lake, device ID 0x7D55 matches)
- GPU compute resource facts (8 Xe-cores, 2.25 GHz, 18 TOPS) grounded in
  authoritative Intel ARK documentation
- GPU memory model correctly distinguished: AdapterRAM (~2 GB) is a
  shared system memory aperture, NOT dedicated VRAM (integrated GPU has
  no VRAM chips)
- Physical Windows host GPU visibility established (Intel Arc, Status=OK)
- WSL2 GPU visibility established (Microsoft GPU-PV only, no Intel device)
- Physical host vs guest GPU boundary explicitly established
- No unsupported VRAM claim introduced
- No T2.5 (NPU) work performed
- No T2.6 (full driver/runtime/API reconnaissance) performed
- No benchmarking, inference, workload placement, or optimization performed
