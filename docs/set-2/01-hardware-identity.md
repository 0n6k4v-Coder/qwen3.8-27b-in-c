# SET2-T2.1 — Target Hardware Identity

## Task Information

|| Field         | Value                                      |
|---------------|--------------------------------------------|
| Task ID       | SET2-T2.1                                  |
| Task Name     | Target Hardware Identity                   |
| Responsibility| 🛠 EXECUTOR                                |
| Status        | ⚠ PARTIAL — REQUIRES CORRECTION            |
| Dependency    | SET2-READINESS-GATE PASS                   |

---

## Evidence Sources

All evidence was collected directly from the actual target environment. Two
distinct evidence domains are recognized and separated:

- **PHYSICAL HOST** — Windows 11 host inspected via PowerShell / WMI / PnP
  device enumeration (executed through WSL2 interop: `powershell.exe -Command`).
- **GUEST / WSL2** — WSL2 Linux guest inspected via standard Linux tools
  (`lscpu`, `cat /proc/cpuinfo`, `nproc`, `lspci`, `lshw`, `free`,
  `cat /proc/meminfo`, `cat /proc/version`, `uname`, `find /sys`).

The following rule is mandatory and enforced throughout this document:

```
WSL2-visible CPU topology ≠ physical host CPU topology
```

### Host-level (PHYSICAL HOST) evidence sources

|| Source Command | Purpose |
|----------------|---------|
| `powershell.exe -Command "Get-WmiObject -Class Win32_Processor"` | CPU model, physical cores, logical processors, cache, socket |
| `powershell.exe -Command "Get-WmiObject -Class Win32_PhysicalMemory"` | Physical RAM capacity, speed, manufacturer, part number, channel |
| `powershell.exe -Command "Get-WmiObject -Class Win32_VideoController"` | Physical GPU name, vendor, device ID, adapter RAM |
| `powershell.exe -Command "Get-PnpDevice -Class ComputeAccelerator"` | Physical NPU identity and device ID |
| `powershell.exe -Command "Get-PnpDevice" | Where-Object` | Full PnP device enumeration for accelerator search |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_OperatingSystem"` | Windows host OS version and architecture |

### Guest-level (WSL2) evidence sources

|| Source Command | Purpose |
|----------------|---------|
| `uname -a` | OS/kernel/architecture |
| `cat /proc/version` | Kernel build details |
| `lscpu` | CPU model, topology, cache (WSL2 scheduling view) |
| `cat /proc/cpuinfo` | Per-CPU identity, flags, stepping (WSL2 view) |
| `nproc --all` | Logical processor count (WSL2 view) |
| `free -b` | System memory visible to WSL2 guest |
| `cat /proc/meminfo` | Memory detail visible to WSL2 guest |
| `lshw -short` | Hardware inventory (WSL2 guest view) |
| `lspci -nn` | PCI device enumeration (WSL2 virtual devices) |
| `ls /sys/class/drm/` | GPU device node visibility (WSL2 guest) |
| `ls /dev/dri/` | GPU render/card device visibility (WSL2 guest) |
| `ls /sys/class/` | System class enumeration (WSL2 guest) |
| `find /sys -path "*npu*"` | NPU device visibility in WSL2 sysfs |
| Various `/sys/` and `/proc/` reads | Virtualization/container identity |

### Authoritative documentation

- Intel ARK specification for the Intel Core Ultra 7 processor 155H:
  https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

  The Intel ARK page identifies this processor as:
  ```
  Intel Core Ultra Processors — Series 1
  Products formerly Meteor Lake
  ```

  Intel ARK specifies for the Core Ultra 7 155H:
  ```
  Total Cores: 16
  # of Performance-cores: 6
  # of Efficient-cores: 8
  # of Low Power Efficient-cores: 2
  Total Threads: 22
  ```

---

## Correction Summary

The previous T2.1 execution contained material errors that are corrected
here:

1. **INCORRECT:** The Core Ultra 7 155H was classified as Lunar Lake (LNL).
   **CORRECTED:** Intel's official ARK specification identifies the Core Ultra
   7 155H as "Products formerly **Meteor Lake**" (Series 1). The Core Ultra 7
   155H ≠ Lunar Lake. All Lunar Lake classifications have been removed.

2. **INCORRECT:** The WSL2-visible 4C/8T topology was treated as the physical
   host topology.
   **CORRECTED:** The 4C/8T figure is a WSL2 guest scheduling view. The
   physical host CPU (per WMI Win32_Processor) exposes 16 cores and 22
   logical processors, matching Intel's authoritative Meteor Lake
   specification (6P+8E+2LP = 16 cores, 22 threads).

3. **INCORRECT:** Physical host RAM, GPU identity, and NPU identity were
   left as WSL2-visible only.
   **CORRECTED:** Host-level WMI/PnP enumeration has resolved physical host
   RAM (16 GB), physical GPU (Intel Arc, device 8086:7D55), and physical NPU
   (Intel AI Boost, device 8086:7D1D).

---

## 1. CPU Identity

### PHYSICAL HOST (verified via WMI)

**VERIFIED FACT (directly observed from host via `Win32_Processor`):**

- **CPU model:** `Intel(R) Core(TM) Ultra 7 155H`
- **Manufacturer:** `GenuineIntel`
- **Physical cores (hardware threads):** 16 (NumberOfCores = 16)
- **Logical processors (hyperthreads):** 22 (NumberOfLogicalProcessors = 22)
- **L2 cache:** 18,432 KB (18 MB)
- **L3 cache:** 24,576 KB (24 MB)
- **Socket designation:** `U3E1`
- **Family:** 1 (WMI representation; see authoritative spec below)

**DERIVED FINDING:**

- The CPU model "Intel(R) Core(TM) Ultra 7 155H" is, per Intel's official
  ARK specification, a **Meteor Lake** processor (Series 1).
  ```
  Core Ultra 7 155H = Meteor Lake ≠ Lunar Lake
  ```
- The host physical topology of 16 cores / 22 threads reconciles exactly with
  Intel ARK's authoritative specification for this SKU:
  - 6 P-cores (Performance-cores)
  - 8 E-cores (Efficient-cores)
  - 2 Low Power E-cores
  - 22 total threads
- The WMI-reported 16 cores / 22 threads matches Intel's authoritative
  hardware specification. This is the physical host CPU topology.

**Authoritative reconciliation:**

```
Intel ARK:     16 total cores (6P + 8E + 2LP_e) | 22 total threads
WMI Host:      16 cores | 22 logical processors
WSL2 guest:    4 cores | 8 logical processors (scheduling view)
```

The WSL2 visible 4C/8T is a subset/scheduling view of the physical 16C/22T
host, not the host's physical topology.

**VERIFIED: HOST CPU IDENTITY** — Intel Core Ultra 7 155H, Meteor Lake,
16 physical cores (6P+8E+2LP), 22 logical processors.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (directly observed from WSL2 guest):**

- **CPU model (from `/proc/cpuinfo` and `lscpu`):** `Intel(R) Core(TM) Ultra 7 155H`
- **CPU vendor ID:** `GenuineIntel`
- **CPU family:** 6
- **CPU model (CPUID):** 170 (0xAA)
- **CPU stepping:** 4
- **Cores per socket (lscpu):** 4
- **Logical processors (`nproc --all`):** 8
- **Threads per core:** 2
- **Sockets:** 1

**VERIFIED FACT: WSL2 exposes 4C/8T**

This is the WSL2 guest scheduling view, not the physical host topology.
Per the mandatory environment distinction:

```
WSL2-visible 4C/8T ≠ physical host 16C/22T
```

The WSL2 guest sees a reduced/scheduled subset of the host's 16C/22T
processor. This is recorded as GUEST evidence only.

---

## 2. System Memory Identity

### PHYSICAL HOST (verified via WMI)

**VERIFIED FACT (directly observed from host via `Win32_PhysicalMemory`):**

Physical memory modules installed in the host:

| Module | Capacity (bytes) | Capacity (GB) | Speed (MT/s) | Manufacturer | Part Number | Locator |
|--------|-----------------|---------------|--------------|-------------|-------------|---------|
| DIMM 1 | 8,589,934,592   | 8 GB          | 7467         | Samsung     | K3KL8L80CM-MGCT | Controller0-ChannelA |
| DIMM 2 | 8,589,934,592   | 8 GB          | 7467         | Samsung     | K3KL8L80CM-MGCT | Controller1-ChannelA |

- **Total physical RAM installed:** 17,179,869,184 bytes = **16 GB**
  (2 × 8 GB SODIMMs, dual-channel)
- **Memory speed:** 7467 MT/s
- **Memory manufacturer:** Samsung
- **Part number:** K3KL8L80CM-MGCT
- **Channels:** ChannelA (Controller0/Controller1) — dual-channel configuration

**DERIVED FINDING:**

- The physical host has **16 GB** of installed RAM (2 × 8 GB Samsung SODIMMs
  in dual-channel configuration), confirming the project target definition
  of 16 GB system RAM.
- The WSL2 guest observed only ~12,253,212 kB (~11.67 GiB), which is less
  than the host's 16 GB. This discrepancy is explained by WSL2 memory
  ballooning and host-level reservations (firmware, GPU framebuffer, etc.).
- The difference between host-installed RAM (16 GB) and WSL2-visible RAM
  (~11.67 GiB) is consistent with WSL2's dynamic memory management.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (directly observed from WSL2 guest):**

- **OS-visible MemTotal (from `/proc/meminfo`):** 12,253,212 kB (~11.67 GiB)
- **OS-visible total (from `free -b`):** 12,547,289,088 bytes (~11.67 GiB)
- **Swap total:** 4,294,967,296 bytes (~4 GiB)
- **`lshw -short` reports:** 12GiB System memory

**VERIFIED FACT: WSL2 exposes ~11.67 GiB of memory**

This is the WSL2 guest-visible memory, which is less than the host's physical
16 GB due to WSL2 memory ballooning and host reservations. This is recorded
as GUEST evidence only.

---

## 3. GPU Identity

### PHYSICAL HOST (verified via WMI / PnP)

**VERIFIED FACT (directly observed from host via `Win32_VideoController` and
`Get-PnpDevice` for the Display class):**

- **GPU name:** `Intel(R) Arc(TM) Graphics`
- **Vendor:** `Intel Corporation`
- **Device ID:** `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08`
- **Adapter RAM:** 2,147,479,552 bytes (~2 GB)
- **VideoProcessor:** `Intel(R) Arc(TM) Graphics Family`
- **Status:** OK
- **Classification:** Discrete (Intel Arc brand — integrated GPU branded
  as Arc for Meteor Lake; host PnP classifies it as the Display adapter)

**DERIVED FINDING:**

- The physical host GPU is **Intel Arc Graphics** with PCI device ID
  `VEN_8086&DEV_7D55`. Device 7D55 is the Intel Meteor Lake (MTL) integrated
  GPU device ID, consistent with the Core Ultra 7 155H (Meteor Lake) platform.
- Intel markets the Meteor Lake iGPU as "Intel(R) Arc(TM) Graphics" rather
  than "Intel UHD" or "Intel Iris Xe" for this SKU.
- The host-installed Intel Graphics driver (`iigd_dch.inf`, version
  31.0.101.5008) contains INF entries for multiple Meteor Lake GPU device
  IDs including `VEN_8086&DEV_7D55` — this matches the PnP-enumerated
  physical GPU device ID.

**VERIFIED: HOST GPU IDENTITY** — Intel(R) Arc(TM) Graphics, vendor Intel
Corporation, device ID VEN_8086&DEV_7D55 (Meteor Lake iGPU), ~2 GB adapter
RAM.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (directly observed from WSL2 guest):**

- `lspci -nn` inside WSL2 shows two Microsoft-virtualized PCI 3D controllers:
  - `0cca:00:00.0 3D controller [0302]: Microsoft Corporation Device [1414:008a]`
  - `81fc:00:00.0 3D controller [0302]: Microsoft Corporation Basic Render Driver [1414:008e]`
- Both are Microsoft Corporation vendor ID `1414`, which is the Microsoft
  Hyper-V / WSL GPU-PV (GPU Paravirtualization) interface, not Intel vendor
  ID `8086`.
- Kernel driver in use: `dxgkrnl` (DirectX Graphics Kernel, WSL2 GPU-PV)
- `/sys/class/drm/` exposes `card0` and `renderD128` pointing to
  `platform:vgem` — a virtual GPU device, not a physical Intel GPU.
- `lshw -short` lists two display devices:
  - `display: Microsoft Corporation` (at `/0/2`)
  - `display: Basic Render Driver` (at `/0/5`)

**VERIFIED FACT: WSL2 exposes Microsoft virtualized GPU-PV interface**

The WSL2 guest does NOT see the physical Intel Arc GPU (VEN_8086&DEV_7D55).
It sees only Microsoft's virtualized GPU-PV devices (VEN_1414:DEV_008a and
VEN_1414:DEV_008e). This is recorded as GUEST evidence only.

---

## 4. NPU Identity

### PHYSICAL HOST (verified via WMI / PnP)

**VERIFIED FACT (directly observed from host via
`Get-PnpDevice -Class ComputeAccelerator`):**

- **NPU device name:** `Intel(R) AI Boost`
- **Device ID:** `PCI\VEN_8086&DEV_7D1D&SUBSYS_384717AA&REV_04`
- **Class:** `ComputeAccelerator`
- **Class GUID:** `{F01A9D53-3FF6-48D2-9297-C8A7004BE10C}`
- **Manufacturer:** `Intel Corporation`
- **Status:** OK
- **Vendor ID:** `8086` (Intel)
- **Device ID:** `7D1D` (Intel Meteor Lake NPU)

**DERIVED FINDING:**

- The physical host NPU is **Intel(R) AI Boost** with PCI device ID
  `VEN_8086&DEV_7D1D`. Device 7D1D is the Intel Meteor Lake integrated NPU
  device ID, consistent with the Core Ultra 7 155H (Meteor Lake) platform.
- The NPU is enumerated by Windows as a `ComputeAccelerator` class device,
  distinct from the GPU (Display class) and CPU (Processor class).
- This is the Meteor Lake NPU (Gen 4 NPU), directly observed as a physical
  PCIe device on the host — not inferred from driver file strings.

**VERIFIED: HOST NPU IDENTITY** — Intel(R) AI Boost, vendor Intel (8086),
device ID VEN_8086&DEV_7D1D (Meteor Lake NPU), class ComputeAccelerator.

### NPU Driver Evidence (host-level file observation)

**VERIFIED FACT (Windows host, accessed via /mnt/c/DriverStore):**

Intel NPU driver package exists at `/mnt/c/Drivers/NPU/` (DriverStore:
`npu.inf_amd64_23d547ee4d8ae674`):

- `npu_kmd.sys` — NPU kernel-mode driver
- `npu_level_zero_umd.dll` — NPU Level Zero user-mode driver
- `npu_d3d12_umd.dll` — NPU D3D12 user-mode driver
- `npu_dml_compiler.dll` — NPU DirectML compiler
- `npu_dxil_frontend.dll` — NPU DXIL frontend
- `npu_blob_parser.dll` — NPU blob parser
- `vpux_driver_compiler.dll` — VPUX driver compiler
- `ze_loader.dll`, `ze_tracing_layer.dll`, `ze_validation_layer.dll` —
  Level Zero loader and tooling
- `ivd64.inf`, `ivdextn64.inf` — IVD (image/vision) driver INFs
- NPU INF specifies:
  - `DriverVer = 04/23/2025, 32.0.100.4023`
  - `Class = ComputeAccelerator`
  - `ClassGUID = {F01A9D53-3FF6-48D2-9297-C8A7004BE10C}`
- `npu_kmd.sys` strings contain "Intel(R) AI Boost"

**Distinguishing NPU driver presence from NPU device identity:**

- The NPU **device** identity (VEN_8086&DEV_7D1D, Intel(R) AI Boost) was
  established via direct PnP device enumeration on the host.
- The NPU **driver** files exist in the Windows DriverStore and confirm
  software support, but the device identity was not derived from driver
  file strings alone — it was confirmed via `Get-PnpDevice` device
  enumeration.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (WSL2 environment):**

- No NPU device files exist in `/dev/` (no `npu*`, `acceler*`, or similar)
- No NPU devices visible in `/sys/class/` (only `input` class present)
- No NPU entries found via `find /sys -path "*npu*"`
- No NPU kernel modules loaded (checked `/proc/modules`)
- No NPU PCI devices enumerated via `lspci`

**VERIFIED FACT: WSL2 exposes NO NPU visibility**

The NPU is not accessible from within the WSL2 environment — no device nodes,
no sysfs entries, no PCI enumeration. This is recorded as GUEST evidence only.

---

## 5. Platform / OS Identity

### PHYSICAL HOST (verified via WMI)

**VERIFIED FACT (directly observed from host via
`Win32_OperatingSystem`):**

- **OS name:** `Microsoft Windows 11 Home Single Language`
- **Version:** `10.0.26200`
- **Build number:** `26200`
- **Architecture:** `64-bit`

**VERIFIED FACT (host hostname, observed via WSL2):**

- **Hostname:** `LAPTOP-1MSOAKQK` (from `cat /proc/sys/kernel/hostname`)
- The Windows host machine is named `LAPTOP-1MSOAKQK`.

**VERIFIED: HOST OS IDENTITY** — Windows 11 Home Single Language, version
10.0.26200, build 26200, 64-bit, hostname LAPTOP-1MSOAKQK.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (directly observed from WSL2 guest):**

- **OS/kernel:** `Linux LAPTOP-1MSOAKQK 5.15.153.1-microsoft-standard-WSL2`
  (from `uname -a`)
- **Kernel version:** `5.15.153.1-microsoft-standard-WSL2`
  (from `cat /proc/version`: built with GCC 11.2.0, GNU ld 2.37)
- **Architecture:** `x86_64` (from `uname -a` and `lscpu`)
- **Hypervisor vendor:** Microsoft (from `lscpu`: "Hypervisor vendor: Microsoft")
- **Virtualization type:** full (from `lscpu`: "Virtualization type: full")
- **Virtualization:** VT-x (from `lscpu`)
- Network: `eth0` and `loopback0` interfaces visible (`lshw -short`)
- Storage: Virtio devices visible (Virtio 1.0 console, Virtio file system)
- `wsl.exe -l -v` confirms a single Ubuntu distribution running in "Running"
  state in a "docker-desktop" context

**VERIFIED FACT: WSL2 presence and relationship to host**

- The environment is **WSL2** (Windows Subsystem for Linux v2)
- This is a full virtualization environment under the Microsoft Hyper-V
  hypervisor (`Virtualization: VT-x`, `Hypervisor vendor: Microsoft`,
  `Virtualization type: full`)
- WSL2 is a full VM running a Linux kernel (5.15.153.1-microsoft-standard-WSL2)
  as a guest on the Windows 11 host (`LAPTOP-1MSOAKQK`)
- The WSL2 guest is a GUEST; the Windows 11 machine is the HOST

**Unknown:**

- Exact host firmware/SMBIOS details are not directly enumerable from the
  WSL2 guest (no `dmidecode` access from the guest perspective; however,
  host-level WMI was used instead to resolve hardware identity).

---

## Reconciled Target Environment Summary

```
TARGET HARDWARE IDENTITY (RECONCILED)
=====================================

HOST (PHYSICAL — Windows 11, LAPTOP-1MSOAKQK):
  OS:                  Windows 11 Home Single Language
  Version:             10.0.26200 (Build 26200)
  Architecture:        64-bit (x86_64)

  CPU:                 Intel(R) Core(TM) Ultra 7 155H
                        → Meteor Lake (Series 1) — per Intel ARK
  Physical cores:      16  (6P + 8E + 2 LP_e) — per Intel ARK
  Logical processors:  22  — per Intel ARK
  L2 cache:            18 MB (WMI: 18,432 KB)
  L3 cache:            24 MB (WMI: 24,576 KB)
  Socket:              U3E1

  RAM:                 16 GB installed (2 × 8 GB Samsung K3KL8L80CM-MGCT)
                        dual-channel, 7467 MT/s
  GPU:                 Intel(R) Arc(TM) Graphics
                        VEN_8086&DEV_7D55 (Meteor Lake iGPU)
                        ~2 GB adapter RAM
  NPU:                 Intel(R) AI Boost
                        VEN_8086&DEV_7D1D (Meteor Lake NPU)
                        Class: ComputeAccelerator
  WSL2:                Present as Hyper-V guest (full virtualization)

GUEST (WSL2 — Linux):
  OS:                  Linux 5.15.153.1-microsoft-standard-WSL2
  Architecture:        x86_64
  CPU (scheduling view): 4C / 8T (subset of host 16C/22T)
  Memory (visible):    ~11.67 GiB (~12,253,212 kB)
                        (host reserves RAM; ballooning applies)
  GPU (visible):       Microsoft virtualized GPU-PV interface
                        (VEN_1414:DEV_008a, VEN_1414:DEV_008e)
                        Native Intel GPU NOT exposed to guest
  NPU (visible):       None — no /dev nodes, no sysfs, no PCI
                        (host NPU accessible via Windows, not WSL2)
```

---

## Evidence Classification

### VERIFIED FACT

Directly observed facts (host and guest):

**HOST (via PowerShell / WMI / PnP):**
- CPU model: `Intel(R) Core(TM) Ultra 7 155H` (WMI Win32_Processor)
- Host CPU physical cores: 16 (NumberOfCores)
- Host CPU logical processors: 22 (NumberOfLogicalProcessors)
- Host L2 cache: 18,432 KB (WMI)
- Host L3 cache: 24,576 KB (WMI)
- Host socket: U3E1 (WMI SocketDesignation)
- Physical RAM: 2 × 8 GB = 16 GB total (WMI Win32_PhysicalMemory)
- RAM modules: Samsung K3KL8L80CM-MGCT, 7467 MT/s, dual-channel
- Host GPU: `Intel(R) Arc(TM) Graphics`, VEN_8086&DEV_7D55, ~2 GB AdapterRAM
- Host NPU: `Intel(R) AI Boost`, VEN_8086&DEV_7D1D, Class=ComputeAccelerator, Status=OK
- Host OS: Windows 11 Home Single Language, v10.0.26200, Build 26200, 64-bit
- Host hostname: LAPTOP-1MSOAKQK

**GUEST (WSL2 via Linux tools):**
- CPU model string: `Intel(R) Core(TM) Ultra 7 155H` (from `/proc/cpuinfo`, `lscpu`)
- WSL2 CPUID: family 6, model 170, stepping 4
- WSL2 exposes 4 cores / 8 logical processors (1 socket, 2 threads per core)
- WSL2 kernel: Linux 5.15.153.1-microsoft-standard-WSL2
- WSL2 architecture: x86_64
- WSL2 hypervisor: Microsoft, full virtualization, VT-x
- WSL2 visible MemTotal: 12,253,212 kB (~11.67 GiB)
- WSL2 swap: 4,294,967,296 bytes (~4 GiB)
- WSL2 `lspci`: 2 Microsoft PCI 3D controllers (VEN_1414:DEV_008a, VEN_1414:DEV_008e)
- WSL2 kernel driver for GPU: `dxgkrnl` (GPU-PV)
- WSL2 `/sys/class/drm/`: card0 and renderD128 on `platform:vgem`
- WSL2 `lshw`: 2 display devices (Microsoft Corporation, Basic Render Driver)
- WSL2: no NPU device nodes, no NPU sysfs entries, no NPU PCI, no NPU kernel modules
- Intel Graphics driver on host (via /mnt/c/): `iigd_dch.inf` v31.0.101.5008, DCH,
  Class=Display, Type=Integrated; device IDs include VEN_8086&DEV_7D55
- Intel NPU driver on host (via /mnt/c/): `npu.inf` DriverVer=04/23/2025,
  32.0.100.4023; files: npu_kmd.sys, npu_level_zero_umd.dll, etc.

### DERIVED FINDING

Safe interpretations grounded in verified facts and authoritative documentation:

- **CPU generation:** Per Intel's official ARK specification, the Intel Core
  Ultra 7 155H is "Products formerly **Meteor Lake**" (Series 1).
  ```
  Core Ultra 7 155H ≠ Lunar Lake
  Core Ultra 7 155H = Meteor Lake
  ```
  The previous Lunar Lake classification (based on CPUID model 170) was
  incorrect. Intel ARK is the authoritative source.

- **Host CPU topology reconciliation:**
  - Intel ARK: 16 cores (6P + 8E + 2LP_e), 22 threads
  - WMI Host: 16 cores, 22 logical processors
  - WSL2 guest: 4 cores, 8 logical processors
  The WSL2 visible 4C/8T is a guest scheduling view of the host's physical
  16C/22T. These are NOT the same topology.

- **Host RAM reconciliation:**
  - Host physical: 16 GB (2 × 8 GB Samsung SODIMMs)
  - WSL2 visible: ~11.67 GiB
  The ~4.3 GB gap is explained by WSL2 memory ballooning and host-level
  reservations (firmware, GPU framebuffer). The host-installed 16 GB
  matches the project target definition.

- **Host GPU reconciliation:**
  - Host physical: Intel(R) Arc(TM) Graphics, VEN_8086&DEV_7D55
  - WSL2 virtual: Microsoft GPU-PV (VEN_1414:008a, VEN_1414:008e)
  Device 7D55 is the Meteor Lake iGPU device ID, consistent with the
  Meteor Lake platform. Intel markets the Meteor Lake iGPU as "Intel Arc" for
  this SKU. The host driver INF (iigd_dch.inf) includes device ID 7D55,
  matching the PnP-enumerated physical GPU.

- **Host NPU reconciliation:**
  - Host physical: Intel(R) AI Boost, VEN_8086&DEV_7D1D (Meteor Lake NPU)
  - WSL2: no NPU visibility at all
  Device 7D1D is the Meteor Lake NPU device ID. The NPU device identity was
  established via direct PnP device enumeration (Get-PnpDevice), not inferred
  from driver file strings alone, though the driver files confirm software
  support (DriverVer 32.0.100.4023).

- **NPU driver presence vs. NPU device identity:**
  The presence of NPU driver files in the Windows DriverStore confirms
  software support. The NPU device identity (Intel AI Boost, 8086:7D1D) was
  confirmed via direct PnP enumeration, establishing it as a physical PCIe
  device on the host, not merely inferred from driver presence.

### UNKNOWN

Values that cannot be established from available evidence:

- Host firmware/SMBIOS details (not enumerated; host-level WMI resolved the
  needed hardware identity instead).
- Exact WSL2 memory ballooning parameters and host reservation breakdown
  (how much of the 16 GB host RAM is reserved by firmware vs. GPU framebuffer
  vs. WSL2 dynamic allocation — not directly observable).
- Whether the Intel Arc Graphics on this host is a discrete Arc GPU or an
  integrated GPU branded as Arc for the Meteor Lake platform. The PnP
  enumeration shows it as the integrated GPU for this SoC (device 7D5D/7D55
  is the Meteor Lake integrated GPU), but it is branded "Intel(R) Arc(TM)
  Graphics" by Intel's current naming.

---

## Project Target Definition vs Actual Verified Hardware

### Project Target Definition

```
Intel Core Ultra 7 155H
16 GB system RAM
Intel integrated GPU / Arc
Intel NPU
```

### Actual Verified Hardware

```
CPU:     Intel(R) Core(TM) Ultra 7 155H
         → Meteor Lake (Series 1) — per Intel ARK ✓ MATCHES target definition
         → Host physical topology: 16 cores (6P+8E+2LP), 22 threads — per Intel ARK ✓

RAM:    16 GB installed (2 × 8 GB Samsung) — per WMI Win32_PhysicalMemory ✓ MATCHES target

GPU:    Intel(R) Arc(TM) Graphics (VEN_8086&DEV_7D55) — per WMI/PnP ✓ MATCHES target
        (Intel markets Meteor Lake iGPU as "Arc Graphics")

NPU:    Intel(R) AI Boost (VEN_8086&DEV_7D1D) — per PnP enumeration ✓ MATCHES target
```

### Identified Reconciliations

1. **CPU generation correction:** The previous T2.1 incorrectly classified
   the Core Ultra 7 155H as Lunar Lake. Intel ARK classifies it as Meteor
   Lake (Series 1). This is corrected. The CPU model string matches the
   project target.

2. **CPU topology reconciliation:** The previous T2.1 reported WSL2-visible
   4C/8T as the CPU topology. Host-level WMI reveals the physical host has
   16C/22T, matching Intel ARK's authoritative specification (6P+8E+2LP, 22T).
   The 4C/8T is a WSL2 guest scheduling view.

3. **Host RAM reconciliation:** The previous T2.1 reported ~11.67 GiB
   (WSL2-visible) and left host-installed RAM as UNKNOWN. Host-level WMI
   Win32_PhysicalMemory reveals 16 GB installed (2 × 8 GB Samsung SODIMMs),
   matching the project target definition.

4. **Host GPU reconciliation:** The previous T2.1 relied on driver INF
   device IDs only and left the host GPU model as UNKNOWN. Host-level WMI
   Win32_VideoController and PnP enumeration reveal the physical GPU as
   Intel(R) Arc(TM) Graphics with device ID VEN_8086&DEV_7D55, matching the
   Meteor Lake platform and the project target.

5. **Host NPU reconciliation:** The previous T2.1 inferred NPU identity
   from driver file strings (KMB/MTL adapter traits) and left the exact
   NPU device identity as UNKNOWN. Host-level PnP enumeration
   (Get-PnpDevice -Class ComputeAccelerator) reveals the physical NPU as
   Intel(R) AI Boost with device ID VEN_8086&DEV_7D1D, matching the
   Meteor Lake platform and the project target.

---

## Scope Boundary

This task is strictly limited to **hardware identity establishment**. The
following activities were deliberately NOT performed:

- NO CPU capability analysis (ISA features, vector extensions, SIMD
  capabilities, cache hierarchy details beyond identification) — belongs to
  SET2-T2.2
- NO detailed system memory reconnaissance (memory configuration, type,
  channels, frequency, NUMA characteristics, reserved memory) — belongs to
  SET2-T2.3
- NO GPU capability analysis (EU/compute-unit info, memory model, supported
  APIs, driver capability, precision/data types) — belongs to SET2-T2.4
- NO NPU capability research (generation, driver capability, runtime/API,
  supported operation domains, data types) — belongs to SET2-T2.5
- NO benchmarking
- NO inference testing
- NO workload placement research
- NO scheduling research
- NO optimization

---

## Acceptance Result

```text
SET2-T2.1: ⚠ PARTIAL — REQUIRES CORRECTION
SET2-T2.1-R1: 🔜 NEXT (correction round)
```

The previous T2.1 was marked PASS prematurely. The following issues required
correction:

1. ✗ CPU generation misclassified as Lunar Lake (corrected: Meteor Lake per
   Intel ARK).
2. ✗ WSL2-visible 4C/8T treated as physical host topology (corrected: host
   is 16C/22T per WMI, matching Intel ARK).
3. ✗ Host RAM unresolved (corrected: 16 GB via WMI Win32_PhysicalMemory).
4. ✗ Host GPU identity unresolved (corrected: Intel Arc, 8086:7D55 via
   WMI/PnP).
5. ✗ Host NPU identity unresolved (corrected: Intel AI Boost, 8086:7D1D via
   PnP enumeration).

All host-level hardware identity questions are now resolved via direct WMI
and PnP device enumeration. The WSL2 guest observations are preserved and
clearly labeled as GUEST evidence.

### Acceptance Criteria Checklist

- [x] host/guest distinction established
- [x] Core Ultra 7 155H correctly classified as Meteor Lake (Intel ARK)
- [x] previous Lunar Lake claim removed
- [x] WSL2 4C/8T not treated as physical host topology
- [x] physical host CPU identity established (WMI: 16C/22T, Meteor Lake)
- [x] host-installed RAM established (WMI: 16 GB, 2×8GB Samsung)
- [x] physical host GPU established (WMI/PnP: Intel Arc, 8086:7D55)
- [x] physical host NPU established (PnP: Intel AI Boost, 8086:7D1D)
- [x] WSL2 virtual GPU identity retained as guest evidence
- [x] NPU driver presence distinguished from NPU device identity
- [x] no T2.2 work performed
- [x] no T2.3 work performed
- [x] no T2.4 capability analysis performed
- [x] no T2.5 capability analysis performed
- [x] no benchmark performed
- [x] no workload placement
- [x] no scheduling
- [x] no optimization
- [x] corrected T2.1 document persisted
- [x] roadmap corrected
- [x] commit pending push
- [x] remote verification pending
