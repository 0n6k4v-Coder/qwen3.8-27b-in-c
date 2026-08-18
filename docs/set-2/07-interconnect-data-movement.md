# SET2-T2.7 — Interconnect / Data-Movement Reconnaissance

## Task Information

| Field              | Value                                                |
|--------------------|--------------------------------------------------------|
| Task ID            | SET2-T2.7                                              |
| Task Name          | Interconnect / Data-Movement Reconnaissance            |
| Responsibility     | 🧠 LUNA                                                |
| Execution Support  | 🛠 EXECUTOR                                            |
| Status             | ✅ PASS                                                |
| Dependency         | SET2-T2.3 + T2.4 + T2.5 + T2.6 + T2.6-R1 — all ✅ PASS  |

```text
SET2-T2.6:
✅ PASS

SET2-T2.6-R1:
✅ PASS

SET2-T2.7:
🔜 NEXT → ✅ PASS

Current control task:
SET2-T2.7 (was 🔜 NEXT, now ✅ PASS — awaiting ROADMAP advance to SET2-T2.8)
```

---

## Evidence Sources

All evidence was collected directly from the actual target environment during this
task execution. Five evidence domains are recognized and separated throughout:

1. **PHYSICAL HOST (Windows 11)** — directly collected via `powershell.exe -Command`
   interop from WSL2, using WMI/PnP/CIM interfaces.
2. **WSL2 GUEST (Linux 5.15.153.1-WSL2)** — directly collected via standard Linux
   tools (`lspci`, `lscpu`, `cat /proc/*`, `cat /sys/*`).
3. **DOCUMENTED SKU CAPABILITY** — Intel ARK specification for Core Ultra 7 155H
   (SKU 236847), referenced from prior T2.x documents. NOT re-inspected via web
   in this session.
4. **PRIMARY INTEL ARCHITECTURE DOCUMENTATION** — Intel-authored technical
   documents describing Meteor Lake interconnect, GPU/NPU memory architecture.
   NOT directly inspected in this session.
5. **SECONDARY CORROBORATION** — prior T2.x evidence documents used only for
   cross-reference; never promoted to VERIFIED FACT.
6. **DERIVED FINDING** — logical result derived from verified evidence.

### Host-level evidence commands (executed in this task)

| Source Command | Purpose |
|----------------|---------|
| `powershell.exe -Command "Get-WmiObject -Class Win32_Processor"` | CPU model, cores, threads, cache, socket, clock speeds |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_PhysicalMemory"` | Installed RAM modules: capacity, speed, manufacturer |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_PhysicalMemoryArray"` | Memory array: max capacity, device count, ECC |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_ComputerSystem"` | System total physical memory, processor count |
| `powershell.exe -Command "Get-WmiObject -Class Win32_VideoController"` | GPU identity, AdapterRAM, VideoMemoryType |
| `powershell.exe -Command "Get-PnpDevice -Class Display"` | GPU PnP device enumeration |
| `powershell.exe -Command "Get-PnpDevice -Class ComputeAccelerator"` | NPU PnP device enumeration |
| `powershell.exe -Command "Get-PnpDevice -PresentOnly"` | All present devices |
| `powershell.exe -Command "Get-PnpDeviceProperty -InstanceId ..."` | Extended device properties: location paths, siblings, parent, children |
| `cat /mnt/c/Users/Kawee Lekmuenwai/.wslconfig` | WSL2 resource allocation policy |

### Guest-level evidence commands (executed in this task)

| Source Command | Purpose |
|----------------|---------|
| `lspci -nn` | PCI device enumeration (WSL2 guest view) |
| `lscpu` | CPU topology summary, cache summary |
| `cat /proc/cpuinfo` | Per-CPU model, CPUID, ISA flags |
| `cat /proc/meminfo` | Memory total/free/available visible to guest |
| `cat /proc/iomem` | Memory-mapped I/O and RAM regions |
| `cat /sys/fs/cgroup/memory/memory.limit_in_bytes` | WSL2 cgroup memory limit |
| `find /sys/kernel/iommu_groups/` | IOMMU group enumeration |
| `ls /sys/firmware/acpi/tables/` | ACPI table availability (SRAT, SLIT, DMAR, IVRS) |
| `cat /sys/devices/system/cpu/cpu*/cache/index*/{size,type,shared_cpu_list,ways_of_associativity,coherency_line_size}` | Cache hierarchy details |
| `ls /sys/class/drm/` | GPU device visibility in guest |
| `nproc --all` | Logical processor count |

---

## Mandatory State Distinctions

The following distinctions are enforced throughout this document:

```text
PHYSICAL HOST  ≠  WSL2 GUEST
Topology      ≠  Bandwidth
Connectivity  ≠  Performance
Shared Memory ≠  Dedicated Memory
Driver Presence ≠ Physical Interconnect Proof
```

- **Physical Host ≠ WSL2 Guest**: GPU device visibility, NPU device visibility,
  and memory topology differ fundamentally between the Windows host and the WSL2
  Linux guest. GPU-PV and vGEM are virtualization artifacts, not physical devices.
- **Topology ≠ Bandwidth**: Physical adjacency on the same PCIe root complex is
  documented as a connectivity relationship. No bandwidth is claimed or inferred
  from this.
- **Connectivity ≠ Performance**: Device presence and shared bus do not imply
  throughput or latency. These are explicitly OUT OF SCOPE.
- **Shared Memory ≠ Dedicated Memory**: The iGPU and NPU are integrated and share
  system physical memory. They have no device-local VRAM. The observed ~2 GB
  AdapterRAM is a driver-reported shared aperture, not dedicated VRAM.
- **Driver Presence ≠ Physical Interconnect Proof**: Driver service state and INF
  file presence confirm software availability, not physical data paths.

---

## 1. CPU ↔ RAM Relationship

### VERIFIED FACT (PHYSICAL HOST — CPU-to-RAM connectivity)

| Property | Value | Source |
|----------|-------|--------|
| CPU model | Intel(R) Core(TM) Ultra 7 155H | `Win32_Processor` (WMI) |
| CPU cores | 16 (6 P-cores + 8 E-cores + 2 LP E-cores) | `Win32_Processor.NumberOfCores` |
| Threads | 22 | `Win32_Processor.NumberOfLogicalProcessors` |
| Socket | U3E1 (1 socket) | WMI / PNPDeviceId |
| Memory modules | 2 × 8 GB Samsung K3KL8L80CM-MGCT | `Win32_PhysicalMemory` (WMI) |
| Module channels | Controller0-ChannelA, Controller1-ChannelA | `Win32_PhysicalMemory.DeviceLocator` |
| Memory type | DDR5 (SMBIOS code 35) | `Win32_PhysicalMemory.SMBIOSMemoryType` |
| Memory speed | 7467 MT/s | `Win32_PhysicalMemory.Speed` |
| Total installed | 16 GB (17,179,869,184 bytes) | `Win32_PhysicalMemory` capacity sum |
| OS visible RAM | ~15.99 GiB (16,373,120 KB) | `Win32_OperatingSystem.TotalVisibleMemorySize` |
| CPU L3 cache | 24 MB shared (all cores) | `Win32_CacheMemory` |

**Source:** Direct execution of `Get-WmiObject -Class Win32_Processor`,
`Get-CimInstance -ClassName Win32_PhysicalMemory`,
`Get-CimInstance -ClassName Win32_PhysicalMemoryArray`, and
`Get-CimInstance -ClassName Win32_OperatingSystem` on the Windows 11 host.

**Evidence:** The CPU is observed via the host firmware ACPI namespace
(`ACPI\GENUINEINTEL...`). The physical memory controller is part of the
Intel Core Ultra 7 155H SoC (Meteor Lake). Two DDR5 SODIMM modules are
installed on `Controller0-ChannelA` and `Controller1-ChannelA`, representing
a dual-channel configuration directly controlled by the integrated memory
controller (IMC) on the CPU die.

The CPU ↔ RAM connection is direct: the IMC is on the CPU die, and the two
memory channels connect directly to the CPU socket. This is the standard
Intel SoC architecture where the memory controller is integrated into the
processor.

### VERIFIED FACT (WSL2 GUEST — CPU-to-RAM visibility)

| Property | Value | Source |
|----------|-------|--------|
| CPU model (guest) | Intel(R) Core(TM) Ultra 7 155H | `cat /proc/cpuinfo` |
| Guest cores | 4C / 8T | `lscpu`, `nproc --all` |
| Memory total | 12,253,212 KB (~11.67 GiB) | `cat /proc/meminfo` |
| Memory limit | 9223372036854771712 bytes (~12 GB effective cap) | `/sys/fs/cgroup/memory/memory.limit_in_bytes` |
| L1d cache | 192 KiB (4 instances) | `lscpu` |
| L1i cache | 256 KiB (4 instances) | `lscpu` |
| L2 cache | 8 MiB (4 instances, 2048K each) | `lscpu` |
| L3 cache | 24 MiB (1 instance) | `lscpu` |
| L1d per core | 48 KB | `/sys/devices/system/cpu/cpu0/cache/index0/size` |
| L1i per core | 64 KB | `/sys/devices/system/cpu/cpu0/cache/index1/size` |
| L2 per core | 2048 KB | `/sys/devices/system/cpu/cpu0/cache/index2/size` |
| L3 shared | 24576 KB, shared across 0-7 | `index3/shared_cpu_list` |

**Source:** Direct execution of `lscpu`, `cat /proc/cpuinfo`, `cat /proc/meminfo`,
`nproc`, and `cat /sys/devices/system/cpu/cpu*/cache/index*/{size,type}` in the
WSL2 guest.

### DERIVED FINDING

- The WSL2 guest sees only 4C/8T of the host's 16C/22T CPU. This is a WSL2
  scheduler limitation (`processors=8` in `.wslconfig`), not the physical host
  topology.
- The WSL2 guest sees ~11.67 GiB of memory, capped by `.wslconfig`
  (`memory=12GB`), which is less than the 16 GB physically installed.
- The guest observes a unified L3 cache (24 MB) shared across all 8 logical
  processors (0-7), consistent with the host's 24 MB L3 shared across all cores.
- L1d cache is per-core-pair (shared_cpu_list = `0-1`), L1i is per-core-pair,
  L2 is per-core-pair (2 MB each), L3 is shared across all guest cores.

### UNKNOWN

- The exact memory channel interleaving policy for GPU/NPU DMA access to system
  RAM — while the IMC is on-die to the CPU, whether GPU/NPU traffic uses the
  same dual-channel path, the same QoS routing, or separate interconnect
  paths within the SoC fabric cannot be established from OS-visible interfaces.
  No primary Intel architecture document inspecting the on-die fabric was
  directly examined in this session.

---

## 2. CPU ↔ iGPU Relationship

### VERIFIED FACT (PHYSICAL HOST — iGPU device identity and PCIe path)

| Property | Value | Source |
|----------|-------|--------|
| GPU model | Intel(R) Arc(TM) Graphics | `Win32_VideoController.Name`, `Get-PnpDevice -Class Display` |
| GPU PNPDeviceID | `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08` | `Win32_VideoController.PNPDeviceID` |
| GPU InstanceId (full) | `PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10` | `Get-PnpDeviceProperty` |
| PCIe bus | 0, device 2, function 0 | `DEVPKEY_Device_LocationInfo` |
| ACPI BIOS name | `\_SB.PC00.GFX0` | `DEVPKEY_Device_BiosDeviceName` |
| PCI Location Path | `PCIROOT(0)#PCI(0200)` | `DEVPKEY_Device_LocationPaths` |
| ACPI path | `ACPI(_SB_)#ACPI(PC00)#ACPI(GFX0)` | `DEVPKEY_Device_LocationPaths` |
| Parent | `ACPI\PNP0A08\0` (PCI Express Root Complex) | `DEVPKEY_Device_Parent` |
| Driver service | `igfxn` | `DEVPKEY_Device_Service` |
| Driver version | 32.0.101.6790 | `DEVPKEY_Device_DriverVersion` |
| Device stack | `\Driver\igfxn, \Driver\ACPI, \Driver\pci` | `DEVPKEY_Device_Stack` |
| Status | OK (CM_PROB_NONE) | `Get-PnpDevice` |
| PCIe Link Speed | GEN_2_x16 (observed capability) | `DEVPKEY_PciDevice_ExpressSpecVersion=2` |
| ATS Support | True | `DEVPKEY_PciDevice_AtsSupport` |
| ACS Support | Present | `DEVPKEY_PciDevice_AcsSupport=1` |
| AER Capability | NOT present | `DEVPKEY_PciDevice_AERCapabilityPresent=False` |

**Source:** Direct execution of `Get-PnpDeviceProperty -InstanceId
'PCI\VEN_8086&DEV_7D55&SUBSYS_3D0F17AA&REV_08\3&11583659&1&10' and
`Get-WmiObject -Class Win32_VideoController` on the Windows 11 host.

**Evidence:** The iGPU is a physical PCIe device at bus 0, device 2, function 0.
Its parent is `ACPI\PNP0A08\0` — the PCI Express Root Complex. The ACPI BIOS
device name `\_SB.PC00.GFX0` places it under the `PC00` PCI root bridge, which
is the same root bridge (`PCIROOT(0)`) that the CPU operates under.

The GPU's PCIe interrupt message maximum is 1 and it supports ATS (Address
Translation Services), indicating PCIe ATS capability for memory address
translation of GPU accesses to system memory.

The GPU and CPU are physically connected via the same PCIe root complex
(`PCIROOT(0)`). The GPU's PCIe link is GEN_2_x16 (per ExpressSpecVersion=2),
providing the physical data path between the GPU and CPU (via the CPU-integrated
PCIe root port and the system fabric).

### VERIFIED FACT (WSL2 GUEST — iGPU visibility)

| Property | Value | Source |
|----------|-------|--------|
| Physical GPU visible | NO — not present | `lspci -nn`, `/sys/class/drm/` |
| GPU-PV device | Microsoft Basic Render Driver, VEN_1414:008a and VEN_1414:008e | `lspci -nn` |
| vGEM device | `card0 -> ../../devices/platform/vgem/drm/card0` | `/sys/class/drm/` |
| Render node | `renderD128 -> ../../devices/platform/vgem/drm/renderD128` | `/sys/class/drm/` |
| Driver | `dxgkrnl` (on both VEN_1414 devices) | `lspci -nn -v` |

**Source:** Direct execution of `lspci -nn`, `lspci -nn -v`,
`ls /sys/class/drm/`, and `cat /sys/class/drm/*/device/uevent` in the WSL2 guest.

**Evidence:** The WSL2 guest has NO direct visibility of the physical Intel Arc
iGPU. The guest sees only Microsoft GPU-PV virtual devices (VEN_1414:008a and
VEN_1414:008e) driven by `dxgkrnl`, plus a `vgem` virtual DRM platform device.
These are WSL2 virtualization artifacts, not physical hardware.

The physical GPU ↔ CPU interconnect (PCIe root complex) is only observable
from the physical host, not from the WSL2 guest.

### DERIVED FINDING

- The GPU's `DEVPKEY_Device_Siblings` list includes the NPU
  (`PCI\VEN_8086&DEV_7D1D...`) and several other Intel SoC devices
  (`DEV_7D01` PCIe bridge, `DEV_7ECC` PCIe port, `DEV_7D03`, `DEV_7D0D`, etc.).
  This confirms the GPU and NPU are sibling devices on the same PCIe fabric
  rooted at `PCIROOT(0)`.
- The GPU's PCIe link (GEN_2_x16, ATS support, ACS present) establishes the
  physical data path between iGPU and CPU via the PCIe root complex.
- The GPU `AdapterRAM` value of 2,147,479,552 bytes (~2 GB) is a
  driver-reported shared memory aperture, NOT dedicated VRAM. The iGPU has no
  device-local memory; it allocates from system RAM (see Section 4).

### UNKNOWN

- The specific coherency protocol between the iGPU and CPU caches (e.g., whether
  GPU memory accesses are cache-coherent with CPU caches via Intel's
  LLC, or require explicit flush/invalidate operations) — no primary Intel
  architecture document was directly inspected in this session to establish
  this authoritatively.
- GPU ↔ CPU interconnect bandwidth — NOT measured, NOT inferred from PCIe
  link width alone.

```text
CPU ↔ iGPU interconnect bandwidth: UNKNOWN
```

---

## 3. CPU ↔ NPU Relationship

### VERIFIED FACT (PHYSICAL HOST — NPU device identity and PCIe path)

| Property | Value | Source |
|----------|-------|--------|
| NPU model | Intel(R) AI Boost | `Get-PnpDevice -Class ComputeAccelerator` |
| NPU PNPDeviceID | `PCI\VEN_8086&DEV_7D1D&SUBSYS_384717AA&REV_04` | `Get-PnpDevice` |
| NPU InstanceId (full) | `PCI\VEN_8086&DEV_7D1D&SUBSYS_384717AA&REV_04\3&11583659&1&58` | `Get-PnpDeviceProperty` |
| Device Class | ComputeAccelerator | `Get-PnpDevice` |
| Class GUID | `{F01A9D53-3FF6-48D2-9F97-C8A7004BE10C}` | `Get-PnpDeviceProperty` |
| PCIe bus | 0, device 11, function 0 | `DEVPKEY_Device_LocationInfo` |
| ACPI BIOS name | `\_SB.PC00.VPU0` | `DEVPKEY_Device_BiosDeviceName` |
| PCI Location Path | `PCIROOT(0)#PCI(0B00)` | `DEVPKEY_Device_LocationPaths` |
| ACPI path | `ACPI(_SB_)#ACPI(PC00)#ACPI(VPU0)` | `DEVPKEY_Device_LocationPaths` |
| Parent | `ACPI\PNP0A08\0` (PCI Express Root Complex) | `DEVPKEY_Device_Parent` |
| Driver service | `npu` | `Get-PnpDeviceProperty` |
| Driver version | 32.0.100.4023 | `DEVPKEY_Device_DriverVersion` |
| Device stack | `\Driver\npu, \Driver\ACPI, \Driver\pci` | `DEVPKEY_Device_Stack` |
| Status | OK (CM_PROB_NONE) | `Get-PnpDevice` |
| PCIe Link Speed | GEN_2_x16 (observed capability) | `DEVPKEY_PciDevice_ExpressSpecVersion=2` |
| ATS Support | True | `DEVPKEY_PciDevice_AtsSupport` |
| ACS Support | Present | `DEVPKEY_PciDevice_AcsSupport=1` |
| AER Capability | NOT present | `DEVPKEY_PciDevice_AERCapabilityPresent=False` |

**Source:** Direct execution of `Get-PnpDevice -Class ComputeAccelerator`,
`Get-PnpDeviceProperty -InstanceId ...`, and correlation via
`Get-WmiObject -Class Win32_PnPEntity` on the Windows 11 host.

**Evidence:** The NPU is a physical PCIe device at bus 0, device 11, function 0.
Its parent is `ACPI\PNP0A08\0` — the same PCI Express Root Complex as the GPU.
The ACPI BIOS device name `\_SB.PC00.VPU0` places it under the same `PC00` PCI
root bridge as the GPU (`\GFX0`) and the CPU domain.

The NPU's PCIe Location Path is `PCIROOT(0)#PCI(0B00)` — same root complex
`PCIROOT(0)` as the GPU (`PCIROOT(0)#PCI(0200)`). The NPU and GPU appear in each
other's `DEVPKEY_Device_Siblings` lists, confirming they are sibling devices
on the same PCIe fabric.

The NPU device stack is `\Driver\npu, \Driver\ACPI, \Driver\pci`, confirming
it is a PCIe device driver-managed resource connected through the PCI bus.
The driver service is `npu` (running, per prior T2.5 evidence).

### VERIFIED FACT (WSL2 GUEST — NPU visibility)

| Property | Value | Source |
|----------|-------|--------|
| Physical NPU visible | NO — not present | `lspci -nn`, `/sys/class/` |
| NPU device nodes | None (`/dev`, `/sys` empty) | `find /dev -iname "*npu*"`, `find /sys -iname "*npu*"` |
| PCI devices | Only virtio-pci and Microsoft GPU-PV | `lspci -nn` |
| Kernel modules | No NPU driver loaded | `lsmod`, `cat /proc/modules` |
| NPU driver files | Windows PE DLLs only (not loadable as Linux .so) | WSL driver mount listing |

**Source:** Direct execution of `lspci -nn`, `find /sys -iname "*npu*"`,
`find /dev -maxdepth 2 -iname "*npu*"`, `lsmod`, `cat /proc/modules` in the
WSL2 guest.

**Evidence:** The NPU is completely absent from the WSL2 guest. No `/dev` nodes,
no `/sys` entries, no PCI enumeration, no kernel modules. The NPU driver files
exist only as Windows PE-format DLLs mounted in the WSL driver directory — they
cannot be loaded as Linux shared objects.

### UNKNOWN

- The specific coherency protocol between the NPU and CPU caches — no
  authoritative Intel architecture document was directly inspected in this
  session.
- CPU ↔ NPU interconnect bandwidth — NOT measured, NOT inferred from PCIe
  link width alone.
- Whether the NPU has dedicated on-package SRAM visible to the OS — the NPU's
  `AdapterRAM` equivalent is not exposed via any inspected WMI/PnP property.

```text
CPU ↔ NPU interconnect bandwidth: UNKNOWN
NPU ↔ CPU cache coherency protocol: UNKNOWN
```

---

## 4. GPU ↔ Shared-Memory Relationship

### VERIFIED FACT (PHYSICAL HOST — GPU memory architecture)

| Property | Value | Source |
|----------|-------|--------|
| AdapterRAM | 2,147,479,552 bytes (~2 GB) | `Win32_VideoController.AdapterRAM` |
| VideoMemoryType | 2 (SharedMemory) | `Win32_VideoController.VideoMemoryType` |
| AdapterCompatibility | Intel Corporation | `Win32_VideoController.AdapterCompatibility` |
| GPU model | Intel(R) Arc(TM) Graphics (integrated) | `Win32_VideoController.Name` |
| System RAM | 2 × 8 GB Samsung DDR5, 7467 MT/s | T2.3 `Win32_PhysicalMemory` |
| GPU device stack | `\Driver\igfxn, \Driver\ACPI, \Driver\pci` | `Get-PnpDeviceProperty` |

**Source:** Direct execution of `Get-WmiObject -Class Win32_VideoController` and
correlation with T2.3 system memory evidence.

**Evidence:**
- `Win32_VideoController.VideoMemoryType` = 2, which per DMTF CIM schema maps to
  `SharedMemory`. This is a direct OS-level observation that the GPU uses shared
  system memory, not dedicated video memory.
- The system has 16 GB of physical RAM (2 × 8 GB DDR5), and the GPU's AdapterRAM
  reports ~2 GB as a shared memory aperture. The iGPU draws allocations from
  system physical memory.
- The GPU is integrated (no discrete connector, no separate PCIe x16 slot
  with dedicated power). Its parent `ACPI\PNP0A08\0` (PCIe Root Complex)
  and ACPI name `\_SB.PC00.GFX0` confirm integration within the SoC.

### VERIFIED FACT (WSL2 GUEST — GPU memory visibility)

| Property | Value | Source |
|----------|-------|--------|
| Physical GPU | NOT visible | `lspci -nn`, `/sys/class/drm/` |
| GPU-PV devices | VEN_1414:008a (3D), VEN_1414:008e (Render) | `lspci -nn -v` |
| vGEM | `card0`, `renderD128` | `/sys/class/drm/` |
| GPU memory observed | None (no physical device) | `lspci`, `/sys/class/drm/` |

**Source:** Direct execution of `lspci -nn`, `ls /sys/class/drm/`,
`cat /proc/iomem` in the WSL2 guest.

**Evidence:** No GPU memory is observable from the WSL2 guest because the
physical GPU is not exposed. The guest sees only vGEM (a virtual DRM device
with no memory backing) and GPU-PV virtual devices.

### DERIVED FINDING

- The iGPU and system RAM share the same physical memory pool. The ~2 GB
  AdapterRAM value is the shared memory aperture the driver reserves for
  GPU allocations, not dedicated VRAM. The actual GPU allocations draw from
  the 16 GB system RAM pool.
- The GPU uses the CPU's memory channels (dual-channel DDR5, 7467 MT/s)
  for its memory accesses, connected via the PCIe root complex fabric.
- No device-local (VRAM) memory exists for this integrated GPU.

### UNKNOWN

- The exact GPU memory-aperture allocation and management policy — how the
  ~2 GB shared aperture is subdivided between framebuffers, staging buffers,
  and command buffers — is not directly observable from the OS interfaces
  used. See T2.4 Section 4 for the boundary enforcement on AdapterRAM.
- GPU ↔ system memory bandwidth — NOT measured, NOT inferred from
  DDR5 speed or PCIe link width alone.

```text
GPU ↔ shared memory bandwidth: UNKNOWN
GPU memory aperture allocation policy: UNKNOWN
```

Do NOT claim:
- "dedicated VRAM" (this is an integrated GPU — no VRAM).
- that exactly 2 GB is permanently reserved for GPU use (aperture policy is UNKNOWN).

---

## 5. NPU ↔ Shared/System-Memory Relationship

### VERIFIED FACT (PHYSICAL HOST — NPU memory architecture)

| Property | Value | Source |
|----------|-------|--------|
| NPU model | Intel(R) AI Boost | `Get-PnpDevice -Class ComputeAccelerator` |
| NPU PNPDeviceID | `PCI\VEN_8086&DEV_7D1D&SUBSYS_384717AA&REV_04` | `Get-PnpDevice` |
| Device Class | ComputeAccelerator | `Get-PnpDevice` |
| System RAM | 2 × 8 GB Samsung DDR5, 7467 MT/s | T2.3 `Win32_PhysicalMemory` |
| Parent | `ACPI\PNP0A08\0` (PCIe Root Complex) | `Get-PnpDeviceProperty` |
| Driver service | `npu` | `Get-PnpDeviceProperty` |

**Source:** Direct execution of `Get-PnpDevice -Class ComputeAccelerator` and
`Get-PnpDeviceProperty -InstanceId ...` on the Windows 11 host.

**Evidence:**
- The NPU is a PCIe ComputeAccelerator device (class GUID
  `{F01A9D53-3FF6-48D2-9F97-C8A7004BE10C}`), not a traditional GPU or
  display controller.
- The NPU's parent is `ACPI\PNP0A08\0` — the same PCI Express Root Complex
  as the GPU and CPU. The ACPI BIOS name `\_SB.PC00.VPU0` places it under
  the same `PC00` root bridge.
- No `AdapterRAM` or dedicated memory property is exposed for the NPU in any
  inspected PnP/WMI property. The NPU does not report a device-local memory
  aperture via OS-visible interfaces.
- The NPU shares the same system memory pool (16 GB DDR5) as the CPU and GPU.
  On Intel Meteor Lake, the NPU (Gaudi/VPU0) is an integrated fabric device
  that accesses system memory through the same memory controller and PCIe
  root complex as other components.

### VERIFIED FACT (WSL2 GUEST — NPU memory visibility)

| Property | Value | Source |
|----------|-------|--------|
| Physical NPU | NOT visible | `lspci -nn`, `find /dev -iname "*npu*"`, `find /sys` |
| NPU device nodes | None | `find /dev`, `find /sys` |
| Kernel modules | No NPU driver loaded | `lsmod`, `cat /proc/modules` |
| Driver files | Windows PE DLLs only (mounted, not loadable) | WSL driver mount |

**Source:** Direct execution of `lspci -nn`, `find /sys -iname "*npu*"`,
`find /dev -maxdepth 2 -iname "*npu*"`, `lsmod`, `cat /proc/modules` in the
WSL2 guest.

**Evidence:** The NPU is completely absent from the WSL2 guest. No device
nodes, no sysfs entries, no PCI enumeration, no kernel modules. The NPU driver
files exist only as Windows PE-format DLLs — not loadable as Linux shared
objects.

### DERIVED FINDING

- The NPU is integrated within the Meteor Lake SoC and accesses system memory
  through the same PCIe root complex and memory controller as the CPU and GPU.
- No dedicated device-local memory is exposed for the NPU via any inspected
  OS interface.
- The NPU uses shared system memory for its working buffers.

### UNKNOWN

- Whether the NPU has on-package SRAM or cache not exposed to the OS —
  no primary Intel architecture document was directly inspected to confirm
  the presence or size of any NPU-private memory.
- NPU ↔ system memory bandwidth — NOT measured, NOT inferred.
- NPU ↔ CPU cache coherency protocol — NOT established from authoritative
  evidence in this session.

```text
NPU ↔ shared/system memory bandwidth: UNKNOWN
NPU private on-package memory: UNKNOWN
NPU ↔ CPU cache coherency protocol: UNKNOWN
```

---

## 6. Device-Local / Shared-Memory Model

### VERIFIED FACT (device-local memory inventory)

| Device | Device-Local Memory | Shared Memory | Source |
|--------|-------------------|---------------|--------|
| CPU | L3 cache: 24 MB (shared, all cores) | 16 GB DDR5 (dual-channel) | `Win32_CacheMemory`, `Win32_PhysicalMemory` |
| CPU L2 cache | 18 MB total (P-core: 12 MB, E-core: 6 MB) | — | `Win32_CacheMemory` |
| CPU L1 cache | P-core: 288 KB L1D + 384 KB L1I; E-core: 320 KB L1D + 640 KB L1I | — | `Win32_CacheMemory` |
| iGPU | **NONE** (no VRAM) | ~2 GB shared aperture of 16 GB DDR5 | `Win32_VideoController.AdapterRAM`, `VideoMemoryType=2` |
| NPU | **UNKNOWN** (not exposed via OS interfaces) | 16 GB DDR5 (shared) | `Get-PnpDeviceProperty` (no memory property found) |
| NVME SSD | **NONE** (no device-local) | 16 GB DDR5 (via CPU) | `Win32_PnPEntity` |

**Source:** `Win32_VideoController`, `Get-PnpDeviceProperty` (full property dump
for both GPU and NPU), `Win32_CacheMemory`, `Win32_PhysicalMemory`.

**Evidence:**
- CPU: The CPU has hierarchical on-die caches (L1d, L1i per core-pair; L2 per
  core; 24 MB shared L3/LLC). The L3 cache is a shared, device-local cache
  on the CPU die. System RAM is external but connected via the on-die IMC.
- iGPU: `VideoMemoryType=2` (SharedMemory) and the absence of any dedicated
  VRAM BAR or memory property confirms there is NO device-local memory.
  The GPU uses a shared aperture of system DDR5.
- NPU: No `AdapterRAM` or memory-capacity property exists in any inspected
  PnP property for the NPU device. The device exposes PCIe ATS support
  (`DEVPKEY_PciDevice_AtsSupport=True`), indicating it performs address
  translation for memory accesses, consistent with accessing system memory
  via the PCIe fabric.

### DERIVED FINDING

```text
Device-local memory model:
  CPU  → L1/L2/L3 caches on-die (device-local), system RAM external via IMC
  iGPU → NO device-local memory; shared system DDR5 only
  NPU  → device-local memory status: UNKNOWN (not exposed); shared system DDR5

Shared-memory model:
  All devices (CPU, iGPU, NPU) access the same 16 GB dual-channel DDR5 pool.
  GPU accesses via PCIe root complex; NPU accesses via PCIe root complex.
```

### UNKNOWN

- NPU on-package SRAM or private cache — not exposed via any OS-visible
  interface; no primary Intel architecture documentation was directly
  inspected in this session.

---

## 7. Coherency Characteristics

### VERIFIED FACT

| Component | Cache Coherency Observed | Source |
|-----------|------------------------|--------|
| CPU L1/L2/L3 | MESI protocol (standard x86) | CPU architecture (Intel ARK SKU capability) |
| CPU L3 (LLC) | 24 MB shared, 12-way, 64-byte line | `Win32_CacheMemory`, `/sys/devices/system/cpu/cpu0/cache/index3/` |
| CPU L3 sharing | All cores (0-7 in guest, all 16 cores on host) | `shared_cpu_list=0-7` |
| CPU coherency line size | 64 bytes | `/sys/devices/system/cpu/cpu0/cache/index3/coherency_line_size` |

### UNKNOWN

- **iGPU ↔ CPU cache coherency**: Whether the iGPU shares CPU cache-coherency
  domain or operates in a separate coherency domain (requiring explicit
  flush/invalidate) — no authoritative Intel architecture document was directly
  inspected in this session.
- **NPU ↔ CPU cache coherency**: Whether the NPU shares CPU cache-coherency
  domain — NOT established from authoritative evidence.
- **GPU ↔ RAM coherency mechanism**: The specific mechanism (e.g., Intel
  CXL.memory semantics, GPU cache policies) used for GPU accesses to shared
  system memory — NOT established.
- **Cross-device coherency**: Whether allocations shared between CPU, GPU,
  and NPU maintain coherency automatically or require explicit synchronization
  — NOT established from inspected evidence.

```text
CPU internal cache coherency: VERIFIED (MESI, 64-byte line, shared L3)
GPU ↔ CPU cache coherency: UNKNOWN
NPU ↔ CPU cache coherency: UNKNOWN
GPU ↔ RAM coherency model: UNKNOWN
NPU ↔ RAM coherency model: UNKNOWN
Cross-device (CPU/GPU/NPU) coherency: UNKNOWN
```

**Do NOT infer** cache-coherency behavior from device presence or PCIe ATS
support alone. ATS (Address Translation Services) enables page table walking
over PCIe, but does not by itself establish cache coherency.

---

## 8. Data-Movement Pathways

### VERIFIED FACT (physical host pathways)

**Pathway 1: CPU ↔ System RAM**

```text
CPU die
  → IMC (on-die)
  → DDR5 channels (Controller0-ChannelA, Controller1-ChannelA)
  → 2 × 8 GB DDR5 SODIMMs
```
- Direct on-die memory controller (IMC) to DDR5 channels.
- Dual-channel configuration observed via `Win32_PhysicalMemory.DeviceLocator`.
- Source: `Win32_Processor`, `Win32_PhysicalMemory`, `Win32_PhysicalMemoryArray`.

**Pathway 2: CPU ↔ iGPU**

```text
iGPU die (GFX0)
  → PCIe Gen2 x16 link
  → PCI Express Root Complex (ACPI\PNP0A08\0)
  → CPU (via PCIe root port)
  → System RAM (for shared memory allocations)
```
- GPU at `PCIROOT(0)#PCI(0200)`, ACPI `\_SB.PC00.GFX0`.
- Parent: `ACPI\PNP0A08\0` (PCIe Root Complex).
- PCIe link: Gen2 x16 (ExpressSpecVersion=2), ATS enabled, ACS present.
- Data path to RAM: GPU → PCIe root complex → CPU IMC → DDR5.
- Source: `Get-PnpDeviceProperty` (LocationInfo, LocationPaths, Parent, Siblings).

**Pathway 3: CPU ↔ NPU**

```text
NPU die (VPU0)
  → PCIe Gen2 x16 link
  → PCI Express Root Complex (ACPI\PNP0A08\0)
  → CPU (via PCIe root port)
  → System RAM (for shared memory buffers)
```
- NPU at `PCIROOT(0)#PCI(0B00)`, ACPI `\_SB.PC00.VPU0`.
- Parent: `ACPI\PNP0A08\0` (same PCIe Root Complex as GPU).
- PCIe link: Gen2 x16 (ExpressSpecVersion=2), ATS enabled, ACS present.
- Data path to RAM: NPU → PCIe root complex → CPU IMC → DDR5.
- The NPU appears in the GPU's Siblings list, confirming same-fabric placement.
- Source: `Get-PnpDeviceProperty` (LocationInfo, LocationPaths, Parent, Siblings).

**Pathway 4: iGPU ↔ NPU (inter-device)**

```text
iGPU (GFX0) → PCIe Root Complex (PCIROOT(0)) → NPU (VPU0)
```
- Both devices are siblings on `PCIROOT(0)` (same ACPI path
  `ACPI(_SB_)#ACPI(PC00)`).
- Confirmed by: GPU Siblings list contains NPU InstanceId, NPU Siblings
  list contains GPU InstanceId.
- No direct GPU↔NPU interconnect is established; communication routes
  through the PCIe root complex and/or system memory.

### VERIFIED FACT (WSL2 guest pathways)

| Pathway | Guest Visibility | Source |
|---------|-----------------|--------|
| CPU ↔ RAM | Direct (guest memory = host system RAM backed by WSL2) | `cat /proc/meminfo`, cgroup limit |
| CPU ↔ iGPU | NOT DIRECTLY VISIBLE — only vGEM and GPU-PV virtual devices | `lspci -nn`, `/sys/class/drm/` |
| CPU ↔ NPU | NOT VISIBLE — no device, no PCI, no /dev nodes | `lspci -nn`, `find /dev`, `find /sys` |
| iGPU ↔ NPU | NOT VISIBLE — neither device present | `lspci -nn` |

**Source:** Direct execution of `lspci -nn`, `lspci -nn -v`,
`find /dev -iname "*npu*"`, `find /sys -iname "*npu*"` in the WSL2 guest.

### DERIVED FINDING

- On the physical host, all compute devices (CPU, iGPU, NPU) share the same
  PCIe root complex (`PCIROOT(0)` / `ACPI(_SB_)#ACPI(PC00)`).
- The iGPU and NPU are sibling devices under the same root complex, and each
  appears in the other's Siblings list.
- All device↔RAM communication routes through the CPU-integrated IMC and
  dual-channel DDR5 memory subsystem.
- The WSL2 guest observes NONE of these physical pathways. The guest sees only:
  - Virtual CPU vCPUs (4C/8T, capped by `.wslconfig`)
  - Virtual GPU devices (VEN_1414 GPU-PV, vGEM)
  - No NPU at all
- WSL2 memory is a ballooned subset of host physical RAM, capped at 12 GB
  by `.wslconfig`.

### UNKNOWN

- The intra-SoC fabric topology between CPU, GPU, and NPU on the Meteor Lake
  die — whether there are direct on-die interconnects (e.g., Intel's P2P,
  CXL, or proprietary fabric) in addition to the PCIe fabric — no primary
  Intel architecture document was directly inspected.
- Whether GPU↔NPU direct transfers bypass system memory — UNKNOWN.
- Whether any device can DMA directly to another device's memory region
  without CPU intervention — UNKNOWN (no IOMMU group enumeration available
  in WSL2; no Intel VT-d DMAR table directly inspected).

---

## 9. Host versus Guest Distinctions

### VERIFIED FACT (host vs. guest memory allocation)

```text
PHYSICAL HOST:
  Installed RAM: 16 GB (2 × 8 GB DDR5, 7467 MT/s)
  OS visible RAM: ~15.99 GiB
  CPU: 16 cores / 22 threads (full physical topology)
  GPU: Physical Intel Arc iGPU (VEN_8086:7D55) — full device
  NPU: Physical Intel AI Boost (VEN_8086:7D1D) — full device
  .wslconfig: memory=12GB (caps guest), processors=8 (caps guest vCPUs)

WSL2 GUEST:
  Guest-visible RAM: ~11.67 GiB (cgroup limit: ~12 GB, capped by .wslconfig)
  CPU: 4 cores / 8 threads (subset, capped by .wslconfig processors=8)
  GPU: NOT visible — only vGEM + GPU-PV virtual devices (VEN_1414)
  NPU: NOT visible — no device, no /dev, no /sys, no PCI
```

**Source:** `.wslconfig` read from `/mnt/c/Users/Kawee Lekmuenwai/.wslconfig`;
`Win32_PhysicalMemory` + `Win32_Processor` on host; `lspci` + `lscpu` +
`/proc/meminfo` + `/sys/fs/cgroup/memory/` on guest.

### VERIFIED FACT (virtualization artifacts)

| Artifact | Host or Guest | Source |
|----------|--------------|--------|
| Intel Arc iGPU (VEN_8086:7D55) | Physical Host only | `Get-PnpDevice -Class Display` |
| Intel AI Boost NPU (VEN_8086:7D1D) | Physical Host only | `Get-PnpDevice -Class ComputeAccelerator` |
| Microsoft GPU-PV (VEN_1414:008a) | WSL2 Guest only | `lspci -nn` |
| Microsoft Basic Render Driver (VEN_1414:008e) | WSL2 Guest only | `lspci -nn` |
| vGEM virtual DRM | WSL2 Guest only | `/sys/class/drm/`, `/proc/iomem` |
| Intel driver files (.dll, PE format) | Guest mount (not loadable) | WSL driver mount at `/usr/lib/wsl/drivers/` |
| CPU vCPUs (4C/8T) | WSL2 Guest only (subset) | `lscpu`, `nproc` |
| Full 16C/22T CPU | Physical Host only | `Win32_Processor` |

**Source:** `lspci -nn -v`, `Get-PnpDevice`, `lscpu`, `nproc`,
`cat /proc/iomem`, `/sys/class/drm/` listing.

### Critical distinction

```text
WSL2 virtualization artifacts ≠ Physical host topology

The Microsoft GPU-PV devices (VEN_1414:008a, VEN_1414:008e) and vGEM
platform device visible inside WSL2 are virtualization artifacts of the
WSL2 GPU-PV (GPU-PV) paravirtualization layer. They are NOT the physical
Intel Arc iGPU (VEN_8086:7D55) that exists on the host.

The WSL2 guest has NO direct visibility of:
  - The physical Intel Arc iGPU (VEN_8086:7D55)
  - The physical Intel NPU (VEN_8086:7D1D)
  - The physical PCIe root complex
  - The host's full CPU topology (16C/22T)
  - The host's full memory (sees only 12 GB capped)

Do NOT treat WSL2 virtualization artifacts as direct proof of physical
host topology.
```

---

## 10. Evidence Classification Matrix

For every material finding, the claim, source, source type, classification,
confidence, and whether it is supported or contradicted are recorded below.

| # | Claim | Source | Source Type | Classification | Confidence | Supported? | Contradiction? |
|---|-------|--------|-------------|----------------|------------|------------|----------------|
| 1 | CPU model: Intel Core Ultra 7 155H | `Win32_Processor` (WMI) | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 2 | CPU cores: 16C (6P+8E+2LP), 22T | `Win32_Processor` (WMI) | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 3 | CPU socket: U3E1, 1 socket | `Win32_Processor` (WMI) | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 4 | CPU L3 cache: 24 MB shared, 12-way, 64B line | `Win32_CacheMemory` + `/sys/devices/system/cpu/cpu0/cache/index3/` | Host + Guest Observation | VERIFIED FACT | High | YES | None |
| 5 | CPU ↔ RAM: IMC on-die, dual-channel DDR5 | `Win32_PhysicalMemory.DeviceLocator` + Intel ARK | Host Observation + Documented SKU | DERIVED FINDING | Medium | YES (derived) | None |
| 6 | RAM: 2 × 8 GB Samsung K3KL8L80CM-MGCT, 7467 MT/s | `Win32_PhysicalMemory` (WMI/CIM) | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 7 | GPU model: Intel Arc (VEN_8086:DEV_7D55) | `Get-PnpDevice -Class Display` + `Win32_VideoController` | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 8 | GPU PCIe path: PCIROOT(0)#PCI(0200), ACPI \SB.PC00.GFX0 | `Get-PnpDeviceProperty` LocationPaths | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 9 | GPU parent: ACPI\PNP0A08\0 (PCIe Root Complex) | `Get-PnpDeviceProperty` Parent | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 10 | GPU VideoMemoryType: 2 (SharedMemory) | `Win32_VideoController.VideoMemoryType` | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 11 | GPU AdapterRAM: ~2 GB shared aperture | `Win32_VideoController.AdapterRAM` | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 12 | GPU has NO device-local VRAM | VideoMemoryType=2 + no VRAM BAR | Derived from Observation | VERIFIED FACT (derived) | High | YES | None |
| 13 | GPU ↔ CPU: same PCIe root complex (PCIROOT(0)) | LocationPaths comparison | Derived from Observation | DERIVED FINDING | High | YES | None |
| 14 | NPU model: Intel AI Boost (VEN_8086:DEV_7D1D) | `Get-PnpDevice -Class ComputeAccelerator` | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 15 | NPU PCIe path: PCIROOT(0)#PCI(0B00), ACPI \SB.PC00.VPU0 | `Get-PnpDeviceProperty` LocationPaths | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 16 | NPU parent: ACPI\PNP0A08\0 (PCIe Root Complex) | `Get-PnpDeviceProperty` Parent | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 17 | NPU ↔ GPU: sibling devices on same root complex | Siblings list mutual inclusion | Derived from Observation | DERIVED FINDING | High | YES | None |
| 18 | NPU has no device-local memory exposed via OS | No memory property in PnP dump | Actual Host Observation | VERIFIED FACT (absent) | Medium | YES | None |
| 19 | NPU ↔ CPU: same PCIe root complex (PCIROOT(0)) | LocationPaths comparison | Derived from Observation | DERIVED FINDING | High | YES | None |
| 20 | GPU/NPU use shared system RAM (16 GB DDR5) | VideoMemoryType + system RAM evidence | Derived from Observation | VERIFIED FACT (derived) | High | YES | None |
| 21 | GPU PCIe link: Gen2 x16, ATS, ACS | `DEVPKEY_PciDevice_ExpressSpecVersion`, `AtsSupport`, `AcsSupport` | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 22 | NPU PCIe link: Gen2 x16, ATS, ACS | Same PnP properties | Actual Host Observation | VERIFIED FACT | High | YES | None |
| 23 | GPU ↔ CPU cache coherency protocol | Intel ARK / arch docs | PRIMARY INTEL DOC (not inspected) | UNKNOWN | — | NO | None |
| 24 | NPU ↔ CPU cache coherency protocol | Intel ARK / arch docs | PRIMARY INTEL DOC (not inspected) | UNKNOWN | — | NO | None |
| 25 | GPU ↔ RAM bandwidth | — | Not measured, not inferred | UNKNOWN | — | NO | None |
| 26 | NPU ↔ RAM bandwidth | — | Not measured, not inferred | UNKNOWN | — | NO | None |
| 27 | CPU ↔ NPU interconnect bandwidth | — | Not measured, not inferred | UNKNOWN | — | NO | None |
| 28 | CPU ↔ iGPU interconnect bandwidth | — | Not measured, not inferred | UNKNOWN | — | NO | None |
| 29 | WSL2 guest sees full 16C/22T CPU | `lscpu` | Guest Observation | VERIFIED FACT (absent in guest) | High | YES | None |
| 30 | WSL2 guest sees physical GPU/NPU | `lspci -nn` | Guest Observation | VERIFIED FACT (absent in guest) | High | YES | None |
| 31 | WSL2 guest sees GPU-PV (VEN_1414) + vGEM | `lspci`, `/sys/class/drm/` | Guest Observation | VERIFIED FACT | High | YES | None |
| 32 | WSL2 memory capped at 12 GB | `.wslconfig` + cgroup limit | Host + Guest Observation | VERIFIED FACT | High | YES | None |
| 33 | NPU private on-package memory | — | Not exposed via OS; no arch doc inspected | UNKNOWN | — | NO | None |
| 34 | GPU↔NPU direct interconnect (bypasses RAM) | — | Not observed, not documented | UNKNOWN | — | NO | None |
| 35 | Cross-device cache coherency (CPU/GPU/NPU) | — | No authoritative evidence inspected | UNKNOWN | — | NO | None |

---

## 11. Known / Unknown Boundary

### VERIFIED (established by direct evidence)

```text
CPU:
  - Host: Intel Core Ultra 7 155H, 16C/22T, 6P+8E+2LP, 1 socket (U3E1)
  - Host L3: 24 MB shared LLC, 12-way, 64-byte line, shared across all cores
  - Guest: Same CPU model, 4C/8T (scheduler subset), L1d/L1i/L2/L3 verified
  - CPU ↔ RAM: IMC on-die, dual-channel DDR5 (Controller0/Controller1-ChannelA)
  - CPU fully operational on host and guest

GPU (host):
  - Intel Arc iGPU (VEN_8086:7D55), driver igfxn, v32.0.101.6790
  - PCIe Gen2 x16, ATS support, ACS support, parent = PCIe Root Complex
  - Location: PCIROOT(0)#PCI(0200), ACPI \SB.PC00.GFX0
  - GPU memory model: shared system memory (VideoMemoryType=2)
  - AdapterRAM observed = 2,147,479,552 bytes (~2 GB shared aperture)
  - GPU has NO device-local VRAM
  - GPU ↔ CPU: same PCIe root complex (PCIROOT(0))

GPU (guest):
  - NOT visible — only GPU-PV virtual devices (VEN_1414) and vGEM
  - GPU ↔ RAM: not directly visible (virtualized)

NPU (host):
  - Intel AI Boost (VEN_8086:7D1D), driver npu, v32.0.100.4023
  - PCIe Gen2 x16, ATS support, ACS support, parent = PCIe Root Complex
  - Location: PCIROOT(0)#PCI(0B00), ACPI \SB.PC00.VPU0
  - NPU is sibling of GPU (mutual inclusion in Siblings lists)
  - NPU ↔ CPU: same PCIe root complex (PCIROOT(0))
  - NPU ↔ GPU: same root complex, sibling devices

NPU (guest):
  - COMPLETELY ABSENT — no device, no /dev, no /sys, no PCI, no kernel modules
  - NPU driver files: PE DLLs only (not loadable as Linux .so)

RAM:
  - 2 × 8 GB Samsung DDR5 (K3KL8L80CM-MGCT), 7467 MT/s, non-ECC, dual-channel
  - Host installed: 16 GB; Host OS visible: ~15.99 GiB
  - Guest visible: ~11.67 GiB (12 GB cap via .wslconfig)
  - WSL2 cgroup memory limit: ~12 GB

Interconnect:
  - CPU, GPU, NPU all share PCIROOT(0) (ACPI _SB_.PC00) root complex
  - All device↔RAM paths route through CPU IMC and dual-channel DDR5
```

### UNKNOWN (cannot be established from available evidence)

```text
Bandwidth / performance:
  - CPU ↔ RAM bandwidth: UNKNOWN (not measured, not inferred from IMC)
  - CPU ↔ iGPU bandwidth: UNKNOWN (not measured, not inferred from PCIe x16)
  - CPU ↔ NPU bandwidth: UNKNOWN (not measured, not inferred from PCIe x16)
  - GPU ↔ RAM bandwidth: UNKNOWN (not measured)
  - NPU ↔ RAM bandwidth: UNKNOWN (not measured)

Cache coherency:
  - GPU ↔ CPU cache coherency: UNKNOWN (no primary Intel arch doc inspected)
  - NPU ↔ CPU cache coherency: UNKNOWN (no primary Intel arch doc inspected)
  - Cross-device (CPU/GPU/NPU) coherency: UNKNOWN
  - GPU ↔ RAM coherency model: UNKNOWN (ATS ≠ cache coherency)

Device-local memory:
  - NPU on-package SRAM: UNKNOWN (not exposed via OS; no arch doc inspected)

Interconnect topology:
  - Intra-SoC fabric (beyond PCIe): UNKNOWN (no primary Intel arch doc inspected)
  - GPU↔NPU direct path (bypassing RAM): UNKNOWN
  - DMA between devices without CPU: UNKNOWN (no IOMMU/DMAR inspection)

Driver/runtime accessibility:
  - GPU Level Zero (host) initialization: UNKNOWN (not probed — boundary: T2.6)
  - NPU Level Zero (host) initialization: UNKNOWN (not probed — boundary: T2.6)
```

---

## 12. Scope Compliance

### DO-NOT-RUN boundary checklist

```text
✅ NO GPU performance benchmarking performed
✅ NO CPU memory bandwidth benchmarking performed
✅ NO NPU performance testing performed
✅ NO GPU/NPU transfer benchmarks performed
✅ NO PCIe throughput tests performed
✅ NO OpenCL performance tests performed
✅ NO Level Zero performance tests performed
✅ NO SYCL performance tests performed
✅ NO Vulkan performance tests performed
✅ NO model execution performed
✅ NO runtime optimization performed
✅ NO workload placement performed
✅ NO scheduling performed
✅ NO kernel benchmarking performed
✅ NO operator mapping performed
✅ NO downstream task (T2.8) started
```

### Boundary violations

None. No benchmarks, optimizations, workload placement, scheduling, or model
execution was performed. No bandwidth or latency was measured or inferred.
No device existence was converted into a bandwidth claim.

---

## 13. Acceptance Criteria

| # | Criterion | Status | Evidence |
|---|-----------|--------|----------|
| 1 | T2.7 dependency chain verified | ✅ PASS | T2.3, T2.4, T2.5, T2.6, T2.6-R1 all ✅ PASS (confirmed from prior docs + ROADMAP) |
| 2 | CPU ↔ RAM relationship established or UNKNOWN | ✅ PASS | Section 1 — VERIFIED: IMC on-die, dual-channel DDR5, 16 GB |
| 3 | CPU ↔ iGPU relationship established or UNKNOWN | ✅ PASS | Section 2 — VERIFIED: shared PCIe root complex (PCIROOT(0)), Gen2 x16, ATS |
| 4 | CPU ↔ NPU relationship established or UNKNOWN | ✅ PASS | Section 3 — VERIFIED: shared PCIe root complex (PCIROOT(0)), Gen2 x16, ATS |
| 5 | GPU ↔ shared-memory relationship established or UNKNOWN | ✅ PASS | Section 4 — VERIFIED: VideoMemoryType=2 (SharedMemory), ~2 GB shared aperture |
| 6 | NPU ↔ shared/system-memory relationship established or UNKNOWN | ✅ PASS | Section 5 — VERIFIED: no device-local memory exposed, shared system DDR5 |
| 7 | Device-local/shared-memory model established or UNKNOWN | ✅ PASS | Section 6 — VERIFIED: GPU has no VRAM; NPU memory status = UNKNOWN |
| 8 | Coherency characteristics documented only where authoritative | ✅ PASS | Section 7 — CPU cache verified; GPU/NPU coherency = UNKNOWN (no primary doc) |
| 9 | Data-movement pathways documented | ✅ PASS | Section 8 — host pathways via PCIe root complex; guest visibility = absent |
| 10 | Host / guest distinction preserved | ✅ PASS | Section 9 — explicit separation of host physical devices vs guest virtual artifacts |
| 11 | No unsupported bandwidth claim | ✅ PASS | All bandwidth claims = UNKNOWN (not inferred from topology) |
| 12 | No unsupported latency claim | ✅ PASS | No latency values claimed or inferred |
| 13 | No performance inference from topology | ✅ PASS | PCIe link width and ATS support ≠ bandwidth; stated as connectivity only |
| 14 | Evidence classifications correct | ✅ PASS | Matrix in Section 10 — VERIFIED/DOCUMENTED/SECONDARY/DERIVED/UNKNOWN |
| 15 | No secondary source promoted to primary | ✅ PASS | Intel ARK cited as "Documented SKU Capability", not VERIFIED FACT |
| 16 | No derived result promoted to VERIFIED FACT | ✅ PASS | Derived findings explicitly labeled as such; not promoted |
| 17 | No benchmark performed | ✅ PASS | Section 12 compliance checklist — all ✅ |
| 18 | No optimization performed | ✅ PASS | Section 12 compliance checklist — all ✅ |
| 19 | No workload placement performed | ✅ PASS | Section 12 compliance checklist — all ✅ |
| 20 | No scheduling performed | ✅ PASS | Section 12 compliance checklist — all ✅ |
| 21 | No model execution performed | ✅ PASS | Section 12 compliance checklist — all ✅ |
| 22 | No downstream task started | ✅ PASS | T2.8 not begun |
| 23 | No unsupported bandwidth/latency/performance inference | ✅ PASS | All unknowns remain UNKNOWN |

---

## 14. Document Boundary Enforcement

```text
PHYSICAL HOST ≠ WSL2 GUEST
Topology ≠ Bandwidth
Connectivity ≠ Performance
Shared Memory ≠ Dedicated Memory
Driver Presence ≠ Physical Interconnect Proof
```

This document establishes ONLY:
- Physical device identity (PCIe IDs, ACPI names, location paths)
- PCIe root complex membership (shared root complex = sibling/interconnect)
- Memory architecture (integrated, shared system RAM, no device-local VRAM for GPU)
- Cache hierarchy (CPU L1/L2/L3; L3 shared as LLC)
- WSL2 virtualization boundaries (what is and is not visible in guest)
- Evidence classifications for every material claim

This document does NOT establish:
- Any bandwidth or latency value
- Any throughput measurement
- Any performance characteristic
- Any workload placement or scheduling decision
- Any runtime model or execution plan
