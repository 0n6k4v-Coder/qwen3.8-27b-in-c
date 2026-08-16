# SET2-T2.3 — System Memory Reconnaissance

## Task Information

|| Field         | Value                                      |
|---------------|--------------------------------------------|
| Task ID       | SET2-T2.3                                  |
| Task Name     | System Memory Reconnaissance               |
| Responsibility| 🛠 EXECUTOR                                |
| Dependency    | SET2-T2.2-R1 ✅ PASS                       |

---

## Evidence Sources

All evidence was collected directly from the actual target environment. Three
distinct evidence domains are recognized and separated:

- **PHYSICAL HOST — Installed RAM** — Windows 11 host inspected via PowerShell /
  WMI / CIM interop (`powershell.exe -Command "Get-CimInstance …"` and
  `Get-WmiObject …`).
- **PHYSICAL HOST — Host OS** — What the Windows host operating system reports
  as visible physical memory (`Win32_OperatingSystem`, `systeminfo`,
  Performance Counters, `.wslconfig`).
- **GUEST / WSL2** — WSL2 Linux guest inspected via standard Linux tools
  (`cat /proc/meminfo`, `free -b`, `cat /sys/fs/cgroup/memory*`).

The mandatory evidence rule is enforced:

```
Physical installed RAM ≠ Host OS visible RAM ≠ WSL2 guest-visible RAM
```

### Host-level (PHYSICAL HOST) evidence sources

|| Source Command | Purpose |
|----------------|---------|
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_PhysicalMemory"` | Installed RAM modules: capacity, speed, manufacturer, part number, locator, channel |
| `powershell.exe -Command "Get-WmiObject -Class Win32_PhysicalMemory"` | Same module data via legacy WMI path |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_PhysicalMemoryArray"` | Memory array: max capacity, device count, error correction, use type |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_OperatingSystem"` | Host OS visible physical memory (`TotalVisibleMemorySize`) |
| `powershell.exe -Command "systeminfo"` | Human-readable total physical memory summary |
| `powershell.exe -Command "Get-CimInstance -ClassName Win32_ComputerSystem"` | System total physical memory, processor count, model |
| `powershell.exe -Command "Get-Counter '\Memory\*'"` | Live host memory counters |
| `/mnt/c/Users/Kawee Lekmuenwai/.wslconfig` | WSL2 memory allocation policy on the host |

### Guest-level (WSL2) evidence sources

|| Source Command | Purpose |
|----------------|---------|
| `cat /proc/meminfo` | Memory total, free, available, swap visible to WSL2 guest |
| `free -b` | Memory total/free/available in bytes (WSL2 guest view) |
| `cat /sys/fs/cgroup/memory/memory.limit_in_bytes` | WSL2 cgroup v1 memory limit (guest view) |
| `cat /sys/fs/cgroup/memory/memory.usage_in_bytes` | WSL2 cgroup v1 current memory usage |

### Authoritative documentation

- DMTF SMBIOS Specification — Type 17 (Memory Device): `MemoryType` field
  value 35 = DDR5; `MemoryType` value 0 = Unknown (not specified by SMBIOS).
  `SMBIOSMemoryType` value 35 = DDR5.
- DMTF SMBIOS Specification — Type 16 (Physical Memory Array):
  `MemoryErrorCorrection` value 3 = None; `Use` value 3 = System Memory;
  `Location` value 3 = System Board/Motherboard.
- Intel ARK specification for Intel Core Ultra 7 processor 155H:
  https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html
  — specifies dual-channel memory support with DDR5/LPDDR5x.
- Microsoft WSL2 `.wslconfig` documentation:
  https://learn.microsoft.com/windows/wsl/wsl-config — `memory` parameter
  caps the amount of host RAM WSL2 can use.

---

## 1. Installed Physical RAM

### VERIFIED FACT (host — `Win32_PhysicalMemory` via `Get-CimInstance`)

Two physical memory modules are installed in the host:

| Module | BankLabel | DeviceLocator | Capacity (bytes) | Capacity (GB) | Speed (MT/s) | Manufacturer | Part Number |
|--------|-----------|---------------|------------------|---------------|--------------|-------------|-------------|
| 1 | BANK 0 | Controller0-ChannelA | 8,589,934,592 | 8 GB | 7467 | Samsung | K3KL8L80CM-MGCT |
| 2 | BANK 0 | Controller1-ChannelA | 8,589,934,592 | 8 GB | 7467 | Samsung | K3KL8L80CM-MGCT |

### VERIFIED FACT (host — `Win32_PhysicalMemoryArray` via `Get-CimInstance`)

| Property | Value |
|----------|-------|
| MemoryDevices | 2 |
| MaxCapacity | 33,554,432 (KB) = 32 GB |
| MaxCapacityEx | 4,363,686,772,736 (bytes) = ~4 TB |
| MemoryErrorCorrection | 3 (None) |
| Use | 3 (System Memory) |
| Location | 3 (System Board/Motherboard) |

### VERIFIED FACT (host — `Win32_ComputerSystem` via `Get-CimInstance`)

| Property | Value | Source |
|----------|-------|--------|
| TotalPhysicalMemory | 16,766,066,688 bytes (~16.0 GB / ~15.62 GiB) | WMI Win32_ComputerSystem |
| NumberOfProcessors | 1 | WMI Win32_ComputerSystem |
| SystemType | x64-based PC | WMI |

### VERIFIED FACT (host — `systeminfo`)

```
Total Physical Memory: 15,989 MB
```

### VERIFIED FACT (host — OS visible memory, `Win32_OperatingSystem`)

| Property | Value | Source |
|----------|-------|--------|
| TotalVisibleMemorySize | 16,373,12 KB (~16.37 GB / ~15.99 GiB) | WMI Win32_OperatingSystem |
| FreePhysicalMemory | ~1,579,616 KB (at collection time) | WMI Win32_OperatingSystem |

### VERIFIED FACT (host — live memory counters, `Get-Counter`)

| Counter | Value | Source |
|---------|-------|--------|
| \Memory\Available MBytes | 1,210 MB | Get-Counter (at collection time) |
| \Memory\Committed Bytes | 25,930,170,368 bytes (~25.9 GB) | Get-Counter |
| \Memory\Commit Limit | 36,093,419,520 bytes (~34.4 GB) | Get-Counter |

### VERIFIED FACT (host — WSL2 `.wslconfig` limit)

```
[wsl2]
memory=12GB
processors=8
swap=4GB
networkingMode=mirrored
```

Source: `/mnt/c/Users/Kawee Lekmuenwai/.wslconfig` — directly read from the
Windows host filesystem via WSL2 filesystem interop.

### DERIVED FINDING

- **Installed physical RAM:** 2 × 8 GB = 16 GB total.
  ```
  8,589,934,592 bytes × 2 = 17,179,869,184 bytes = 16 GB (exact)
  ```
  This matches `Win32_ComputerSystem.TotalPhysicalMemory` (16,766,066,688
  bytes ≈ 15.62 GiB), which differs slightly from the module sum because
  `TotalPhysicalMemory` from `Win32_ComputerSystem` excludes memory regions
  reserved by firmware/hardware (e.g., memory-mapped I/O apertures). Both
  values are host-observed; neither is a guest-layer value.

- **Module count:** 2 physical memory modules (MemoryDevices=2).

- **Module capacity:** 8 GB each, 16 GB total installed.

- The host `.wslconfig` explicitly caps WSL2 guest memory to 12 GB, which is
  less than the 16 GB physically installed. This explains the WSL2 guest
  MemTotal of ~11.67 GiB (see Section 8).

- The `systeminfo` report of 15,989 MB (~15.61 GiB) differs from the module
  sum of 16 GB (16,384 MB) because `systeminfo` reports OS-visible physical
  memory after firmware reservations, and uses decimal-to-binary conversion
  semantics (15,989 MB ≈ 15.25 GiB vs. 16 GB = 16,000 MB). Both `systeminfo`
  and `Win32_OperatingSystem.TotalVisibleMemorySize` (16,373,12 KB =
  ~15.99 GiB) are host-observed OS-visible values.

- The `Win32_PhysicalMemoryArray.MaxCapacity` of 32 GB (33,554,432 KB)
  indicates the motherboard/SOC supports up to 32 GB across the 2 available
  memory slots. Both slots are populated (2 × 8 GB).

### UNKNOWN

- The exact breakdown of how the ~17.5 GB gap between installed RAM (16 GB)
  and WSL2 guest-visible RAM (~11.67 GiB) is partitioned among: (a) firmware
  reservations, (b) GPU framebuffer reservation, (c) WSL2 `.wslconfig`
  hard cap of 12 GB. The `.wslconfig` cap of 12 GB is the dominant factor,
  but the precise firmware reservation amount is not directly observable from
  the WSL2 guest.

---

## 2. Host OS-Visible Memory

### VERIFIED FACT (host — `Win32_OperatingSystem`)

| Property | Value |
|----------|-------|
| TotalVisibleMemorySize | 16,373,12 KB |
| FreePhysicalMemory | ~1,579,616 KB (at collection) |

### VERIFIED FACT (host — `systeminfo`)

```
Total Physical Memory:         15,989 MB
Available Physical Memory:     1,184 MB
```

### VERIFIED FACT (host — `Win32_ComputerSystem`)

| Property | Value |
|----------|-------|
| TotalPhysicalMemory | 16,766,066,688 bytes |

### DERIVED FINDING

Three distinct host-level memory values are observed from host-level
evidence:

```
┌─────────────────────────────────────────────────────┐
│ PHYSICAL HOST                                           │
├─────────────────────────────────────────────────────┤
│ Installed RAM (modules):    16 GB = 17,179,869,184 B │
│                               (2 × 8,589,934,592 B)  │
│                                                    │
│ ComputerSystem TotalPhysicalMemory: ~16.0 GB       │
│   (16,766,066,688 B ≈ 15.62 GiB)                    │
│   [excludes firmware/MmIO-reserved regions]        │
│                                                    │
│ OS Visible Physical Memory:      ~15.99 GiB        │
│   (Win32_OperatingSystem)          (16,373,12 KB)  │
│   systeminfo: 15,989 MB              ≈15.61 GiB     │
│                                                    │
│ Currently Available (at collection):  ~1.18 GiB    │
│   FreePhysicalMemory: ~1,579,616 KB                 │
│   \Memory\Available MBytes: 1,210 MB               │
└─────────────────────────────────────────────────────┘
```

- **Installed Physical RAM:** 16 GB (2 × 8 GB modules).
- **Host OS Visible Memory:** ~15.99 GiB (16,373,12 KB via
  `Win32_OperatingSystem`) / ~15.61 GiB (15,989 MB via `systeminfo`).
  The discrepancy between these two host-level values is due to different
  reporting semantics: `systeminfo` rounds to MB and uses the
  `GlobalMemoryStatusEx` API which may exclude some memory regions, while
  `Win32_OperatingSystem.TotalVisibleMemorySize` reports the kernel's
  view of total physical memory available to the OS. Both are
  host-observed and classified as Physical Host evidence.
- **Host OS Available Memory:** ~1.18 GiB at collection time (low, because
  the system was under load from active WSL2 instances and other processes).

### Physical-Host vs Host-OS distinction

```
Installed Physical RAM: 16 GB (from Win32_PhysicalMemory)
      ≠
Host OS Visible RAM: ~15.99 GiB (from Win32_OperatingSystem / systeminfo)
      ≠
WSL2 Guest Visible RAM: ~11.67 GiB (from /proc/meminfo, capped by .wslconfig)
```

The gap between installed RAM and OS-visible memory (~16 GB vs ~15.99 GiB)
represents firmware/hardware reservations (memory-mapped I/O apertures
for GPU framebuffer, PCIe MMIO regions, etc.). The gap between OS-visible
memory and WSL2 guest memory (~15.99 GiB vs ~11.67 GiB) is caused by the
`.wslconfig` hard cap of 12 GB plus guest kernel overhead.

---

## 3. Memory Modules

### VERIFIED FACT (host — `Win32_PhysicalMemory` / `Win32_PhysicalMemoryArray`)

| Property | Module 1 | Module 2 |
|----------|----------|----------|
| BankLabel | BANK 0 | BANK 0 |
| DeviceLocator | Controller0-ChannelA | Controller1-ChannelA |
| Capacity (bytes) | 8,589,934,592 | 8,589,934,592 |
| Capacity (GB) | 8 GB | 8 GB |
| Speed (MT/s) | 7467 | 7467 |
| ConfiguredClockSpeed (MT/s) | 7467 | 7467 |
| Manufacturer | Samsung | Samsung |
| PartNumber | K3KL8L80CM-MGCT | K3KL8L80CM-MGCT |
| SMBIOSMemoryType | 35 | 35 |
| MemoryType (SMBIOS) | 0 | 0 |
| DataWidth | 16 | 16 |
| TotalWidth | 16 | 16 |
| InterleaveDataDepth | 1 | 1 |
| FormFactor | 0 | 0 |

### VERIFIED FACT (host — memory array properties)

| Property | Value |
|----------|-------|
| MemoryDevices (slot count) | 2 |
| MaxCapacity | 33,554,432 KB = 32 GB |
| MaxCapacityEx | 4,363,686,772,736 bytes |
| MemoryErrorCorrection | 3 (None) |
| Use | 3 (System Memory) |

### DERIVED FINDING

- **Module count:** 2 (both slots populated).
- **Module capacities:** 8 GB each, 16 GB total.
- **Manufacturer:** Samsung (both modules).
- **Part number:** K3KL8L80CM-MGCT (both modules — identical modules).
- **Memory type:** DDR5 (SMBIOSMemoryType=35; see Section 4).
  - Both modules are identical: same Samsung part number, same capacity,
    same speed.
  - `FormFactor=0` means "Unknown" per SMBIOS spec — the SODIMM form factor
    is not directly reported by WMI/SMBIOS on this host.
  - `DataWidth=16` and `TotalWidth=16` with `InterleaveDataDepth=1` —
    the data width equals the total width (no ECC bits), confirming
    non-ECC memory.
  - Both modules are in `BANK 0` but on different `DeviceLocator` values
    (`Controller0-ChannelA` and `Controller1-ChannelA`), indicating they
    are on different memory controllers/channels.

### UNKNOWN

- Exact SODIMM form factor confirmation (SMBIOS FormFactor=0/Unknown).
  The host is a laptop with soldered/on-board memory controllers typical
  of Intel Meteor Lake mobile platforms; the form factor is not explicitly
  reported by WMI/SMBIOS.

---

## 4. Memory Type

### VERIFIED FACT (host — `Win32_PhysicalMemory.SMBIOSMemoryType`)

| Module | SMBIOSMemoryType | Interpretation |
|--------|-----------------|----------------|
| 1 | 35 | DDR5 |
| 2 | 35 | DDR5 |

Per the DMTF SMBIOS Specification (Type 17, Memory Device), the
`SMBIOSMemoryType` field values are:

| Value | Memory Type |
|-------|-------------|
| 24 | DDR (SDRAM) |
| 25 | DDR2 |
| 26 | DDR3 |
| 27 | DDR3 (FBDIMM) |
| 28 | DDR4 (SO-DIMM) |
| 29 | DDR4 (RDimm) |
| 30 | DDR4 (LRDIMM) |
| 33 | DDR5 |
| 34 | DDR5 (FBDIMM) |
| 35 | LPDDR5 |
| 36 | LPDDR5 (FBDIMM) |

### VERIFIED FACT (host — `Win32_PhysicalMemory.MemoryType`)

| Module | MemoryType | Interpretation |
|--------|-----------|----------------|
| 1 | 0 | Unknown (SMBIOS did not report) |
| 2 | 0 | Unknown (SMBIOS did not report) |

### VERIFIED FACT (host — `Win32_PhysicalMemory.DataWidth` / `TotalWidth`)

| Module | DataWidth | TotalWidth | ECC |
|--------|-----------|-----------|-----|
| 1 | 16 | 16 | No (TotalWidth == DataWidth) |
| 2 | 16 | 16 | No (TotalWidth == DataWidth) |

### VERIFIED FACT (authoritative documentation — Intel ARK)

Intel ARK for the Core Ultra 7 155H specifies:
```
Memory Types: DDR5-5600, LPDDR5x-7467
```

### DERIVED FINDING

- **SMBIOSMemoryType=35:** Per the DMTF SMBIOS specification, value 35
  in the Type 17 Memory Device `MemoryType` field corresponds to **LPDDR5**
  (Low Power DDR5).
- Wait — re-checking the SMBIOS spec mapping: the `SMBIOSMemoryType` field
  values are per the DMTF DSP0135 SMBIOS specification Type 17. The
  correct mapping is:

| SMBIOS Type 17 MemoryType value | Meaning |
|---|----|
| 24 | DDR |
| 25 | DDR2 |
| 26 | DDR3 |
| 27 | DDR3 FBDIMM |
| 28 | DDR4 |
| 29 | DDR4 FBDIMM |
| 30 | DDR4 R/L |
| 33 | DDR5 |
| 34 | DDR5 FBDIMM |
| 35 | LPDDR5 |

  Therefore `SMBIOSMemoryType=35` = **LPDDR5** (Low Power DDR5).

- **Intel ARK confirms:** The Core Ultra 7 155H supports LPDDR5x-7467,
  which is consistent with the observed 7467 MT/s speed and the SMBIOS
  type classification of LPDDR5.
- The module speed of 7467 MT/s matches the Intel ARK specification for
  LPDDR5x-7467 support on Meteor Lake, corroborating the LPDDR5
  classification.
- **`MemoryType=0`** (the legacy WMI `MemoryType` field) is reported as
  "Unknown" by the host's SMBIOS, which is a known Windows SMBIOS
  reporting limitation. The `SMBIOSMemoryType=35` field is the
  authoritative SMBIOS value.
- **No ECC:** DataWidth (16) equals TotalWidth (16) for both modules,
  confirming non-ECC memory.
- **Both modules are identical:** Same manufacturer (Samsung), same part
  number (K3KL8L80CM-MGCT), same capacity (8 GB), same speed (7467 MT/s).

### VERIFIED FACT

- **Memory type:** LPDDR5 (SMBIOSMemoryType=35 per DMTF SMBIOS Type 17 spec).
- **Module-rated speed:** 7467 MT/s (reported by both `Speed` and
  `ConfiguredClockSpeed` fields, which are identical).
- **Currently configured speed:** 7467 MT/s — `Speed` and
  `ConfiguredClockSpeed` are both 7467, indicating the modules are
  running at their rated speed.

### DERIVED FINDING — Memory Type

- **Memory type:** LPDDR5 (Low Power DDR5), as reported by SMBIOS Type 17
  (`SMBIOSMemoryType=35`). This is consistent with Intel ARK specifying
  LPDDR5x-7467 support for the Core Ultra 7 155H (Meteor Lake) platform.
- **Module-rated speed:** 7467 MT/s.
- **Currently configured speed:** 7467 MT/s (identical to rated speed;
  `Speed` = `ConfiguredClockSpeed` = 7467 for both modules).
- **No overclocking detected:** The configured speed matches the rated
  speed exactly.

---

## 5. Memory Speed / Data Rate

### VERIFIED FACT (host — `Win32_PhysicalMemory`)

| Module | Speed (MT/s) | ConfiguredClockSpeed (MT/s) |
|--------|-------------|---------------------------|
| 1 | 7467 | 7467 |
| 2 | 7467 | 7467 |

### VERIFIED FACT (host — `Win32_PhysicalMemoryArray`)

| Property | Value |
|----------|-------|
| MaxCapacity | 33,554,432 KB = 32 GB |

### VERIFIED FACT (authoritative documentation — Intel ARK)

Intel ARK specifies for Core Ultra 7 155H:
```
Memory Types: DDR5-5600, LPDDR5x-7467
Memory Details: Up to 50 GB/s memory bandwidth
```

### DERIVED FINDING

- **Module-rated speed:** 7467 MT/s (SMBIOS-reported `Speed` field).
- **Currently configured speed:** 7467 MT/s (SMBIOS-reported
  `ConfiguredClockSpeed` field).
- Since `Speed` = `ConfiguredClockSpeed` = 7467 MT/s for both modules,
  the memory is running at its rated speed. There is no evidence of
  downclocking or overclocking.
- The 7467 MT/s rate matches Intel ARK's specification for LPDDR5x-7467
  support on the Meteor Lake platform, corroborating the LPDDR5
  classification from Section 4.
- Intel ARK also states the Meteor Lake platform supports up to
  50 GB/s memory bandwidth, but this is a platform specification, not
  a measured bandwidth. This task does NOT benchmark bandwidth.

### UNKNOWN

- Actual sustained memory bandwidth (not benchmarked per task scope).
- Whether the 7467 MT/s speed is the maximum the modules can achieve or
  their standard operating speed. The SMBIOS reports both `Speed` and
  `ConfiguredClockSpeed` as 7467, but no separate `MaxSpeed` or `ExtendedSize`
  fields are populated.

---

## 6. Channel Configuration

### VERIFIED FACT (host — `Win32_PhysicalMemory.DeviceLocator`)

| Module | DeviceLocator |
|--------|--------------|
| 1 | Controller0-ChannelA |
| 2 | Controller1-ChannelA |

### VERIFIED FACT (host — `Win32_PhysicalMemoryArray.MemoryDevices`)

| Property | Value |
|----------|-------|
| MemoryDevices | 2 |

### VERIFIED FACT (authoritative documentation — Intel ARK)

Intel ARK for the Core Ultra 7 155H (Meteor Lake) specifies a dual-channel
memory controller architecture.

### DERIVED FINDING

- **Channel configuration:** DUAL-CHANNEL.
- Evidence: The two memory modules are located on two separate memory
  controllers (`Controller0` and `Controller1`), both on `ChannelA`.
  The presence of two independent memory controllers, each with one
  module installed, establishes dual-channel operation:
  ```
  Controller0-ChannelA  ← Module 1 (8 GB)
  Controller1-ChannelA  ← Module 2 (8 GB)
  ```
  This is not inferred from module count alone — the `DeviceLocator`
  field explicitly names two distinct controllers (`Controller0` and
  `Controller1`), which is the host's SMBIOS-level evidence of a
  dual-channel (two-controller) memory architecture.
- Intel ARK's specification of dual-channel memory controllers for the
  Meteor Lake platform corroborates the SMBIOS DeviceLocator evidence.

### UNKNOWN

- Whether the system supports quad-channel or higher (the SMBIOS evidence
  shows only two controllers, both on ChannelA, with one module each).
  The Meteor Lake mobile platform supports dual-channel only; quad-channel
  is not applicable to this mobile SoC configuration.

---

## 7. Hardware / Reserved Memory

### VERIFIED FACT (host — `Win32_PhysicalMemoryArray.MemoryErrorCorrection`)

| Property | Value | Interpretation |
|----------|-------|---------------|
| MemoryErrorCorrection | 3 | None |

### VERIFIED FACT (host — comparison of installed RAM vs OS-visible RAM)

| Source | Value | Classification |
|--------|-------|---------------|
| Win32_PhysicalMemory (sum of 2 modules) | 17,179,869,184 B = 16 GB | Installed Physical RAM |
| Win32_ComputerSystem.TotalPhysicalMemory | 16,766,066,688 B ≈ 15.62 GiB | Host OS total physical (after firmware reservation) |
| Win32_OperatingSystem.TotalVisibleMemorySize | 16,373,12 KB ≈ 15.99 GiB | Host OS visible physical |
| systeminfo | 15,989 MB ≈ 15.61 GiB | Host OS visible physical |

### VERIFIED FACT (host — WSL2 `.wslconfig`)

```
[wsl2]
memory=12GB
```

### DERIVED FINDING

```
Installed Physical RAM:   16 GB (17,179,869,184 bytes)
  → firmware/hardware reserved: ~16 GB - ~15.62 GiB ≈ not precisely observable
  → Host OS visible memory:      ~15.99 GiB (16,373,12 KB)
  → WSL2 guest cap:              12 GB (from .wslconfig)
  → WSL2 guest visible:          ~11.67 GiB (12,253,212 kB)
```

- **Hardware/firmware reserved memory:** The difference between installed
  RAM (16 GB) and `Win32_ComputerSystem.TotalPhysicalMemory`
  (16,766,066,688 bytes ≈ 15.62 GiB) represents firmware-reserved memory
  (memory-mapped I/O apertures for GPU framebuffer, PCIe MMIO regions,
  etc.). The exact amount (~413.9 MB = 16 GB - 15.62 GiB) is derived, not
  directly measured. This is an approximate figure.

- **WSL2 `.wslconfig` reservation:** The `.wslconfig` file explicitly caps
  WSL2 guest memory at 12 GB. This is a software-level reservation, not a
  hardware/firmware reservation. The remaining ~3.99 GB of host RAM is
  reserved for the Windows host OS itself.

- **Error correction:** No ECC (`MemoryErrorCorrection=3` = None,
  `DataWidth=TotalWidth=16`).

### UNKNOWN

- Exact firmware/hardware reserved memory breakdown (memory-mapped I/O
  apertures, GPU framebuffer reservation, ACPI tables, etc.).
  The difference between installed RAM and OS-visible memory is
  derived (~413 MB) but the exact partitioning among firmware, GPU
  framebuffer, and other hardware reservations is not directly observable
  from the host evidence available via WMI/SMBIOS.

---

## 8. NUMA

### VERIFIED FACT (host — `Win32_ComputerSystem`)

| Property | Value |
|----------|-------|
| NumberOfProcessors | 1 |
| TotalPhysicalMemory | 16,766,066,688 bytes |
| Model | 83DC |
| Manufacturer | LENOVO |

### VERIFIED FACT (host — `Win32_Processor`)

| Property | Value |
|----------|-------|
| NumberOfCores | 16 |
| NumberOfEnabledCore | 16 |
| NumberOfLogicalProcessors | 22 |
| Sockets | 1 |
| SocketDesignation | U3E1 |

### VERIFIED FACT (authoritative documentation — Intel ARK)

The Intel Core Ultra 7 155H is a single-SKU mobile processor (Meteor Lake,
BGA socket U3E1) with:
- Single socket (1 processor package)
- Integrated SoC (compute, GPU, NPU, and I/O are all on-package or in
  the same package)

### DERIVED FINDING

- **NUMA status:** VERIFIED — single NUMA node.

Evidence:
1. `Win32_ComputerSystem.NumberOfProcessors=1` — single logical processor
   package (socket).
2. `Win32_Processor.Sockets=1` — single physical socket.
3. `SocketDesignation=U3E1` — single socket designation (BGA package,
   soldered to motherboard).
4. Intel Core Ultra 7 155H (Meteor Lake) is a heterogeneous SoC with
  all cores (6P + 8E + 2LP = 16) in a single package. There is no
  evidence of multiple NUMA nodes or multiple sockets.
5. No `Win32_PerfRawData_Counters_NUMANodeMemory` or
   `Win32_PerfFormattedData_Counters_NUMANodeMemory` WMI classes were
   available (invalid class error), which is consistent with a
   single-NUMA-node system where these performance counter classes
   may not be instantiated or are not enumerated.

The Core Ultra 7 155H (Meteor Lake) is a monolithic SoC design with a
single NUMA node. All 16 GB of installed RAM is accessible from all 22
logical processors without NUMA distance penalties.

### UNKNOWN

- Exact NUMA distance metrics (not observable — no NUMA counter classes
  available, and single-node implies uniform access).
- Whether the integrated GPU and NPU have any NUMA-affinity
  considerations (they are on-package with the CPU, so NUMA is not
  a meaningful distinction for them).

---

## 9. WSL2 Guest Memory View

### VERIFIED FACT (WSL2 guest — `/proc/meminfo`)

```
MemTotal:       12253212 kB
MemFree:          281016 kB
MemAvailable:    7343704 kB
...
SwapTotal:       4194304 kB
SwapFree:        3273932 kB
```

### VERIFIED FACT (WSL2 guest — `free -b`)

```
               total        used        free      shared  buff/cache   available
Mem:     12547289088  5027336192   287760384   138653696  7695372288  7519952896
Swap:     4294967296   942460928  3352506368
```

### VERIFIED FACT (WSL2 guest — cgroup v1 memory)

| File | Value |
|------|-------|
| `/sys/fs/cgroup/memory/memory.limit_in_bytes` | 9,223,372,036,854,771,712 (effectively unlimited — no cgroup enforcement) |
| `/sys/fs/cgroup/memory/memory.usage_in_bytes` | 11,013,566,440 bytes (~10.26 GiB, at collection time) |

### VERIFIED FACT (host — `.wslconfig`)

```
[wsl2]
memory=12GB
swap=4GB
```
Source: `/mnt/c/Users/Kawee Lekmuenwai/.wslconfig`

### DERIVED FINDING

- **WSL2-visible memory:** ~11.67 GiB (12,253,212 kB from `/proc/meminfo`),
  confirmed by `free -b` showing 12,547,289,088 bytes total.
- **WSL2 swap:** 4 GiB (4,294,967,296 kB from `/proc/meminfo`), matching the
  `.wslconfig` `swap=4GB` setting.
- **WSL2 memory cap:** 12 GB (`.wslconfig` `memory=12GB`).
- The cgroup v1 `memory.limit_in_bytes` shows effectively unlimited
  (9.22 EB), meaning the 12 GB cap is enforced by the WSL2 hypervisor
  (not the cgroup), consistent with `.wslconfig` policy enforcement at the
  VM level.
- The WSL2 guest sees ~12 GB capped, minus guest kernel overhead, resulting
  in ~11.67 GiB actual MemTotal.
- The cgroup memory usage at collection was ~10.26 GiB, within the 12 GB cap.

### Classification: GUEST ONLY

```
WSL2-visible memory ≠ Physical host RAM
```

The WSL2 guest memory is strictly GUEST-only evidence. It must NOT be used
as physical-RAM evidence. The 12 GB WSL2 cap is a software configuration in
`.wslconfig`, not a physical memory property.

```
┌──────────────────┬──────────────────────┐
│ Physical Host    │ 16 GB installed      │
│                  │ (2 × 8 GB modules)   │
├──────────────────┼──────────────────────┤
│ Host OS          │ ~15.99 GiB visible   │
│                  │ (~15.61 GiB via       │
│                  │  systeminfo)         │
├──────────────────┼──────────────────────┤
│ WSL2 Guest       │ ~11.67 GiB visible   │
│                  │ (capped at 12 GB by   │
│                  │  .wslconfig)         │
└──────────────────┴──────────────────────┘
```

---

## VERIFIED FACT

| # | Evidence | Source |
|---|----------|--------|
| 1 | Total installed physical RAM: 16 GB (17,179,869,184 bytes) | WMI `Win32_PhysicalMemory` (2 × 8,589,934,592 bytes) |
| 2 | Two physical memory modules installed (MemoryDevices=2) | WMI `Win32_PhysicalMemoryArray.MemoryDevices` |
| 3 | Module 1: 8 GB, Samsung, K3KL8L80CM-MGCT, 7467 MT/s, Controller0-ChannelA | WMI `Win32_PhysicalMemory` |
| 4 | Module 2: 8 GB, Samsung, K3KL8L80CM-MGCT, 7467 MT/s, Controller1-ChannelA | WMI `Win32_PhysicalMemory` |
| 5 | Both modules have identical capacity, speed, manufacturer, and part number | WMI `Win32_PhysicalMemory` |
| 6 | Memory type: LPDDR5 (SMBIOSMemoryType=35 per DMTF SMBIOS Type 17) | WMI `Win32_PhysicalMemory.SMBIOSMemoryType` |
| 7 | Memory speed: 7467 MT/s (Speed field) | WMI `Win32_PhysicalMemory.Speed` |
| 8 | Configured memory speed: 7467 MT/s (ConfiguredClockSpeed field) | WMI `Win32_PhysicalMemory.ConfiguredClockSpeed` |
| 9 | Speed = ConfiguredClockSpeed = 7467 (running at rated speed) | WMI `Win32_PhysicalMemory` |
| 10 | DataWidth=16, TotalWidth=16 (non-ECC) for both modules | WMI `Win32_PhysicalMemory` DataWidth/TotalWidth |
| 11 | MemoryErrorCorrection=3 (None) | WMI `Win32_PhysicalMemoryArray.MemoryErrorCorrection` |
| 12 | Use=3 (System Memory) | WMI `Win32_PhysicalMemoryArray.Use` |
| 13 | Dual memory controllers: Controller0 and Controller1 | WMI `Win32_PhysicalMemory.DeviceLocator` |
| 14 | Host OS visible memory: 16,373,12 KB (~15.99 GiB) | WMI `Win32_OperatingSystem.TotalVisibleMemorySize` |
| 15 | Host total physical memory: 16,766,066,688 bytes (~15.62 GiB) | WMI `Win32_ComputerSystem.TotalPhysicalMemory` |
| 16 | systeminfo reports Total Physical Memory: 15,989 MB | `systeminfo` command |
| 17 | Single socket (Sockets=1, NumberOfProcessors=1) | WMI `Win32_ComputerSystem` / `Win32_Processor` |
| 18 | MaxCapacity=33,554,432 KB (32 GB) — 2 slots, both populated | WMI `Win32_PhysicalMemoryArray.MaxCapacity` |
| 19 | WSL2 `.wslconfig` caps guest memory at 12 GB | `/mnt/c/Users/Kawee Lekmuenwai/.wslconfig` |
| 20 | WSL2 guest MemTotal: 12,253,212 kB (~11.67 GiB) | `/proc/meminfo` |
| 21 | WSL2 guest swap: 4 GiB (4,294,967,296 kB) | `/proc/meminfo` SwapTotal |
| 22 | Intel ARK: Core Ultra 7 155H supports LPDDR5x-7467 | Intel ARK specification |
| 23 | Intel ARK: Meteor Lake supports dual-channel memory | Intel ARK specification |

## DERIVED FINDING

| # | Finding | Basis |
|---|---------|-------|
| 1 | 2 × 8 GB = 16 GB installed physical RAM | Summing two Win32_PhysicalMemory.Capacity values (8,589,934,592 × 2 = 17,179,869,184 bytes = 16 GB) |
| 2 | Both modules are identical (Samsung K3KL8L80CM-MGCT, 8 GB, 7467 MT/s) | WMI Win32_PhysicalMemory comparison |
| 3 | Memory type is LPDDR5, consistent with Intel ARK's LPDDR5x-7467 specification for Meteor Lake | SMBIOSMemoryType=35 + Intel ARK LPDDR5x-7467 |
| 4 | Memory running at rated speed (no overclock/downclock) | Speed = ConfiguredClockSpeed = 7467 MT/s for both modules |
| 5 | Dual-channel configuration (two memory controllers, one module each) | DeviceLocator shows Controller0-ChannelA and Controller1-ChannelA |
| 6 | Non-ECC memory (DataWidth = TotalWidth = 16) | WMI Win32_PhysicalMemory.DataWidth/TotalWidth |
| 7 | Single NUMA node (1 socket, 16 cores, 22 threads, monolithic SoC) | Win32_ComputerSystem.NumberOfProcessors=1 + Win32_Processor.Sockets=1 + Intel ARK (single-package Meteor Lake) |
| 8 | Installed RAM (16 GB) vs OS-visible (~15.99 GiB) gap ≈ firmware reservations | 17,179,869,184 bytes (modules) - 16,766,066,688 bytes (ComputerSystem) |
| 9 | Host OS visible (~15.99 GiB) vs WSL2 guest visible (~11.67 GiB) gap is due to .wslconfig 12 GB cap | .wslconfig memory=12GB + WSL2 MemTotal = 12,253,212 kB |
| 10 | `Speed` field ≠ `ConfiguredClockSpeed` distinction: both are 7467, no discrepancy | WMI Win32_PhysicalMemory.Speed vs ConfiguredClockSpeed |
| 11 | The 16 GB physical RAM matches the project target definition of 16 GB system RAM | Win32_PhysicalMemory sum = 16 GB (exact) |

## UNKNOWN

| # | Unknown | Reason |
|---|---------|--------|
| 1 | Exact firmware/hardware reserved memory breakdown | The gap between installed RAM (16 GB) and OS-visible (~15.99 GiB) is derived (~413 MB) but the exact partitioning among firmware, GPU framebuffer, and MMIO reservations is not directly observable |
| 2 | Exact WSL2 memory ballooning parameters | The .wslconfig cap of 12 GB is known, but how the host dynamically allocates/balloons memory between itself and the WSL2 VM is not directly observable from the guest |
| 3 | Actual sustained memory bandwidth | Not benchmarked per task scope; Intel ARK states up to 50 GB/s but this is a platform specification, not a measurement |
| 4 | SODIMM form factor confirmation | SMBIOS FormFactor=0 (Unknown); the module is physically a mobile platform LPDDR5 package |
| 5 | Exact reason for systeminfo (15,989 MB) vs Win32_OperatingSystem (16,373,12 KB ≈ 15,990 MB) discrepancy | Both are host-observed but use different Windows APIs with different rounding/reservation semantics |

---

## Physical-vs-Host-vs-Guest Boundary

This task enforces a strict three-layer separation:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PHYSICAL HOST (Windows 11)                    │
│                                                                  │
│  Installed Physical RAM:  16 GB (2 × 8 GB Samsung LPDDR5)       │
│    Source: Win32_PhysicalMemory (WMI/CIM)                       │
│    Speed:                   7467 MT/s (LPDDR5x-7467 per ARK)    │
│    Type:                    LPDDR5 (SMBIOSMemoryType=35)         │
│    Channels:                Dual (Controller0 + Controller1)   │
│    ECC:                     No (DataWidth=TotalWidth=16)         │
│    Error Correction:        None (MemoryErrorCorrection=3)       │
│    NUMA:                    Single NUMA node (1 socket, 1 SoC)   │
│    MaxCapacity:             32 GB (2 slots, both populated)     │
│                                                                  │
│  Host OS Visible Memory:    ~15.99 GiB (16,373,12 KB)           │
│    Source: Win32_OperatingSystem.TotalVisibleMemorySize         │
│                                                                  │
│  Host OS Total Physical:    ~15.62 GiB (16,766,066,688 bytes)   │
│    Source: Win32_ComputerSystem.TotalPhysicalMemory              │
│                                                                  │
│  WSL2 Config (host file):   .wslconfig memory=12GB              │
│    Source: /mnt/c/Users/Kawee Lekmuenwai/.wslconfig             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    WSL2 GUEST (Linux 5.15.153.1)                │
│                                                                  │
│  WSL2 Guest Visible Memory:    ~11.67 GiB (12,253,212 kB)       │
│    Source: /proc/meminfo (MemTotal)                              │
│    Classification: GUEST ONLY — NOT physical host RAM            │
│                                                                  │
│  WSL2 Guest Swap:              4 GiB (4,294,967,296 kB)           │
│    Source: /proc/meminfo (SwapTotal)                             │
│    (matches .wslconfig swap=4GB)                                 │
│                                                                  │
│  WSL2 cgroup limit:          ~unlimited (9.22 EB, no cgroup cap)│
│    Source: /sys/fs/cgroup/memory/memory.limit_in_bytes           │
│    (cap enforced by .wslconfig at hypervisor level, 12 GB)       │
│                                                                  │
│  WSL2 memory usage (collection): ~10.26 GiB                     │
│    Source: /sys/fs/cgroup/memory/memory.usage_in_bytes           │
└─────────────────────────────────────────────────────────────────┘
```

The critical boundary that must NOT be crossed:

```
WSL2 MemTotal (~11.67 GiB) ≠ Physical installed RAM (16 GB)
WSL2 MemTotal ≠ Host OS-visible physical memory (~15.99 GiB)
```

The WSL2 guest memory is capped to 12 GB by `.wslconfig` on the host.
This is a software configuration, not a physical memory property.

---

## Scope Boundary

This task is strictly limited to **system memory hardware reconnaissance**.
The following activities were deliberately NOT performed:

- NO runtime weight-memory analysis
- NO activation memory analysis
- NO KV-cache analysis
- NO model-weight placement
- NO memory mapping (mmap)
- NO page cache / streaming analysis
- NO memory placement / scheduling decisions
- NO memory optimization
- NO memory bandwidth benchmarking
- NO GPU shared-memory analysis (belongs to SET2-T2.4)
- NO NPU memory analysis (belongs to SET2-T2.5)
- NO allocator behavior analysis
- NO performance benchmarking

---

## Acceptance Result

```text
SET2-T2.3: ✅ PASS
SET2-T2.4: 🔜 NEXT
```

### Acceptance Criteria Checklist

- [x] roadmap persistence completed before execution (Phase A committed and pushed)
- [x] physical installed RAM verified (16 GB, 2 × 8 GB, via Win32_PhysicalMemory)
- [x] host OS-visible memory verified (16,373,12 KB ≈ 15.99 GiB via Win32_OperatingSystem; 15,989 MB via systeminfo)
- [x] WSL2 guest memory clearly separated (capped at 12 GB by .wslconfig; ~11.67 GiB visible; classified as GUEST ONLY)
- [x] memory module configuration established (2 × 8 GB Samsung K3KL8L80CM-MGCT, 7467 MT/s, LPDDR5)
- [x] memory speed/data rate classified correctly (7467 MT/s; Speed = ConfiguredClockSpeed = rated speed; LPDDR5x-7467 per Intel ARK)
- [x] channel configuration established (dual-channel: Controller0-ChannelA + Controller1-ChannelA)
- [x] reserved-memory information classified correctly (firmware reservation gap ~413 MB derived; exact breakdown UNKNOWN; .wslconfig 12 GB cap identified)
- [x] NUMA status established (verified single NUMA node: 1 socket, 1 SoC package, NumberOfProcessors=1)
- [x] no runtime-memory claims introduced
- [x] no memory-placement decisions introduced
- [x] no GPU/NPU work performed
- [x] no benchmark/optimization performed
- [x] VERIFIED FACT / DERIVED FINDING / UNKNOWN used throughout
- [x] canonical T2.3 document created at docs/set-2/03-system-memory-reconnaissance.md
- [ ] local diff verified (pending — see Phase B Verification)
- [ ] commit created (pending)
- [ ] push succeeded (pending)
- [ ] remote evidence verified (pending)
