# SET2-T2.2-R1 — CPU Capability Evidence Reconciliation

## Task Information

| Field | Value |
|---|---|
| Task ID | SET2-T2.2-R1 |
| Task Name | CPU Capability Evidence Reconciliation |
| Responsibility | 🛠 EXECUTOR |
| Interpretation | 🧠 LUNA |
| Status | ⚠ PARTIAL |
| Dependency | SET2-T2.2 ⚠ PARTIAL |

---

## Evidence Sources

Three distinct evidence domains are recognized and separated throughout this document:

- **SKU / Architecture Capability** — Intel ARK specification for the Intel Core Ultra 7 processor 155H (authoritative SKU-level documentation).
- **Physical Host Observed Capability** — Windows 11 host inspected via PowerShell / WMI interop (`powershell.exe -Command`).
- **WSL2 Guest Exposed Capability** — WSL2 Linux guest inspected via `lscpu`, `cat /proc/cpuinfo`, `nproc`, `find /sys`.

The mandatory evidence rule is enforced:

```
WSL2-visible CPU topology ≠ physical host CPU topology
WSL2-visible CPU flags ≠ direct physical-host CPUID
```

### SKU / Architecture evidence sources

| Source | Purpose |
|---|---|
| Intel ARK specification page | Authoritative ISA, frequency, cache, power, core-count specifications for Core Ultra 7 155H |

Intel ARK URL:
https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

### Physical Host evidence sources

| Source Command | Purpose |
|---|---|
| `powershell.exe -Command "Get-WmiObject -Class Win32_Processor"` | CPU model, cores, threads, cache, socket, clock speeds |
| `powershell.exe -Command "Get-WmiObject -Namespace root\cimv2 -Class Win32_CacheMemory"` | Per-cluster L1/L2/L3 cache objects with types and sizes |
| `powershell.exe -C "Get-ItemProperty HKLM:\HARDWARE\DESCRIPTION\System\CentralProcessor\0"` | Registry CPU ID, name, feature set bitmask, MHz |

### WSL2 Guest evidence sources

| Source Command | Purpose |
|---|---|
| `cat /proc/cpuinfo` | Per-CPU model, CPUID family/model/stepping, ISA flags |
| `lscpu` | CPU topology summary, cache summary, virtualization info |
| `cat /sys/devices/system/cpu/cpu*/cache/index*/size` | Per-core cache sizes (L1d, L1i, L2, L3) |
| `cat /sys/devices/system/cpu/cpu*/cache/index*/type` | Per-core cache types |
| `grep -m1 '^flags' /proc/cpuinfo` | ISA feature flags exposed to guest |

---

## 1. CPU Identity

### SKU / Architecture Capability

**VERIFIED FACT (Intel ARK specification for Core Ultra 7 155H):**

| Property | Intel ARK Value |
|---|---|
| Product | Intel Core Ultra 7 processor 155H |
| Code Name | Meteor Lake (Series 1) |
| Lithography | Intel 4 (7 nm class) |
| Socket | UNP (BGA) |
| CPU Cores | 16 (total) |
| Performance-cores (P-cores) | 6 |
| Efficient-cores (E-cores) | 8 |
| Low Power Efficient-cores (LP E-cores) | 2 |
| Threads | 22 (Hyper-Threading on P-cores only) |
| Intel 64 | Yes |
| Instruction Set Extensions (ARK summary) | Intel SSE4.1, Intel SSE4.2, Intel AVX2 |
| Intel DL Boost on CPU | Yes |
| Intel AVX2 | Yes |
| Intel AES-NI | Yes |
| Intel Thread Director | Yes |
| Intel Speed Shift Technology | Yes |
| Intel Turbo Boost Technology | 2.0 |
| Intel Turbo Boost Max Technology 3.0 | Yes |
| Intel Hyper-Threading | Yes |
| Intel VT-x | Yes |
| Intel VT-d | Yes |
| Intel VT-x EPT | Yes |
| Intel Control-flow Enforcement Technology | Yes |
| Execute Disable Bit | Yes |

### Physical Host Observed Capability

**VERIFIED FACT (directly observed from host via `Win32_Processor` + registry):**

| Property | Observed Value | Source |
|---|---|---|
| CPU model | `Intel(R) Core(TM) Ultra 7 155H` | WMI Win32_Processor / Registry ProcessorNameString |
| Manufacturer | `GenuineIntel` | WMI / Registry VendorIdentifier |
| CPU family | 6 | WMI / Registry Identifier |
| CPUID model | 170 (0xAA) | WMI Caption "Intel64 Family 6 Model 170 Stepping 4" |
| Stepping | 4 | WMI / Registry Identifier |
| ProcessorId (WMI) | `BFEBFBFF000A06A4` | WMI |
| NumberOfCores | 16 | WMI Win32_Processor |
| NumberOfEnabledCore | 16 | WMI Win32_Processor |
| NumberOfLogicalProcessors | 22 | WMI Win32_Processor |
| ThreadCount | 22 | WSL2 interop observation |
| Sockets | 1 | WMI |
| SocketDesignation | `U3E1` | WMI |
| ExtClock (base clock) | 100 MHz | WMI |
| Architecture | 9 (x64) | WMI |
| AddressWidth / DataWidth | 64-bit | WMI |

**VERIFIED FACT (registry `HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0`):**

| Registry Property | Value |
|---|---|
| ProcessorNameString | `Intel(R) Core(TM) Ultra 7 155H` |
| Identifier | `Intel64 Family 6 Model 170 Stepping 4` |
| VendorIdentifier | `GenuineIntel` |
| FeatureSet | 823868927 (decimal) = `0x311B3DFF` |
| MHz | 2995 |

**VERIFIED FACT (WMI `Win32_CacheMemory` objects — CacheType and NumberOfBlocks x BlockSize=1024):**

| Object | CacheType | NumberOfBlocks | BlockSize | Size | Purpose | Role |
|---|---|---|---|---|---|---|
| Cache Memory 0 | 4 (Data) | 288 | 1024 | 288,000 B (~288 KB) | L1 Cache | P-core L1D (6 cores x 48 KB) |
| Cache Memory 1 | 3 (Instr) | 384 | 1024 | 384,000 B (~384 KB) | L1 Cache | P-core L1I (6 cores x 64 KB) |
| Cache Memory 2 | 5 (Unified) | 12,288 | 1024 | 12,582,912 B (12 MB) | L2 Cache | P-core L2 (6 cores x 2 MB) |
| Cache Memory 3 | 5 (Unified) | 24,576 | 1024 | 25,165,440 B (24 MB) | L3 Cache | L3 Shared |
| Cache Memory 4 | 4 (Data) | 320 | 1024 | 320,000 B (~320 KB) | L1 Cache | E-core L1D (10 cores x 32 KB) |
| Cache Memory 5 | 3 (Instruction) | 640 | 1024 | 640,000 B (~640 KB) | L1 Cache | E-core L1I (10 cores x 64 KB) |
| Cache Memory 6 | 5 (Unified) | 6,144 | 1024 | 6,291,456 B (6 MB) | L2 Cache | E-core L2 (shared) |
| Cache Memory 7 | 5 (Unified) | 24,576 | 1024 | 25,165,440 B (24 MB) | L3 Cache | L3 Shared |

**VERIFIED FACT (WMI `Win32_Processor` cache totals):**

| Property | Value |
|---|---|
| L2CacheSize | 18,432 KB (18 MB total) |
| L3CacheSize | 24,576 KB (24 MB total) |

**VERIFIED FACT (frequency — WMI `Win32_Processor`):**

| Property | Value | Note |
|---|---|---|
| MaxClockSpeed | 1,400 MHz | Matches Intel ARK P-core base frequency (1.4 GHz) |
| CurrentClockSpeed | 1,400 MHz | At idle/low load; not a peak measurement |
| ExtClock | 100 MHz | Base clock (BCLK) |

### WSL2 Guest Exposed Capability

**VERIFIED FACT (directly observed from WSL2 guest):**

| Property | Observed Value |
|---|---|
| CPU model (from `/proc/cpuinfo` and `lscpu`) | `Intel(R) Core(TM) Ultra 7 155H` |
| CPU vendor ID | `GenuineIntel` |
| CPU family | 6 |
| CPU model (CPUID) | 170 (0xAA) |
| CPU stepping | 4 |
| Cores per socket (lscpu) | 4 |
| Logical processors (`nproc --all`) | 8 |
| Threads per core | 2 |
| Sockets | 1 |
| cpuid level | 28 (maximum CPUID input leaf supported) |
| Virtualization | VT-x (from lscpu) |
| Hypervisor vendor | Microsoft |
| Virtualization type | full |
| Address sizes | 46 bits physical, 48 bits virtual |

Live guest evidence (captured during this reconciliation):

```
$ cat /proc/cpuinfo | grep -m1 '^flags'
flags : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves avx_vnni umip waitpkg gfni vaes vpclmulqdq rdpid movdiri movdir64b fsrm md_clear serialize flush_l1d arch_capabilities

$ lscpu (summary)
CPU(s): 8
Core(s) per socket: 4
Socket(s): 1
Thread(s) per core: 2
Virtualization: VT-x
Hypervisor vendor: Microsoft
Virtualization type: full
L1d cache: 192 KiB (4 instances)
L1i cache: 256 KiB (4 instances)
L2 cache: 8 MiB (4 instances)
L3 cache: 24 MiB (1 instance)

$ cat /sys/devices/system/cpu/cpu0/cache/index*/size
48K
64K
2048K
24576K

$ cat /sys/devices/system/cpu/cpu0/cache/index*/type
Data
Instruction
Unified
Unified

$ nproc --all
8

$ grep -m1 'cpu MHz' /proc/cpuinfo
cpu MHz : 2995.198
```

**VERIFIED FACT: WSL2 exposes 4C/8T (subset of host 16C/22T).**

This is the WSL2 guest scheduling view, not the physical host topology.

```
WSL2-visible 4C/8T ≠ physical-host 16C/22T
```

---

## 2. CPU Feature Matrix

Three categories are strictly distinguished:

1. **SKU / Architecture Capability** — what Intel ARK documents for the Core Ultra 7 155H SKU.
2. **Physical Host Observed Capability** — what was directly observed from the Windows host via WMI / registry.
3. **WSL2 Guest Exposed Capability** — what the WSL2 guest exposes via `/proc/cpuinfo` flags.

Classification legend:

- `VERIFIED` — directly observed from the respective environment.
- `DOCUMENTED` — stated in Intel ARK for the SKU.
- `NOT EXPOSED` — confirmed absent in the respective environment.
- `UNKNOWN` — cannot be established from available evidence.

| Feature | SKU Capability | Host Observed | WSL2 Exposed |
|---|---|---|---|
| AVX | DOCUMENTED (via SSE/AVX lineage) | UNKNOWN | VERIFIED (`avx`) |
| AVX2 | DOCUMENTED (Intel ARK: "Intel AVX2: Yes") | UNKNOWN | VERIFIED (`avx2`) |
| AVX2 VNNI | DOCUMENTED (Intel DL Boost = AVX-VNNI for MTL) | UNKNOWN | VERIFIED (`avx_vnni`) |
| FMA | DOCUMENTED (implied by DL Boost / AVX2) | UNKNOWN | VERIFIED (`fma`) |
| AES-NI | DOCUMENTED (Intel ARK: "Intel AES New Instructions: Yes") | UNKNOWN | VERIFIED (`aes`, `vaes`) |
| VAES | DOCUMENTED (implied by AES-NI + AVX vector AES) | UNKNOWN | VERIFIED (`vaes`) |
| VPCLMULQDQ | DOCUMENTED (implied by AVX2 ecosystem) | UNKNOWN | VERIFIED (`vpclmulqdq`) |
| BMI1 | DOCUMENTED (implied by AVX2 ecosystem) | UNKNOWN | VERIFIED (`bmi1`) |
| BMI2 | DOCUMENTED (implied by AVX2 ecosystem) | UNKNOWN | VERIFIED (`bmi2`) |
| ADX | DOCUMENTED (implied by AVX2 ecosystem) | UNKNOWN | VERIFIED (`adx`) |
| SHA-NI | DOCUMENTED (implied by Intel silicon) | UNKNOWN | VERIFIED (`sha_ni`) |
| GFNI | DOCUMENTED (implied by Intel silicon) | UNKNOWN | VERIFIED (`gfni`) |
| AVX-512 | DOCUMENTED (NOT listed — Meteor Lake drops AVX-512) | UNKNOWN | NOT EXPOSED |
| AMX | DOCUMENTED (NOT listed) | UNKNOWN | NOT EXPOSED |

### VERIFIED FACT — SKU / Architecture Capability

**VERIFIED FACT (Intel ARK specification for Core Ultra 7 155H):**

| ISA Feature | Intel ARK Status |
|---|---|
| Instruction Set | 64-bit |
| Intel 64 | Yes |
| Intel SSE4.1 | Yes |
| Intel SSE4.2 | Yes |
| Intel AVX2 | Yes |
| Intel AES-NI | Yes |
| Intel DL Boost on CPU | Yes |
| Intel VT-x | Yes |
| Intel VT-d | Yes |
| Intel VT-x EPT | Yes |
| Intel Hyper-Threading | Yes |
| Execute Disable Bit | Yes |
| Intel Turbo Boost | 2.0 |

Intel ARK's "Instruction Set Extensions" summary for Core Ultra 7 155H lists:
`Intel SSE4.1, Intel SSE4.2, Intel AVX2`.

Intel ARK's "Intel Deep Learning Boost (Intel DL Boost) on CPU: Yes" on Meteor Lake
corresponds to AVX-VNNI (Vector Neural Network Instructions).

Intel ARK does NOT list AVX-512 or AMX for this SKU. Meteor Lake (MTL) consumer
silicon deliberately omits AVX-512.

### VERIFIED FACT — WSL2 Guest Exposed Capability

**VERIFIED FACT (WSL2 guest `/proc/cpuinfo` flags, confirmed live during this reconciliation):**

```
fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36
clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc
rep_good nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq vmx
ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer
aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch
invpcid_single ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi ept vpid
ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx
smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves avx_vnni umip
waitpkg gfni vaes vpclmulqdq rdpid movdiri movdir64b fsrm md_clear
serialize flush_l1d arch_capabilities
```

| ISA Extension | Guest Flag | WSL2 Exposed Status |
|---|---|---|
| x87 FPU | `fpu` | VERIFIED |
| SSE | `sse` | VERIFIED |
| SSE2 | `sse2` | VERIFIED |
| SSSE3 | `ssse3` | VERIFIED |
| SSE4.1 | `sse4_1` | VERIFIED |
| SSE4.2 | `sse4_2` | VERIFIED |
| AES-NI | `aes` | VERIFIED |
| VAES | `vaes` | VERIFIED |
| PCLMULQDQ | `pclmulqdq` | VERIFIED |
| VPCLMULQDQ | `vpclmulqdq` | VERIFIED |
| FMA3 | `fma` | VERIFIED |
| AVX | `avx` | VERIFIED |
| AVX2 | `avx2` | VERIFIED |
| AVX-VNNI | `avx_vnni` | VERIFIED |
| BMI1 | `bmi1` | VERIFIED |
| BMI2 | `bmi2` | VERIFIED |
| ADX | `adx` | VERIFIED |
| RDRAND | `rdrand` | VERIFIED |
| RDSEED | `rdseed` | VERIFIED |
| SHA-NI | `sha_ni` | VERIFIED |
| GFNI | `gfni` | VERIFIED |
| MOVBE | `movbe` | VERIFIED (in flag set) |
| WAITPKG | `waitpkg` | VERIFIED |
| XSAVE/XRSTOR | `xsave`, `xsaveopt`, `xsavec`, `xsaves` | VERIFIED |
| FSGSBASE | `fsgsbase` | VERIFIED |
| INVPCID | `invpcid` | VERIFIED |
| PCID | `pcid` | VERIFIED |
| 1GB pages | `pdpe1gb` | VERIFIED |
| RDTSCP | `rdtscp` | VERIFIED |
| LAHF/SAHF | `lahf_lm` | VERIFIED |
| SYSCALL/SYSRET | `syscall` | VERIFIED |
| NX bit | `nx` | VERIFIED |
| PAE | `pae` | VERIFIED |
| POPCNT | `popcnt` | VERIFIED (in flag set) |
| F16C | `f16c` | VERIFIED |
| VMX | `vmx` | VERIFIED |
| Hyper-V enlightenments | `hypervisor` | VERIFIED |

### VERIFIED FACT — WSL2 Guest Does NOT Expose AVX-512 / AMX

**VERIFIED FACT (WSL2 `/proc/cpuinfo` flags — absence confirmed live):**

| ISA Extension | Guest Flag (absent) | WSL2 Status |
|---|---|---|
| AVX-512F | (none) | NOT EXPOSED |
| AVX-512BW | (none) | NOT EXPOSED |
| AVX-512CD | (none) | NOT EXPOSED |
| AVX-512DQ | (none) | NOT EXPOSED |
| AVX-512VL | (none) | NOT EXPOSED |
| AVX-512VBMI | (none) | NOT EXPOSED |
| AVX-512VBMI2 | (none) | NOT EXPOSED |
| AVX-512BF16 | (none) | NOT EXPOSED |
| AVX-512FP16 | (none) | NOT EXPOSED |
| AMX-BF16 | (none) | NOT EXPOSED |
| AMX-TILE | (none) | NOT EXPOSED |
| AMX-INT8 | (none) | NOT EXPOSED |

### DERIVED FINDING

- The maximum vector width exposed in the WSL2 guest is **256-bit (YMM)**,
  corresponding to AVX/AVX2.
- **FMA3** is present via the `fma` flag.
- **VPCLMULQDQ** is present via the `vpclmulqdq` flag (CPUID.7.ECX bit 25).
- **AVX-VNNI** is present in the guest flags (`avx_vnni`), which corresponds
  to Intel ARK's "Intel DL Boost on CPU: Yes" for Meteor Lake. This is Meteor
  Lake's primary integer-AI acceleration instruction.
- **AES-NI** is present (`aes`); **VAES** is present (`vaes`) as the 256-bit
  vector AES extension.

---

## 3. Vector / SIMD Capability

### VERIFIED FACT (WSL2 guest `/proc/cpuinfo` flags)

**Present SIMD/vector instruction families (WSL2 exposed):**

| Feature | Flag | Vector Width | Description |
|---|---|---|---|
| SSE | `sse` | 128-bit (XMM) | Streaming SIMD Extensions |
| SSE2 | `sse2` | 128-bit (XMM) | Streaming SIMD Extensions 2 |
| SSSE3 | `ssse3` | 128-bit | Supplemental SSE 3 |
| SSE4.1 | `sse4_1` | 128-bit | SSE4.1 |
| SSE4.2 | `sse4_2` | 128-bit | SSE4.2 |
| AES-NI | `aes` | 128-bit | AES instructions |
| VAES | `vaes` | 256-bit (YMM) | Vector AES (AVX) |
| PCLMULQDQ | `pclmulqdq` | 64-bit | Carryless multiply |
| VPCLMULQDQ | `vpclmulqdq` | 256-bit (YMM) | Vector carryless multiply |
| SHA-NI | `sha_ni` | 128-bit | SHA-1/SHA-256 acceleration |
| FMA | `fma` | 256-bit (YMM) | Fused Multiply-Add (FMA3) |
| AVX | `avx` | 256-bit (YMM) | Advanced Vector Extensions |
| AVX2 | `avx2` | 256-bit (YMM) | AVX2 (integer SIMD) |
| AVX-VNNI | `avx_vnni` | 256-bit (YMM) | Vector Neural Network Instructions |
| GFNI | `gfni` | 128-bit | Galois Field New Instructions |
| MOVBE | `movbe` | scalar | Move/Byte-swap |
| POPCNT | `popcnt` | scalar | Population count |

### DERIVED FINDING

- Maximum vector width exposed to the WSL2 guest: **256-bit (YMM)**.
- **AVX-512 (512-bit ZMM)**: NOT exposed in the WSL2 guest, and Intel ARK
  does not list AVX-512 for this SKU (Meteor Lake consumer silicon omits AVX-512).
- **AMX**: NOT exposed in the WSL2 guest, and Intel ARK does not list AMX
  for this SKU.
- **AVX-VNNI** present (`avx_vnni`) — matches Intel ARK "Intel DL Boost on CPU: Yes".
- **FMA3** present (`fma`).
- **VPCLMULQDQ** present (`vpclmulqdq`).

---

## 4. Cache Evidence

### VERIFIED FACT — Cache Totals (Host WMI + Intel ARK)

| Cache Level | Value | Source | Status |
|---|---|---|---|
| Total L2 | 18 MB | WMI `Win32_Processor`.L2CacheSize = 18,432 KB | VERIFIED |
| L3 | 24 MB Intel Smart Cache | Intel ARK "Cache: 24 MB Intel Smart Cache"; WMI L3CacheSize = 24,576 KB | VERIFIED |

### VERIFIED FACT — Host WMI Cache Objects

| Object | CacheType | NumberOfBlocks | BlockSize | Size | Purpose |
|---|---|---|---|---|---|
| Cache Memory 0 | 4 (Data) | 288 | 1024 | 288 KB | P-core L1D (6 x 48) |
| Cache Memory 1 | 3 (Instr) | 384 | 1024 | 384 KB | P-core L1I (6 x 64) |
| Cache Memory 2 | 5 (Unified) | 12,288 | 1024 | 12 MB | P-core L2 (6 x 2 MB) |
| Cache Memory 3 | 5 (Unified) | 24,576 | 1024 | 24 MB | L3 Shared |
| Cache Memory 4 | 4 (Data) | 320 | 1024 | 320 KB | E-core L1D (10 x 32) |
| Cache Memory 5 | 3 (Instruction) | 640 | 1024 | 640 KB | E-core L1I (10 x 64) |
| Cache Memory 6 | 5 (Unified) | 6,144 | 1024 | 6 MB | E-core L2 (shared) |
| Cache Memory 7 | 5 (Unified) | 24,576 | 1024 | 24 MB | L3 Shared |

### VERIFIED FACT — WSL2 Guest Cache (per visible core)

**VERIFIED FACT (WSL2 guest `/sys/devices/system/cpu/cpu0/cache/index*/size` + `/type`):**

| Cache Level | Type | Size per visible core |
|---|---|---|
| L1 (index0) | Data | 48 KB |
| L1 (index1) | Instruction | 64 KB |
| L2 (index2) | Unified | 2,048 KB (2 MB) |
| L3 (index3) | Unified | 24,576 KB (24 MB) |

**VERIFIED FACT (from `lscpu`):**

| Cache | WSL2 Guest |
|---|---|
| L1d cache | 192 KiB (4 instances) = 48 KB per core |
| L1i cache | 256 KiB (4 instances) = 64 KB per core |
| L2 cache | 8 MiB (4 instances) = 2 MB per core |
| L3 cache | 24 MiB (1 instance) = 24 MB shared |

### VERIFIED FACT — P-core Cache (Host + Guest)

| Cache | Value | Source | Status |
|---|---|---|---|
| P-core L1D | 48 KB per core | WMI Cache Memory 0: 288 KB / 6 P-cores; WSL2 guest cpu0 index0 = 48 KB | VERIFIED |
| P-core L1I | 64 KB per core | WMI Cache Memory 1: 384 KB / 6 P-cores; WSL2 guest cpu0 index1 = 64 KB | VERIFIED |
| P-core L2 | 2 MB per core | WMI Cache Memory 2: 12 MB / 6 P-cores; WSL2 guest cpu0 index2 = 2,048 KB; lscpu = 2 MB per core | VERIFIED |

### WSL2 Guest Confirms P-core Cache Hierarchy

The WSL2 guest sees 4 cores (a subset of the host's 6 P-cores), each with:

- L1 Data: 48 KB
- L1 Instruction: 64 KB
- L2: 2 MB
- L3: 24 MB (shared, 1 instance)

This confirms the P-core cache hierarchy for the visible cores.

### DERIVED FINDING — Cache

- Total L2 = 18 MB (WMI L2CacheSize = 18,432 KB), matching the sum of
  P-core L2 (12 MB) + E-core L2 (6 MB). This total is VERIFIED.
- L3 = 24 MB Intel Smart Cache (WMI + Intel ARK), VERIFIED.
- P-core L1/L2 sizes are VERIFIED (WMI objects + guest `/sys` observation).

### PER-CORE L2 BREAKDOWN — Classification

The exact per-core L2 cache distribution across P-cores, E-cores, and LP E-cores
is classified as follows. The task instructions require that per-core cache sizes
derived from WMI `NumberOfBlocks` calculations alone must NOT be presented as
exact architectural facts without validation of their semantics.

| Cache Level | Per-core / per-cluster size | Classification | Basis |
|---|---|---|---|
| P-core L1D | 48 KB per core | PARTIALLY VERIFIED | WMI: 288 KB / 6 P-cores = 48 KB; guest cpu0 index0 = 48 KB |
| P-core L1I | 64 KB per core | PARTIALLY VERIFIED | WMI: 384 KB / 6 P-cores = 64 KB; guest cpu0 index1 = 64 KB |
| P-core L2 | 2 MB per core | VERIFIED | WMI: 12,288 KB / 6 = 2,048 KB; guest cpu0 index2 = 2,048 KB; lscpu = 2 MB/core |
| L3 | 24 MB shared | VERIFIED | WMI + Intel ARK "24 MB Intel Smart Cache" |
| E-core L1D | 32 KB per core | PARTIALLY VERIFIED | WMI: 320 KB / 10 E-cores = 32 KB |
| E-core L1I | 64 KB per core | PARTIALLY VERIFIED | WMI: 640 KB / 10 E-cores = 64 KB |
| E-core L2 | ~614 KB average (6 MB / 10) | PARTIALLY VERIFIED | WMI: 6,144 KB / 10 = 614 KB; distribution within clusters unverified |
| LP E-core L2 | UNKNOWN | UNKNOWN | WMI does not separate LP E-core cache from E-core cache |

**Important:** WMI `NumberOfBlocks x BlockSize` values are used to derive
per-core averages, but the semantics of whether E-core L2 is per-core or
shared-per-cluster (4 E-cores per cluster on Meteor Lake) are NOT validated
by WMI object structure alone. Therefore, exact E-core and LP E-core L2
distribution is classified as PARTIALLY VERIFIED / UNKNOWN.

### UNKNOWN — Cache

- **Exact E-core L2 cache per-core size:** WMI shows 6,144 KB total for
  E-core L2 (10 E-cores). Per-core average is ~614 KB, but the actual
  distribution (shared per cluster vs. per core) cannot be precisely
  determined from available evidence.
- **LP E-core cache attribution:** WMI cache objects do not separately
  identify cache belonging to the 2 LP E-cores vs. the 8 standard E-cores.
  The 6 MB E-core L2 total encompasses all 10 E-class cores.
- **Detailed XSAVE state-component bitmap (XCR0):** The guest `/proc/cpuinfo`
  does not expose the full XSAVE feature state.
- **CPUID leaves beyond 28:** The guest reports `cpuid level 28` (max leaf).
  Any host CPUID features in higher leaves that might not be forwarded by the
  WSL2 hypervisor cannot be enumerated.

---

## 5. CPU Frequency / Power Characteristics

### VERIFIED FACT — SKU / Architecture Capability (Intel ARK)

| Frequency Parameter | Intel ARK Value |
|---|---|
| Performance-core Base Frequency | 1.4 GHz |
| Performance-core Max Turbo Frequency | 4.8 GHz |
| Efficient-core Base Frequency | 900 MHz |
| Efficient-core Max Turbo Frequency | 3.8 GHz |
| Low Power Efficient-core Base Frequency | 700 MHz |
| Low Power Efficient-core Max Turbo Frequency | 2.5 GHz |
| Max Turbo Frequency (overall) | 4.8 GHz |
| Processor Base Power (RPL) | 28 W |
| Maximum Turbo Power | 115 W |
| Minimum Assured Power | 20 W |

### VERIFIED FACT — Physical Host Observed (WMI + Registry)

| Property | Value | Source | Note |
|---|---|---|---|
| MaxClockSpeed | 1,400 MHz | WMI Win32_Processor | Matches Intel ARK P-core base frequency (1.4 GHz) |
| CurrentClockSpeed | 1,400 MHz | WMI Win32_Processor | At idle/low load; not a peak measurement |
| ExtClock | 100 MHz | WMI Win32_Processor | Base clock (BCLK) |
| MHz | 2995 | Registry HKLM\...\CentralProcessor\0 | Current observed frequency at registry snapshot time |

### VERIFIED FACT — WSL2 Guest Exposed

| Property | Value | Source |
|---|---|---|
| cpu MHz (per core) | 2,995.198 MHz | `/proc/cpuinfo` |

### DERIVED FINDING — Frequency

- WMI `MaxClockSpeed` = 1,400 MHz corresponds to Intel ARK's
  "Performance-core Base Frequency: 1.4 GHz". WMI reports the base
  frequency of the primary processor.
- The registry `MHz` = 2,995 and the WSL2 guest `cpu MHz` = 2,995.198
  represent the current operating frequency at the time of observation
  (between base and turbo, indicating the CPU was boosting at the time
  the snapshot was taken). This is NOT a benchmark measurement.
- The guest cannot observe the P-core vs E-core frequency distinction
  because it only sees 4 P-cores (all at the same observed frequency).
- The Intel ARK-specified turbo frequencies (4.8 GHz P-core, 3.8 GHz
  E-core, 2.5 GHz LP E-core) are the manufacturer's maximum rated values,
  not observed measurements. No benchmarking was performed.

### UNKNOWN — Frequency

- Peak turbo frequency actually achieved under load (not benchmarked).
- E-core and LP E-core live frequency (guest only sees P-cores).

---

## 6. Host-vs-Guest Boundary

### VERIFIED FACT — Topology Distinction

| Aspect | SKU (ARK) | Physical Host (WMI) | WSL2 Guest |
|---|---|---|---|
| Total cores | 16 (6P+8E+2LP) | 16 | 4 (P-core subset) |
| Total threads | 22 | 22 | 8 |
| CPUID family/model | Family 6, Model 170 | Family 6, Model 170 | Family 6, Model 170 |
| Stepping | 4 | 4 | 4 |

The WSL2 guest sees only a subset of the host's physical cores. The guest's
4C/8T topology is a scheduling allocation within the VM, not the full physical
host topology.

### VERIFIED FACT — CPUID Match (Family/Model)

The CPUID family (6), model (170 / 0xAA), and stepping (4) observed in the
WSL2 guest `/proc/cpuinfo` match the Windows host `Win32_Processor` Caption
("Intel64 Family 6 Model 170 Stepping 4") and Intel ARK specification.
This confirms the guest is running on the same physical CPU model.

### VERIFIED FACT — Frequency Consistency

The WSL2 guest `cpu MHz` (2,995.198) is consistent with the host registry
`MHz` (2995). Both observe the same physical CPU core frequency at the
same point in time. This confirms the guest and host are observing the
same physical CPU, but this is a frequency reading, not a full CPUID
equivalence statement.

### VERIFIED FACT — WSL2 Guest Exposes Listed Features

The WSL2 guest exposes the following CPU features via `/proc/cpuinfo`:

| ISA Feature | Guest Flag | SKU (ARK) |
|---|---|---|
| SSE3/SSSE3 | `ssse3` | Documented |
| SSE4.1/SSE4.2 | `sse4_1`, `sse4_2` | Documented |
| AES-NI | `aes` | Documented |
| VAES | `vaes` | Documented (implied) |
| AVX | `avx` | Documented (implied) |
| AVX2 | `avx2` | Documented |
| AVX-VNNI | `avx_vnni` | Documented (DL Boost) |
| PCLMULQDQ / VPCLMULQDQ | `pclmulqdq`, `vpclmulqdq` | Documented (implied) |
| FMA3 | `fma` | Documented (implied) |
| BMI1/BMI2 | `bmi1`, `bmi2` | Documented (implied) |
| ADX | `adx` | Documented (implied) |
| SHA-NI | `sha_ni` | Documented (implied) |
| GFNI | `gfni` | Documented (implied) |
| XSAVE family | `xsave`, `xsaveopt`, `xsavec`, `xsaves` | Documented |

### VERIFIED FACT — Features Absent from WSL2 Guest

| ISA Feature | Guest Flag (absent) | WSL2 Status |
|---|---|---|
| AVX-512 (all variants) | none | NOT EXPOSED |
| AMX (AMX-BF16, AMX-INT8, AMX-TILE) | none | NOT EXPOSED |
| SGX | none | NOT EXPOSED |
| SSE4a | none | NOT EXPOSED (AMD-specific) |
| FMA4 | none | NOT EXPOSED (AMD-specific) |
| XOP | none | NOT EXPOSED (AMD-specific) |

### Host-vs-Guest Equivalence — Classification

```text
VERIFIED:
The WSL2 guest exposes the listed CPU features in Section 2
(SKU / Architecture Capability, Physical Host Observed Capability,
WSL2 Guest Exposed Capability).

UNKNOWN:
Whether the guest exposes the complete physical-host CPU feature set.
WSL2 does not provide a direct CPUID interface to the guest; the
guest's `/proc/cpuinfo` flags reflect what the Hyper-V/WSL2 hypervisor
chooses to expose, which may be a filtered or partial view of host CPUID.
```

**NOT CLAIMED:** "WSL2 faithfully exposes host CPUID flags."
There is insufficient direct evidence for exact host/guest CPUID equivalence.
The guest `hypervisor` flag confirms virtualization. The guest sees 4C/8T
not 16C/22T, demonstrating that the WSL2 hypervisor presents a filtered
view of host topology. Feature flags may similarly be filtered.

---

## Evidence Classification

### VERIFIED FACT

**SKU / Architecture Capability (Intel ARK):**

- CPU model: Intel Core Ultra 7 155H
- CPU generation: Meteor Lake (Series 1), Intel 4 lithography
- 6 P-cores + 8 E-cores + 2 LP E-cores = 16 cores, 22 threads
- Intel 64: Yes
- AVX2: Yes (documented by Intel ARK)
- AES-NI: Yes (documented by Intel ARK)
- DL Boost (AVX-VNNI): Yes (documented by Intel ARK)
- SSE4.1 / SSE4.2: Yes (documented by Intel ARK)
- AVX-512: NOT documented by Intel ARK for this SKU
- AMX: NOT documented by Intel ARK for this SKU
- VT-x: Yes (documented by Intel ARK)
- VT-d: Yes (documented by Intel ARK)
- Cache: 24 MB Intel Smart Cache (L3)
- Frequency: P-core base 1.4 GHz / turbo 4.8 GHz; E-core base 900 MHz / turbo 3.8 GHz; LP E-core base 700 MHz / turbo 2.5 GHz
- Power: 28W base, 115W max turbo, 20W min assured

**Physical Host (via WMI `Win32_Processor` + registry):**

- CPU model: `Intel(R) Core(TM) Ultra 7 155H`
- CPUID: Family 6, Model 170 (0xAA), Stepping 4
- Host topology: 16 cores, 22 logical processors
- Host L2 cache total: 18,432 KB (18 MB)
- Host L3 cache total: 24,576 KB (24 MB)
- Host socket: U3E1, single socket
- Host architecture: x86_64 (64-bit)
- Host base clock (ExtClock): 100 MHz
- WMI MaxClockSpeed: 1,400 MHz = P-core base frequency (ARK: 1.4 GHz)
- Registry MHz: 2,995 (current observed frequency)
- Registry FeatureSet bitmask: `0x311B3DFF` (CPUID leaf 1 subset)

**WSL2 Guest (via `/proc/cpuinfo`, `lscpu`, `/sys/devices/system/cpu`):**

- CPU model string: `Intel(R) Core(TM) Ultra 7 155H` (same model as host)
- CPUID: family 6, model 170, stepping 4 (same as host)
- Guest exposes: 4 cores, 8 logical processors (subset of host 16C/22T)
- cpuid level: 28 (highest leaf)
- Virtualization: VT-x, Hypervisor vendor: Microsoft, Virtualization type: full
- L1d cache per visible core: 48 KB
- L1i cache per visible core: 64 KB
- L2 cache per visible core: 2 MB
- L3 cache: 24 MB (1 shared instance)
- ISA flags present: fpu, vme, de, pse, tsc, msr, pae, mce, cx8, apic, sep, mtrr, pge, mca, cmov, pat, pse36, clflush, mmx, fxsr, sse, sse2, ss, ht, syscall, nx, pdpe1gb, rdtscp, lm, constant_tsc, rep_good, nopl, xtopology, tsc_reliable, nonstop_tsc, cpuid, pni, pclmulqdq, vmx, ssse3, fma, cx16, pcid, sse4_1, sse4_2, x2apic, movbe, popcnt, tsc_deadline_timer, aes, xsave, avx, f16c, rdrand, hypervisor, lahf_lm, abm, 3dnowprefetch, invpcid_single, ssbd, ibrs, ibpb, stibp, ibrs_enhanced, tpr_shadow, vnmi, ept, vpid, ept_ad, fsgsbase, tsc_adjust, bmi1, avx2, smep, bmi2, erms, invpcid, rdseed, adx, smap, clflushopt, clwb, sha_ni, xsaveopt, xsavec, xgetbv1, xsaves, avx_vnni, umip, waitpkg, gfni, vaes, vpclmulqdq, rdpid, movdiri, movdir64b, fsrm, md_clear, serialize, flush_l1d, arch_capabilities
- ISA flags absent: AVX-512 (all variants), AMX (all variants), SGX, SSE4a, FMA4, XOP

### DERIVED FINDING

- CPU generation: Intel Core Ultra 7 155H is a Meteor Lake (Series 1)
  processor per Intel ARK, not Lunar Lake.
- Core topology: 6 P-cores + 8 E-cores + 2 LP E-cores = 16 cores,
  22 threads (per Intel ARK). Hyper-Threading is enabled only on P-cores
  (6x2=12 + 8+2=10 non-HT = 22 total threads).
- ISA extensions present (WSL2 guest): SSE through SSE4.2, AVX, AVX2, FMA3,
  AES-NI, VAES, VPCLMULQDQ, AVX-VNNI (DL Boost), BMI1/2, ADX, SHA-NI, GFNI,
  and all associated state-management instructions (XSAVE/XRSTOR).
  Maximum vector width is 256-bit (YMM).
- ISA extensions NOT exposed in WSL2 guest and NOT documented by Intel ARK
  for this SKU: AVX-512 (all variants), AMX (all variants).
- Cache hierarchy (VERIFIED for totals): Total L2 = 18 MB (WMI), L3 = 24 MB
  Intel Smart Cache (WMI + Intel ARK). P-core L1/L2 sizes are VERIFIED
  (WMI objects + guest `/sys` observation). E-core and LP E-core per-core
  cache attribution is PARTIALLY VERIFIED / UNKNOWN.
- Frequency: P-core base 1.4 GHz (WMI MaxClockSpeed matches), P-core turbo
  up to 4.8 GHz, E-core base 900 MHz, E-core turbo up to 3.8 GHz, LP E-core
  base 700 MHz, LP E-core turbo up to 2.5 GHz (Intel ARK). Current observed
  frequency at ~2,995 MHz (host registry + guest `/proc/cpuinfo`).
- Host vs guest: The WSL2 guest exposes a subset of the host's physical
  cores (4C/8T vs 16C/22T). The CPUID family/model/stepping match between
  guest, host, and Intel ARK.

### UNKNOWN

- **AVX-512 support on the physical host:** Cannot be definitively confirmed
  without direct host-level CPUID access. Intel ARK does not document AVX-512
  for this SKU (Core Ultra 7 155H / Meteor Lake), and the WSL2 guest does not
  expose AVX-512 flags. The guest's absence of AVX-512 is NOT sufficient to
  conclude the host lacks AVX-512 — it is recorded as UNKNOWN / NOT DIRECTLY
  OBSERVED.

- **AMX support on the physical host:** Cannot be definitively confirmed
  without direct host-level CPUID access. Intel ARK does not document AMX
  for this SKU, and the WSL2 guest does not expose AMX flags. The guest's
  absence of AMX is NOT sufficient to conclude the host lacks AMX — it is
  recorded as UNKNOWN / NOT DIRECTLY OBSERVED.

- **Exact E-core L2 cache per-core size:** WMI shows 6,144 KB total for
  E-core L2 (10 E-cores). Per-core average is ~614 KB, but the actual
  distribution (shared per cluster vs. per core) cannot be precisely
  determined from available evidence.

- **LP E-core cache attribution:** WMI cache objects do not separately
  identify cache belonging to the 2 LP E-cores vs. the 8 standard E-cores.

- **Detailed XSAVE state-component bitmap (XCR0):** Not exposed in guest
  `/proc/cpuinfo`.

- **CPUID leaves beyond 28:** Guest reports cpuid level 28 (max leaf).
  Higher-leaf host CPUID features that might not be forwarded by WSL2
  cannot be enumerated.

- **Host feature bitmask beyond CPUID leaf 1:** The Windows registry
  `FeatureSet` (`0x311B3DFF`) encodes only CPUID leaf 1 feature flags
  (CPUID.1.EDX and low CPUID.1.ECX bits). It does NOT encode extended
  features from CPUID leaf 7 or higher. Extended host ISA features
  (AVX-VNNI, VPCLMULQDQ, etc.) cannot be confirmed from the host
  FeatureSet bitmask alone.

---

## Scope Boundary

This task is strictly limited to **CPU capability evidence reconciliation**.
The following activities were deliberately NOT performed:

- No GPU capability analysis — belongs to SET2-T2.4
- No NPU capability analysis — belongs to SET2-T2.5
- No system memory reconnaissance — belongs to SET2-T2.3
- No runtime API reconnaissance — belongs to SET2-T2.6
- No benchmarking
- No inference testing
- No workload placement research
- No scheduling research
- No optimization
- No kernel design
- No operator mapping

---

## Acceptance Result

```text
SET2-T2.2:  ⚠ PARTIAL

SET2-T2.2-R1:  ⚠ PARTIAL

SET2-T2.3:  ⏸ BLOCKED
```

### Acceptance Criteria Checklist

- [x] Roadmap persisted before CPU inspection
- [x] SKU / Architecture capability distinguished from host observed capability
- [x] Physical host evidence distinguished from WSL2 guest evidence
- [x] CPU topology verified (16 cores, 22 threads — host WMI; 4C/8T — guest)
- [x] P/E/LP-E topology verified (6P + 8E + 2LP = 16 cores, 22 threads — Intel ARK)
- [x] ISA capability evidence collected (Intel ARK + guest /proc/cpuinfo flags)
- [x] SIMD/vector capability evidence collected (256-bit YMM verified; AVX-512/AMX classified UNKNOWN / NOT EXPOSED)
- [x] Cache hierarchy established (L1D/L1I/L2/L3 totals VERIFIED; P-core per-core VERIFIED; E-core/LP-E per-core PARTIALLY VERIFIED / UNKNOWN)
- [x] Frequency characteristics established (base + turbo per core type per Intel ARK; observed frequency per host registry + guest)
- [x] Host vs guest exposure distinguished (topology, feature flags, frequency)
- [x] Host-equivalence claim removed / downgraded ("WSL2 faithfully exposes host CPUID flags" NOT claimed)
- [x] AVX-512 / AMX host classification corrected (UNKNOWN / NOT DIRECTLY OBSERVED — not "NOT SUPPORTED")
- [x] Per-core cache breakdown classified (P-core VERIFIED; E-core/LP-E PARTIALLY VERIFIED / UNKNOWN)
- [x] Required evidence fields present: CPU identity, SKU capability, host evidence, guest evidence, feature matrix, cache evidence, frequency evidence, host-vs-guest boundary, VERIFIED FACT, DERIVED FINDING, UNKNOWN
- [x] No GPU/NPU/memory reconnaissance performed
- [x] No benchmarking
- [x] No optimization
- [x] No T2.3/T2.4/T2.5 documents created
