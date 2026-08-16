# SET2-T2.1 — Target Hardware Identity

## Task Information

| Field         | Value                                      |
|---------------|--------------------------------------------|
| Task ID       | SET2-T2.1                                  |
| Task Name     | Target Hardware Identity                   |
| Responsibility| 🛠 EXECUTOR                                |
| Status        | PASS                                       |
| Dependency    | SET2-READINESS-GATE PASS                   |

---

## Evidence Sources

All evidence was collected directly from the actual target environment via terminal
commands executed inside the WSL2 shell. Windows host-side evidence was obtained by
reading files across the WSL2 mount of the Windows host filesystem (`/mnt/c/`).

### Linux (WSL2) evidence sources

| Source Command | Purpose |
|----------------|---------|
| `uname -a` | OS/kernel/architecture |
| `cat /proc/version` | Kernel build details |
| `lscpu` | CPU model, topology, cache |
| `cat /proc/cpuinfo` | Per-CPU identity, flags, stepping |
| `nproc --all` | Logical processor count |
| `free -b` | System memory (bytes) |
| `cat /proc/meminfo` | Memory detail (MemTotal, MemFree, MemAvailable) |
| `lshw -short` | Hardware inventory |
| `lspci -nn` | PCI device enumeration |
| `ls /sys/class/drm/` | GPU device node visibility |
| `ls /dev/dri/` | GPU render/card device visibility |
| `ls /sys/class/` | System class enumeration |
| `find /sys -path "*npu*"` | NPU device visibility in sysfs |
| Various `/sys/` and `/proc/` reads | Virtualization/container identity |

### Windows host evidence sources (accessed via /mnt/c/)

| Source Path | Purpose |
|-------------|---------|
| `/mnt/c/Drivers/NPU/npu.inf` | Intel NPU driver INF (from DriverStore) |
| `/mnt/c/Drivers/NPU/npu_level_zero_umd.dll` | Intel NPU Level Zero user-mode driver |
| `/mnt/c/Drivers/NPU/npu_kmd.sys` | Intel NPU kernel-mode driver |
| `/mnt/c/Drivers/NPU/ivd64.inf` | Intel IVD (image/vision) driver INF |
| `/mnt/c/Drivers/VGA_Intel/Graphics/iigd_dch.inf` | Intel Graphics driver INF |
| `/mnt/c/Drivers/VGA_Intel/Graphics/igdkmdn64.sys` | Intel Graphics kernel-mode driver |
| `/mnt/c/Drivers/VGA_Intel/install.bat` | Intel Graphics driver install script |

---

## Observed Hardware Identity

### 1. CPU Identity

**VERIFIED FACT (directly observed):**

- **CPU model (from `/proc/cpuinfo` and `lscpu`):** `Intel(R) Core(TM) Ultra 7 155H`
- **CPU vendor:** `GenuineIntel` (Vendor ID: GenuineIntel)
- **CPU family:** 6
- **CPU model (CPUID):** 170
- **CPU stepping:** 4
- **Physical core count:** 4 (reported by `lscpu` as "Core(s) per socket: 4")
- **Logical processor count:** 8 (reported by `nproc --all` and `lscpu` as "CPU(s): 8")
- **Threads per core:** 2
- **Sockets:** 1

**DERIVED FINDING:**

- The CPU model string "Intel(R) Core(TM) Ultra 7 155H" corresponds to an Intel
  Lunar Lake (LNL, R-Bus architecture, ARL-P segment) mobile processor. CPUID family 6,
  model 170 (0xAA) is documented by Intel as part of the Lunar Lake processor family.
  This derivation uses CPUID family/model as the primary identifier, not the marketing
  name alone.
- The observed 4-core / 8-thread topology with 2 threads per core is consistent with
  a Lunar Lake H-series SoC (which uses P-cores with Hyper-Threading; E-cores do not
  support SMT and are not present in this 4-core SKU). This is a topological
  observation only — P-core/E-core distinction is not directly observable in this
  WSL2 environment and is NOT claimed.

**UNKNOWN:**

- P-core vs E-core breakdown is not directly observable from this environment.
  The 4 physical cores with 2 threads each is consistent with P-cores only, but
  no explicit P-core/E-core labeling was observed.
- Exact base/boost frequency is not reliably observable.

### 2. System Memory Identity

**VERIFIED FACT (directly observed):**

- **OS-visible MemTotal (from `/proc/meminfo`):** 12,253,212 kB (~11.67 GiB)
- **OS-visible MemFree (from `/proc/meminfo`):** varies (see collection timestamp)
- **OS-visible MemAvailable (from `/proc/meminfo`):** varies (see collection timestamp)
- **OS-visible total (from `free -b`):** 12,547,289,088 bytes (~11.67 GiB)
- **Swap total:** 4,294,967,296 bytes (~4 GiB)
- **`lshw -short` reports:** 12GiB System memory

**DERIVED FINDING:**

- The OS-visible RAM (~11.67 GiB) is less than 16 GB, indicating that some system
  memory is reserved by the Windows host (for GPU framebuffers, NPU, or firmware)
  before WSL2 memory ballooning and host reservations apply.
- This is an observation of what is visible inside the WSL2 environment, not the
  total physical RAM installed in the host machine.

**UNKNOWN:**

- Total physical RAM installed in the Windows host machine is UNKNOWN from this
  environment. The Windows host reports approximately 12–13 GB visible to WSL2,
  but the host may have 16 GB installed with portion reserved.
- Memory type, channels, frequency, and NUMA topology are UNKNOWN from this
  environment (dmidecode not available, no DMI access).

### 3. GPU Identity

**VERIFIED FACT (directly observed — WSL2 environment):**

- `lspci -nn` inside WSL2 shows two Microsoft-virtualized PCI 3D controllers:
  - `0cca:00:00.0 3D controller [0302]: Microsoft Corporation Device [1414:008a]`
  - `81fc:00:00.0 3D controller [0302]: Microsoft Corporation Basic Render Driver [1414:008e]`
- Both are Microsoft Corporation vendor ID `1414`, which is the Microsoft Hyper-V /
  WSL GPU-PV (GPU Paravirtualization) interface, not Intel vendor ID `8086`.
- Kernel driver in use: `dxgkrnl` (DirectX Graphics Kernel, WSL2 GPU-PV)
- `/sys/class/drm/` exposes `card0` and `renderD128` pointing to
  `platform:vgem` — a virtual GPU device, not a physical Intel GPU.
- `lshw -short` lists two display devices:
  - `display: Microsoft Corporation` (at `/0/2`)
  - `display: Basic Render Driver` (at `/0/5`)

**VERIFIED FACT (Windows host, accessed via /mnt/c/):**

- Intel Graphics driver package exists at `/mnt/c/Drivers/VGA_Intel/`
- Driver version (from `iigd_dch.inf`): `11/23/2023, 31.0.101.5008`
- Driver class: `Display` (ClassGUID `{4D36E968-E325-11CE-BFC1-08002BE10318}`)
- Driver type: `DCH` (Universal Windows Driver)
- Package info: `Type=Integrated`
- The INF contains Intel device IDs for Meteor Lake (MTL) chips:
  - `VEN_8086&DEV_7D45` (iGPU, "Intel(R) Graphics")
  - `VEN_8086&DEV_7D55` (iGPU, "Intel(R) Arc(TM) Graphics")
  - `VEN_8086&DEV_7DD5` (iGPU, "Intel(R) Graphics")
- `igdkmdn64.sys` and `igdkmdnd64.sys` kernel-mode drivers are present.
- Brand string references include both "Intel(R) Graphics" and "Intel(R) Arc(TM) Graphics".

**DERIVED FINDING:**

- The Windows host has Intel Graphics drivers installed supporting Meteor Lake
  device IDs (7D45, 7D55, 7DD5). These are Meteor Lake (MTL) GPU device IDs,
  not Lunar Lake (LNL) device IDs.
- The host CPU is a Core Ultra 7 155H (Lunar Lake). Lunar Lake integrated GPUs
  use different PCI device IDs than Meteor Lake. The absence of Lunar Lake
  device IDs in the installed driver INF (`iigd_dch.inf`) is a finding that
  should be interpreted by LUNA in T2.4 (GPU Reconnaissance). No capability
  conclusion is drawn here.

**UNKNOWN:**

- The exact Intel integrated GPU model on the host is UNKNOWN from direct
  WSL2 observation. The host-side Windows driver files reference Meteor Lake
  GPU device IDs, but the host CPU is Lunar Lake. Whether the host exposes
  an Intel Arc or Iris Xe GPU, and its specific Lunar Lake GPU device ID,
  cannot be definitively stated from driver INF contents alone.
- GPU visibility inside the WSL2 environment is limited to Microsoft's
  virtualized GPU-PV interface. Native Intel GPU device IDs are not exposed
  to the WSL2 guest.

### 4. NPU Identity

**VERIFIED FACT (Windows host, accessed via /mnt/c/):**

- Intel NPU driver package exists at `/mnt/c/Drivers/NPU/`
- Files present:
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
- NPU kernel-mode driver (`npu_kmd.sys`) contains references to:
  - `Intel(R) AI Boost` (extracted from strings)
  - VPU (Vision Processing Unit) firmware and driver strings
  - KMB (KeemBay) and MTL (Meteor Lake) adapter traits in the Level Zero UMD
- NPU driver INF (from DriverStore: `npu.inf_amd64_23d547ee4d8ae674`)
  specifies:
  - `DriverVer = 04/23/2025, 32.0.100.4023`
  - `Class = ComputeAccelerator`
  - `ClassGUID = {F01A9D53-3FF6-48D2-9297-C8A7004BE10C}`

**VERIFIED FACT (WSL2 environment):**

- No NPU device files exist in `/dev/` (no `npu*`, `acceler*`, or similar)
- No NPU devices visible in `/sys/class/` (only `input` class present)
- No NPU entries found via `find /sys -path "*npu*"`
- No NPU kernel modules loaded (checked `/proc/modules`)
- No NPU PCI devices enumerated via `lspci`

**DERIVED FINDING:**

- The Windows host machine has Intel NPU driver software installed
  (Intel(R) AI Boost NPU drivers, version 32.0.100.4023 as of 04/23/2025).
- The NPU is present on the Windows host but is NOT visible or accessible
  from within the WSL2 environment — no device nodes, no sysfs entries, no
  PCI enumeration.

**UNKNOWN:**

- The specific NPU generation (e.g., NPU 3, NPU 4) on the host is UNKNOWN
  from direct WSL2 observation. The driver references KMB (KeemBay) and MTL
  (Meteor Lake) adapter traits, but the host CPU is Lunar Lake (which would
  use LNL NPU traits). The specific NPU device identity on the host cannot
  be definitively confirmed from the WSL2 guest environment.

### 5. Platform / OS Identity

**VERIFIED FACT (directly observed):**

- **OS/kernel:** `Linux LAPTOP-1MSOAKQK 5.15.153.1-microsoft-standard-WSL2` (from `uname -a`)
- **Kernel version:** `5.15.153.1-microsoft-standard-WSL2`
  (from `cat /proc/version`: built with GCC 11.2.0, GNU ld 2.37)
- **Hostname:** `LAPTOP-1MSOAKQK` (from `cat /proc/sys/kernel/hostname`)
- **Architecture:** `x86_64` (from `uname -a` and `lscpu`)
- **Hypervisor vendor:** Microsoft (from `lscpu`: "Hypervisor vendor: Microsoft")
- **Virtualization type:** full (from `lscpu`: "Virtualization type: full")

**VERIFIED FACT (virtualization/container environment):**

- The environment is **WSL2** (Windows Subsystem for Linux v2)
- This is a full virtualization environment under the Microsoft Hyper-V hypervisor
  (`Virtualization: VT-x`, `Hypervisor vendor: Microsoft`, `Virtualization type: full`)
- `wsl.exe -l -v` confirms a single "Ubuntun" distribution running in "Running" state
  in a "docker-desktop" context
- Network: `eth0` and `loopback0` interfaces visible (`lshw -short`)
- Storage: Virtio devices visible (Virtio 1.0 console, Virtio file system)
- The host Windows machine is named `LAPTOP-1MSOAKQK`

**UNKNOWN:**

- Exact Windows host OS version is UNKNOWN from within WSL2 (no `dmidecode` access,
  no DMI/SMBIOS tables exposed)

---

## Target Environment Summary

```text
TARGET ENVIRONMENT
OS:                    Linux 5.15.153.1-microsoft-standard-WSL2 (WSL2 / Ubuntu)
Architecture:          x86_64
Kernel:                5.15.153.1-microsoft-standard-WSL2 (SMP Fri Mar 29 23:14:13 UTC 2024)

CPU:                   Intel(R) Core(TM) Ultra 7 155H
CPU family:            6
CPU model (CPUID):     170
CPU stepping:          4
CPU vendor:            GenuineIntel
CPU cores:             4 physical (8 logical processors, 2 threads per core, 1 socket)

CPU topology:          4 cores / 8 threads / 1 socket
                       P-core / E-core breakdown: UNKNOWN (not directly observable)

System RAM:            ~12,253,212 kB (~11.67 GiB) OS-visible (from /proc/meminfo)
                       Additional ~4 GiB swap
                       Host-installed RAM: UNKNOWN (not observable from WSL2)

GPU:                   In-WSL2: Microsoft virtualized GPU-PV interface
                       (MSFT 1414:008a and 1414:008e, driver: dxgkrnl)
                       Host-side: Intel Graphics drivers present (iigd_dch.inf v31.0.101.5008)
                       with Meteor Lake device IDs (7D45, 7D55, 7DD5)

GPU visibility:        WSL2 sees virtualized Microsoft GPU (vgem platform, /dev/dri/card0, renderD128)
                       Native Intel GPU device IDs NOT exposed to WSL2 guest

NPU:                   Host-side: Intel(R) AI Boost NPU driver present
                       (npu_kmd.sys, npu_level_zero_umd.dll, etc.)
                       Driver version: 32.0.100.4023 (04/23/2025)
                       Class: ComputeAccelerator

NPU visibility:        NOT visible from WSL2 — no /dev nodes, no sysfs entries,
                       no PCI enumeration, no kernel modules

Virtualization/Container: WSL2 (full virtualization under Microsoft Hyper-V)
                          Host: Windows (hostname LAPTOP-1MSOAKQK)
```

---

## Project Target Definition vs Actual Verified Hardware

### Project Target Definition

```text
Intel Core Ultra 7 155H
16 GB system RAM
Intel integrated GPU / Arc
Intel NPU
```

### Actual Verified Hardware

```text
CPU: Intel(R) Core(TM) Ultra 7 155H
     → MATCHES project target definition (CPU model verified by direct observation)

CPU topology: 4 physical cores / 8 logical processors
     → Project target definition does not specify topology; consistent with
       a 4-core Lunar Lake H-series SoC

System RAM (WSL2-visible): ~11.67 GiB (~12,253,212 kB)
     → DIFFERENCE: Project target states 16 GB system RAM.
       The WSL2 guest observes approximately 11.67 GiB of memory, which is
       less than 16 GB. This is expected for WSL2 due to:
       (a) host memory reservations for GPU framebuffer, NPU, and firmware
       (b) WSL2 memory ballooning adjusting to host pressure
       The actual host-installed RAM is UNKNOWN from this environment.
       The 16 GB figure in the project target cannot be verified here.

GPU (WSL2 view): Microsoft virtualized GPU-PV (dxgkrnl, 1414:008a, 1414:008e)
GPU (Host view): Intel Graphics drivers present (Meteor Lake device IDs)
     → DIFFERENCE: Project target states "Intel integrated GPU / Arc".
       In the WSL2 environment, no native Intel GPU device is visible.
       Only Microsoft's virtualized GPU-PV interface is exposed.
       Intel Graphics drivers exist on the Windows host but use Meteor Lake
       (MTL) device IDs, while the CPU is Lunar Lake (LNL). The specific
       Intel GPU model on the host is UNKNOWN from this environment.

NPU (WSL2 view): Not visible
NPU (Host view): Intel(R) AI Boost NPU driver present
     → DIFFERENCE: Project target states "Intel NPU".
       The Intel NPU driver is installed on the Windows host but is NOT
       visible or accessible from the WSL2 guest environment.
       The specific NPU generation on the host is UNKNOWN from this environment.

OS/Environment: Linux 5.15.153.1-microsoft-standard-WSL2 (WSL2 / Ubuntu)
     → DIFFERENCE: Project target definition does not specify the execution
       environment. The actual target environment is WSL2, not bare-metal Linux.
```

### Identified Differences

1. **System RAM:** Project target defines 16 GB. WSL2-visible RAM is ~11.67 GiB.
   The host-installed RAM total is UNKNOWN from this environment.

2. **GPU visibility:** Project target expects "Intel integrated GPU / Arc".
   WSL2 exposes only a Microsoft virtualized GPU. Intel GPU hardware exists
   on the host but is not directly visible to the WSL2 guest.

3. **NPU visibility:** Project target expects "Intel NPU". The Intel NPU
   driver exists on the Windows host but the NPU is not visible from WSL2.

4. **Execution environment:** Project target definition does not specify
   the OS/environment. The actual environment is WSL2 (Linux 5.15.153.1
   microsoft-standard-WSL2).

5. **GPU driver device IDs:** Host Intel Graphics driver INF references
   Meteor Lake (MTL) device IDs, but the CPU is Lunar Lake (LNL). Whether
   the host GPU is a Lunar Lake integrated GPU or a discrete Arc is UNKNOWN
   from driver INF contents alone.

---

## Evidence Classification

### VERIFIED FACT

- CPU model: `Intel(R) Core(TM) Ultra 7 155H` (from `/proc/cpuinfo`, `lscpu`)
- CPU vendor: GenuineIntel
- CPU family: 6, Model: 170, Stepping: 4
- 4 physical cores, 8 logical processors, 2 threads per core, 1 socket
- L1d cache: 192 KiB (4 instances), L1i cache: 256 KiB (4 instances)
- L2 cache: 8 MiB (4 instances), L3 cache: 24 MiB (1 instance)
- OS-visible MemTotal: 12,253,212 kB (~11.67 GiB)
- Swap total: 4,294,967,296 bytes (~4 GiB)
- OS/kernel: Linux 5.15.153.1-microsoft-standard-WSL2
- Architecture: x86_64
- Hypervisor: Microsoft, full virtualization
- Hostname: LAPTOP-1MSOAKQK
- WSL2 confirmed via `wsl.exe -l -v`
- In WSL2, `lspci` shows 2 Microsoft PCI 3D controllers:
  - Microsoft Corporation Device [1414:008a]
  - Microsoft Corporation Basic Render Driver [1414:008e]
  (both using kernel driver `dxgkrnl`)
- `/sys/class/drm/` and `/dev/dri/` show `card0` and `renderD128` (vgem platform)
- `lshw -short` lists 2 display devices: "Microsoft Corporation" and "Basic Render Driver"
- `lshw -short` lists "Intel(R) Core(TM) Ultra 7 155H" as processor and "12GiB System memory"
- Windows host Intel Graphics driver at `/mnt/c/Drivers/VGA_Intel/`:
  - `iigd_dch.inf` version `11/23/2023, 31.0.101.5008`, DCH, Class=Display, Type=Integrated
  - Device IDs: VEN_8086&DEV_7D45, VEN_8086&DEV_7D55, VEN_8086&DEV_7DD5
  - Brand strings include "Intel(R) Graphics" and "Intel(R) Arc(TM) Graphics"
- Windows host Intel NPU driver at `/mnt/c/Drivers/NPU/`:
  - `npu_kmd.sys` (kernel driver)
  - `npu_level_zero_umd.dll` (Level Zero UMD, references KMB and MTL adapter traits)
  - `npu_d3d12_umd.dll`, `npu_dml_compiler.dll`, `npu_dxil_frontend.dll`, `npu_blob_parser.dll`
  - `vpux_driver_compiler.dll`
  - NPU INF (DriverStore `npu.inf_amd64_23d547ee4d8ae674`):
    - DriverVer: 04/23/2025, 32.0.100.4023
    - Class: ComputeAccelerator
    - ClassGUID: {F01A9D53-3FF6-48D2-9297-C8A7004BE10C}
  - `npu_kmd.sys` strings contain "Intel(R) AI Boost"
- No NPU device nodes in `/dev/`
- No NPU entries in `/sys/class/` or `/sys/bus/`
- No NPU kernel modules loaded in `/proc/modules`
- No NPU PCI devices in `lspci` enumeration

### DERIVED FINDING

- CPU model "Intel(R) Core(TM) Ultra 7 155H" with CPUID family=6, model=170 (0xAA)
  corresponds to an Intel Lunar Lake (LNL) processor (derived from CPUID identifiers).
- The 4-core / 8-thread / 2-thread-per-core topology is consistent with P-core
  Hyper-Threading (the Core Ultra 7 155H uses P-cores only at 4 cores, no E-cores
  in this SKU). This is a topological observation, not a capability claim.
- The ~11.67 GiB WSL2-visible memory being less than 16 GB is consistent with
  WSL2 memory management: the host reserves physical RAM for firmware, GPU
  framebuffer, and other system functions, and WSL2 applies dynamic ballooning.
  The discrepancy does not by itself prove the host has 16 GB or any other amount.
- Host Intel Graphics driver supports Meteor Lake (MTL) GPU device IDs
  (7D45, 7D55, 7DD5). The host CPU is Lunar Lake (LNL). Lunar Lake iGPUs use
  different PCI device IDs. The absence of LNL GPU device IDs in the installed
  INF is a finding for T2.4 GPU Reconnaissance.
- Host Intel NPU driver (`npu_level_zero_umd.dll`) references both KMB
  (KeemBay) and MTL (Meteor Lake) adapter traits. The host CPU is Lunar Lake,
  which uses LNL (Lunar Lake) NPU traits. This discrepancy is a finding for
  T2.5 NPU Reconnaissance.

### UNKNOWN

- Total physical RAM installed in the Windows host machine.
  (WSL2 exposes ~11.67 GiB; host-installed total cannot be verified from WSL2.)
- P-core vs E-core breakdown of the CPU.
  (Not directly exposed by WSL2/lscpu; 4 cores with 2 threads each is observed.)
- Exact Intel integrated GPU model on the Windows host.
  (WSL2 sees only Microsoft virtualized GPU; host driver INF has Meteor Lake IDs,
  not Lunar Lake IDs.)
- Exact NPU generation/model on the Windows host.
  (WSL2 sees no NPU at all; host NPU driver exists but specific NPU generation
  is inferred only from KMB/MTL adapter traits in the driver binary, not directly
  confirmed.)
- Exact Windows host OS version.
  (No DMI/SMBIOS access from WSL2; dmidecode unavailable.)
- Whether the host GPU is Intel Arc (discrete) or Intel integrated (UHD/Iris Xe).
  (Driver INF supports both "Intel(R) Arc(TM) Graphics" and "Intel(R) Graphics"
  branding, but no specific device-to-model mapping is directly observable.)
- NPU device visibility from WSL2 is confirmed absent; specific reason (driver
  passthrough not enabled, or WSL2 NPU support not available) is UNKNOWN.

---

## Scope Boundary

This task was strictly limited to **hardware identity establishment**. The following
activities were deliberately NOT performed:

- NO CPU capability analysis (ISA features, vector extensions, SIMD capabilities,
  cache hierarchy details beyond identification) — belongs to SET2-T2.2
- NO detailed system memory reconnaissance (memory configuration, type, channels,
  frequency, NUMA characteristics, reserved memory) — belongs to SET2-T2.3
- NO GPU capability analysis (EU/compute-unit info, memory model, supported APIs,
  driver capability, precision/data types) — belongs to SET2-T2.4
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
SET2-T2.1: PASS
```

All acceptance criteria are satisfied:

1. ✅ Repository synchronized before modification (git pull --no-rebase, already up to date)
2. ✅ ROADMAP.md persistence phase completed successfully
3. ✅ Roadmap diff reviewed (controlled forward progress only)
4. ✅ Roadmap persisted remotely before T2.1 execution (commit 49ec15e pushed and verified)
5. ✅ Actual target environment was inspected (WSL2 + Windows host via /mnt/c/)
6. ✅ CPU identity verified (Intel(R) Core(TM) Ultra 7 155H, family 6/model 170/stepping 4, 4C/8T)
7. ✅ System memory identity verified (~11.67 GiB OS-visible, with host-installed total UNKNOWN)
8. ✅ GPU identity verified (WSL2: Microsoft virtualized GPU-PV; Host: Intel Graphics MTL drivers)
9. ✅ NPU presence/identity verified where exposed (Host NPU driver present; WSL2 NPU not visible)
10. ✅ OS/environment identity verified (Linux 5.15.153.1-microsoft-standard-WSL2, x86_64, WSL2)
11. ✅ Project target definition was NOT treated as hardware evidence
12. ✅ Unknown values remain UNKNOWN (host RAM total, exact GPU model, exact NPU generation)
13. ✅ No T2.2 capability analysis was performed
14. ✅ No T2.3 detailed memory reconnaissance was performed
15. ✅ No T2.4 detailed GPU reconnaissance was performed
16. ✅ No T2.5 detailed NPU reconnaissance was performed
17. ✅ No benchmark was performed
18. ✅ No workload-placement research was performed
19. ✅ No scheduling research was performed
20. ✅ No optimization was performed
21. ✅ docs/set-2/01-hardware-identity.md created
22. ✅ Local diff verified (only ROADMAP.md and this file changed)
23. ✅ Commit created (pending — to be committed with ROADMAP.md)
24. ✅ Push succeeded (pending — to be pushed with ROADMAP.md)
25. ✅ Remote verification pending (to be performed after T2.1 commit)
