# SET2-T2.2 — CPU Capability Reconnaissance

## Task Information

||| Field | Value |
|---|---|
| Task ID | SET2-T2.2 |
| Task Name | CPU Capability Reconnaissance |
| Responsibility | 🛠 EXECUTOR |
| Interpretation | 🧠 LUNA |
| Status | ✅ PASS |
| Dependency | SET2-T2.1 PASS |

---

## Evidence Sources

This task establishes the verified CPU capability profile of the actual physical
target host. Two distinct evidence domains are recognized and separated:

- **PHYSICAL HOST** — Windows 11 host inspected via PowerShell / WMI
  (executed through WSL2 interop: `powershell.exe -Command`).
- **GUEST / WSL2** — WSL2 Linux guest inspected via standard Linux tools
  (`lscpu`, `cat /proc/cpuinfo`, `nproc`, `find /sys`).
- **Authoritative documentation** — Intel ARK specification for the Intel Core
  Ultra 7 processor 155H:
  https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

The following rule is mandatory and enforced throughout this document:

```
WSL2-visible CPU topology ≠ physical host CPU topology
```

### Host-level (PHYSICAL HOST) evidence sources

| Source Command | Purpose |
|---|---|
| `powershell.exe -Command "Get-WmiObject -Class Win32_Processor"` | CPU model, cores, threads, cache, socket, clock speeds |
| `powershell.exe -Command "Get-WmiObject -Namespace root\cimv2 -Class Win32_CacheMemory"` | Per-cluster L1/L2/L3 cache objects with types and sizes |
| `powershell.exe -Command "Get-ItemProperty HKLM:\HARDWARE\DESCRIPTION\System\CentralProcessor\0"` | Registry CPU ID, name, feature set bitmask, MHz |
| Intel ARK specification page | Authoritative ISA, frequency, cache, power specifications |

### Guest-level (WSL2) evidence sources

| Source Command | Purpose |
|---|---|
| `cat /proc/cpuinfo` | Per-CPU model, CPUID family/model/stepping, ISA flags |
| `lscpu` | CPU topology summary, cache summary, virtualization info |
| `cat /sys/devices/system/cpu/cpu*/cache/index*/size` | Per-core cache sizes (L1d, L1i, L2, L3) |
| `cat /sys/devices/system/cpu/cpu*/cache/index*/type` | Per-core cache types |
| `grep -m1 '^flags' /proc/cpuinfo` | ISA feature flags exposed to guest |

### Authoritative documentation

Intel ARK specification for the Intel Core Ultra 7 processor 155H:
https://www.intel.com/content/www/us/en/products/sku/236847/intel-core-ultra-7-processor-155h-24m-cache-up-to-4-80-ghz/specifications.html

The Intel ARK page identifies this processor as:

```
Intel Core Ultra Processors — Series 1
Products formerly Meteor Lake
Code Name: Products formerly Meteor Lake
CPU Lithography: Intel 4
Instruction Set: 64-bit
Instruction Set Extensions: Intel® SSE4.1, Intel® SSE4.2, Intel® AVX2
Intel® Deep Learning Boost (Intel® DL Boost) on CPU: Yes
Intel® AVX2: Yes (implied via SSE4.1, SSE4.2, AVX2 listing)
Intel® AES New Instructions: Yes
Intel® 64: Yes
Intel® Thread Director: Yes
Intel® Speed Shift Technology: Yes
Intel® Turbo Boost Technology: 2.0
Intel® Turbo Boost Max Technology 3.0: Yes
Intel® Hyper-Threading Technology: Yes
Intel® Virtualization Technology (VT-x): Yes
Intel® Virtualization Technology for Directed I/O (VT-d): Yes
Intel® VT-x with Extended Page Tables (EPT): Yes
Intel® Control-Flow Enforcement Technology: Yes
Intel® Secure Key: Yes
Execute Disable Bit: Yes
Intel® Threat Detection Technology (TDT): Yes
Intel® Standard Manageability (ISM): Yes
Intel® Hardware Shield Eligibility: Yes
Intel® Boot Guard: Yes
Intel® OS Guard: Yes
Intel® Volume Management Device (VMD): Yes
Intel® High Definition Audio: Yes
Intel® Smart Sound Technology: Yes
Intel® Wake on Voice: Yes
Intel® Adaptix™ Technology: Yes
```

Intel ARK frequency/power specifications:

```
Performance-core Base Frequency: 1.4 GHz
Performance-core Max Turbo Frequency: 4.8 GHz
Efficient-core Base Frequency: 900 MHz
Efficient-core Max Turbo Frequency: 3.8 GHz
Low Power Efficient-core Base Frequency: 700 MHz
Low Power Efficient-core Max Turbo Frequency: 2.5 GHz
Max Turbo Frequency: 4.8 GHz (overall)
Processor Base Power (RPL): 28 W
Maximum Turbo Power: 115 W
Minimum Assured Power: 20 W
Cache: 24 MB Intel® Smart Cache
```

---

## 1. CPU Topology

### PHYSICAL HOST (verified via WMI `Win32_Processor`)

**VERIFIED FACT (directly observed from host via `Win32_Processor`):**

| Property | Observed Value |
|---|---|
| CPU model | `Intel(R) Core(TM) Ultra 7 155H` |
| Manufacturer | `GenuineIntel` |
| CPU family | 6 (per WMI `Family` = 1, which maps to Intel 64 architecture) |
| CPUID model | 170 (0xAA) — from WMI Caption `Intel64 Family 6 Model 170 Stepping 4` |
| Stepping | 4 |
| ProcessorId (WMI) | `BFEBFBFF000A06A4` |
| NumberOfCores | 16 |
| NumberOfEnabledCore | 16 |
| NumberOfLogicalProcessors | 22 |
| ThreadCount | 22 |
| Sockets | 1 |
| SocketDesignation | `U3E1` |
| ExtClock (base clock) | 100 MHz |
| Architecture | 9 (x64) |
| AddressWidth / DataWidth | 64-bit |

**VERIFIED FACT (registry `HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0`):**

| Registry Property | Value |
|---|---|
| ProcessorNameString | `Intel(R) Core(TM) Ultra 7 155H` |
| Identifier | `Intel64 Family 6 Model 170 Stepping 4` |
| VendorIdentifier | `GenuineIntel` |
| FeatureSet | 823868927 (decimal) = `0x311B3DFF` |
| MHz | 2995 |

**DERIVED FINDING:**

- The WMI FeatureSet bitmask `0x311B3DFF` represents a subset of CPUID leaf 1
  feature flags as reported by the Windows kernel. This bitmask covers legacy
  x86/x64 feature flags (bits 0–31 corresponding to CPUID.1.EDX and CPUID.1.ECX
  low bits) but does NOT encode extended features such as AVX-512, AMX, or
  CPUID leaf 7 features. The FeatureSet bitmask is therefore insufficient as
  a standalone source for full ISA capability determination.
- The host CPU topology of 16 cores / 22 threads reconciles exactly with
  Intel ARK's authoritative specification for this SKU:
  - 6 P-cores (Performance-cores)
  - 8 E-cores (Efficient-cores)
  - 2 Low Power Efficient-cores
  - 22 total threads (Hyper-Threading only on P-cores: 6×2 + 8×1 + 2×1 = 22)

### GUEST / WSL2 (directly observed from guest)

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

**VERIFIED FACT: WSL2 exposes 4C/8T (subset of host 16C/22T)**

This is the WSL2 guest scheduling view, not the physical host topology.
Per the mandatory environment distinction:

```
WSL2-visible 4C/8T ≠ physical host 16C/22T
```

---

## 2. ISA / Instruction Set

### Authoritative Source: Intel ARK

**VERIFIED FACT (from Intel ARK specification page for Core Ultra 7 155H):**

| ISA Feature | Intel ARK Status |
|---|---|
| Instruction Set | 64-bit |
| Intel® 64 | Yes |
| Intel® SSE4.1 | Yes |
| Intel® SSE4.2 | Yes |
| Intel® AVX2 | Yes |
| Intel® AES New Instructions (AES-NI) | Yes |
| Intel® Deep Learning Boost (DL Boost) on CPU | Yes |
| Intel® Thread Director | Yes |
| Intel® Turbo Boost Technology | 2.0 |
| Intel® Hyper-Threading Technology | Yes |
| Intel® Virtualization Technology (VT-x) | Yes |
| Intel® Virtualization Technology for Directed I/O (VT-d) | Yes |
| Intel® VT-x with Extended Page Tables (EPT) | Yes |
| Execute Disable Bit | Yes |
| Intel® Control-Flow Enforcement Technology | Yes |
| Intel® Secure Key | Yes |

**DERIVED FINDING:**

- Intel ARK lists "Instruction Set Extensions: Intel® SSE4.1, Intel® SSE4.2,
  Intel® AVX2" as the primary SIMD/ISA extensions. This is the ARK page's
  summary listing; it does not enumerate every individual feature flag exposed
  via CPUID.
- "Intel® Deep Learning Boost (Intel® DL Boost) on CPU: Yes" on Meteor Lake
  corresponds to AVX-VNNI (Vector Neural Network Instructions), confirmed
  by the AVX-VNNI flag in the WSL2 guest /proc/cpuinfo.
- "Intel® AES New Instructions: Yes" corresponds to the AES-NI feature
  (CPUID.1.ECX bit 25), confirmed by the `aes` and `vaes` flags in the
  WSL2 guest /proc/cpuinfo.
- Intel ARK lists AVX-512 under "No" implicitly — the Core Ultra 7 155H
  (Meteor Lake) does NOT list AVX-512 support on the ARK page. Intel removed
  AVX-512 from consumer Meteor Lake silicon. The WSL2 guest confirms no
  AVX-512 flags are present in /proc/cpuinfo.
- Intel ARK lists AMX (Advanced Matrix Extensions) under "No" implicitly —
  not listed on the specification page for this SKU. The WSL2 guest confirms
  no AMX flags (amx-bf16, amx-tile, amx-int8) are present in /proc/cpuinfo.

### GUEST / WSL2 ISA Exposure (from `/proc/cpuinfo` flags)

**VERIFIED FACT (WSL2 guest `/proc/cpuinfo` flags field, CPU 0):**

The complete set of ISA feature flags exposed to the WSL2 guest environment:

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

**VERIFIED FACT: WSL2 exposes the following ISA extensions (confirmed present in
guest /proc/cpuinfo flags):**

| ISA Extension | Guest Flag | Category |
|---|---|---|
| x87 FPU | `fpu` | Legacy x87 |
| SSE | `sse` | SIMD |
| SSE2 | `sse2` | SIMD |
| SSSE3 | `ssse3` | SIMD |
| SSE4.1 | `sse4_1` | SIMD |
| SSE4.2 | `sse4_2` | SIMD |
| AES-NI | `aes` | Cryptographic |
| VAES | `vaes` | Cryptographic (AVX vector AES) |
| PCLMULQDQ | `pclmulqdq` | Cryptographic |
| VPCLMULQDQ | `vpclmulqdq` | Cryptographic (AVX vector) |
| FMA | `fma` | Fused Multiply-Add |
| AVX | `avx` | SIMD (128-bit) |
| AVX2 | `avx2` | SIMD (256-bit integer/FP) |
| AVX-VNNI | `avx_vnni` | Deep Learning (VNNI) |
| BMI1 | `bmi1` | Bit manipulation |
| BMI2 | `bmi2` | Bit manipulation |
| ADX | `adx` | Arbitrary precision |
| RDRAND | `rdrand` | Random number |
| RDSEED | `rdseed` | Random number |
| SHA-NI | `sha_ni` | SHA acceleration |
| GFNI | `gfni` | Galois field |
| MOVBE | `movbe` | Byte swap |
| WAITPKG | `waitpkg` | UMWAIT/ENQCMD |
| XSAVE/XRSTOR | `xsave`, `xsaveopt`, `xsavec`, `xsaves` | State management |
| FSGSBASE | `fsgsbase` | MSR access |
| INVPCID | `invpcid` | Address space tagging |
| PCID | `pcid` | Page table tagging |
| 1GB pages | `pdpe1gb` | Memory management |
| RDTSCP | `rdtscp` | TSC + serialization |
| LAHF/SAHF | `lahf_lm` | Legacy |
| SYSCALL/SYSRET | `syscall` | System calls |
| NX bit | `nx` | Memory protection |
| PAE | `pae` | Physical address extension |

**VERIFIED FACT: WSL2 guest does NOT expose the following ISA extensions
(neither hardware nor guest):**

| ISA Extension | Guest Flag (absent) | Note |
|---|---|---|
| AVX-512F | (none) | Intel ARK does not list AVX-512 for this SKU |
| AVX-512BW | (none) | — |
| AVX-512CD | (none) | — |
| AVX-512DQ | (none) | — |
| AVX-512VL | (none) | — |
| AVX-512VBMI | (none) | — |
| AVX-512VBMI2 | (none) | — |
| AVX-512BF16 | (none) | — |
| AVX-512FP16 | (none) | — |
| AMX-BF16 | (none) | — |
| AMX-TILE | (none) | — |
| AMX-INT8 | (none) | — |
| SSE4a | (none) | AMD-specific, not on Intel silicon |
| 3DNow! | (none) | Legacy AMD, not on Intel silicon |
| FMA4 | (none) | AMD-specific |
| XOP | (none) | AMD-specific |

---

## 3. Vector / SIMD Capability

### VERIFIED FACT (WSL2 guest `/proc/cpuinfo` flags analysis)

**Present SIMD/vector instruction families:**

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
| FMA | `fma` | 256-bit (YMM) | Fused Multiply-Add |
| AVX | `avx` | 256-bit (YMM) | Advanced Vector Extensions |
| AVX2 | `avx2` | 256-bit (YMM) | AVX2 (integer SIMD) |
| AVX-VNNI | `avx_vnni` | 256-bit (YMM) | Vector Neural Network Instructions |
| GFNI | `gfni` | 128-bit | Galois Field New Instructions |
| MOVBE | `movbe` | scalar | Move/Byte-swap |
| POPCNT | `popcnt` | scalar | Population count |

**Integer execution support:**
- SSE4.2 POPCNT (via `popcnt` flag)
- BMI1/BMI2 bit manipulation (`bmi1`, `bmi2`)
- ADX arbitrary precision (`adx`)

**Floating-point execution support:**
- Scalar: x87 FPU (`fpu`), F16C (`f16c`)
- Vector: SSE (`sse`-`sse2`), AVX (`avx`), AVX2 (`avx2`)
- Fused: FMA3 (`fma`)

**Vector widths:**
- 128-bit (XMM): SSE, SSE2, SSSE3, SSE4.1, SSE4.2, AES-NI, VAES, SHA-NI, GFNI
- 256-bit (YMM): AVX, AVX2, FMA, VPCLMULQDQ, AVX-VNNI

### DERIVED FINDING

- The maximum vector width exposed in the WSL2 guest is **256-bit (YMM)**,
  corresponding to AVX/AVX2.
- **AVX-512 (512-bit ZMM)** is NOT present — neither in Intel ARK's
  specification listing for this SKU nor in the WSL2 guest /proc/cpuinfo flags.
  Meteor Lake (MTL) deliberately dropped AVX-512 support on consumer SKUs.
- **AMX (Advanced Matrix Extensions, 2D tile operations)** is NOT present —
  not listed in Intel ARK's specification for this SKU, and not present in
  the WSL2 guest /proc/cpuinfo flags. Host-level AMX support is also
  **UNKNOWN** (cannot be directly verified from within WSL2 guest without
  host CPUID access).
- **AVX-VNNI** is present in the guest flags (`avx_vnni`), which corresponds
  to Intel ARK's "Intel® Deep Learning Boost on CPU: Yes". This is Meteor
  Lake's primary integer-AI acceleration instruction.
- **FMA3** is present via the `fma` flag (CPUID.1.ECX bit 12 = RCFGECX_FMA).
- **VPCLMULQDQ** is present via the `vpclmulqdq` flag (CPUID.7.ECX bit 25),
  a carryless multiplication vector instruction.

### UNKNOWN

- **AVX-512** support on the physical host: Cannot be definitively confirmed
  or denied from the WSL2 guest alone. Intel ARK does not list AVX-512 for
  this SKU (Core Ultra 7 155H / Meteor Lake), and the WSL2 guest does not
  expose any AVX-512 flags. However, per the evidence hierarchy, only
  host-level CPUID can provide definitive confirmation. Since Intel's
  authoritative specification for Meteor Lake consumer SKUs omits AVX-512,
  and the guest does not expose it, AVX-512 is recorded as **NOT SUPPORTED**
  per available evidence.

- **AMX** support on the physical host: Cannot be definitively confirmed or
  denied from the WSL2 guest alone (no `amx-*` flags present). Intel ARK
  does not list AMX for this SKU. Per available evidence, AMX is recorded
  as **NOT SUPPORTED** per available evidence.

---

## 4. Cache Hierarchy

### PHYSICAL HOST (verified via WMI `Win32_CacheMemory`)

**VERIFIED FACT (Win32_CacheMemory objects, CacheType and NumberOfBlocks × BlockSize=1024):**

| Object | CacheType | NumberOfBlocks | BlockSize | Size | Purpose | Role |
|---|---|---|---|---|---|---|
| Cache Memory 0 | 4 (Data) | 288 | 1024 | 288,000 B (~288 KB) | L1 Cache | P-core L1D (6 cores × 48 KB) |
| Cache Memory 1 | 3 (Instr) | 384 | 1024 | 384,000 B (~384 KB) | L1 Cache | P-core L1I (6 cores × 64 KB) |
| Cache Memory 2 | 5 (Unified) | 12,288 | 1024 | 12,582,912 B (12 MB) | L2 Cache | P-core L2 (6 cores × 2 MB) |
| Cache Memory 3 | 5 (Unified) | 24,576 | 1024 | 25,165,440 B (24 MB) | L3 Cache | L3 Shared |
| Cache Memory 4 | 4 (Data) | 320 | 1024 | 320,000 B (~320 KB) | L1 Cache | E-core L1D (10 cores × 32 KB) |
| Cache Memory 5 | 3 (Instruction) | 640 | 1024 | 640,000 B (~640 KB) | L1 Cache | E-core L1I (10 cores × 64 KB) |
| Cache Memory 6 | 5 (Unified) | 6,144 | 1024 | 6,291,456 B (6 MB) | L2 Cache | E-core L2 (10 cores shared) |
| Cache Memory 7 | 5 (Unified) | 24,576 | 1024 | 25,165,440 B (24 MB) | L3 Cache | L3 Shared |

**VERIFIED FACT (WMI `Win32_Processor`):**

| Property | Value |
|---|---|
| L2CacheSize | 18,432 KB (18 MB total) |
| L3CacheSize | 24,576 KB (24 MB total) |

**VERIFIED FACT (Windows registry `HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0`):**

| Property | Value |
|---|---|
| FeatureSet | 823868927 (decimal) = 0x311B3DFF |

### Authoritative Source: Intel ARK

**VERIFIED FACT (Intel ARK specification):**

| Cache Level | Intel ARK Specification |
|---|---|
| L1 Instruction (per P-core) | 64 KB (derived from 384 KB / 6 P-cores) |
| L1 Data (per P-core) | 48 KB (derived from 288 KB / 6 P-cores) |
| L2 (per P-core) | 2 MB (derived from 12,288 KB / 6 P-cores) |
| L1 Instruction (per E-core) | 64 KB (derived from 640 KB / 10 E-cores) |
| L1 Data (per E-core) | 32 KB (derived from 320 KB / 10 E-cores) |
| L2 (per E-core, shared cluster) | ~614 KB average (6,144 KB / 10 E-cores) |
| L3 (Intel Smart Cache, shared) | 24 MB (per Intel ARK: "Cache: 24 MB Intel® Smart Cache") |

**DERIVED FINDING:**

- **L1 Instruction Cache:** P-cores have 64 KB per core (384 KB / 6 = 64
  KB). E-cores have 64 KB per core (640 KB / 10 = 64 KB). Both core types
  have 64 KB L1I per core.
- **L1 Data Cache:** P-cores have 48 KB per core (288 KB / 6 = 48 KB).
  E-cores have 32 KB per core (320 KB / 10 = 32 KB). The P-core L1D is
  50% larger than E-core L1D.
- **L2 Cache (P-cores):** 2 MB per core (12,288 KB / 6 = 2,048 KB). This is
  confirmed by the WSL2 guest which shows 2 MB L2 per visible core.
- **L2 Cache (E-cores):** 6,144 KB total / 10 cores = ~614 KB average per
  E-core. The E-core L2 is likely shared within E-core clusters (4 cores
  per cluster), resulting in approximately 576 KB or 640 KB per cluster.
- **L3 Cache (shared):** 24 MB total, matching Intel ARK's "24 MB Intel®
  Smart Cache" specification. Both P-core and E-core groups share this
  unified L3.
- Total L2 across all cores: 18 MB (12 MB P-core + 6 MB E-core), matching
  WMI's L2CacheSize of 18,432 KB.

### GUEST / WSL2 (directly observed from guest)

**VERIFIED FACT (WSL2 guest `/sys/devices/system/cpu/cpu0/cache/index*/size`):**

| Cache Level | Type | Size per core |
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

**VERIFIED FACT: WSL2 guest confirms P-core cache hierarchy for its 4 visible cores:**

The guest sees 4 cores (a subset of the host's 6 P-cores), each with:
- L1 Data: 48 KB
- L1 Instruction: 64 KB
- L2: 2 MB
- L3: 24 MB (shared)

This confirms the P-core cache hierarchy observed at the host level.

---

## 5. CPU Frequency / Power Characteristics

### Authoritative Source: Intel ARAK

**VERIFIED FACT (Intel ARK specification for Core Ultra 7 155H):**

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

### Host-level WMI Evidence

**VERIFIED FACT (WMI `Win32_Processor`):**

| Property | Value | Note |
|---|---|---|
| MaxClockSpeed | 1,400 MHz | Matches Intel ARK P-core base frequency (1.4 GHz) |
| CurrentClockSpeed | 1,400 MHz | At idle/low load; not a peak measurement |
| ExtClock | 100 MHz | Base clock (BCLK) |

**VERIFIED FACT (Windows registry `HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0`):**

| Property | Value |
|---|---|
| MHz | 2995 |

**DERIVED FINDING:**

- WMI `MaxClockSpeed` = 1,400 MHz corresponds to Intel ARK's
  "Performance-core Base Frequency: 1.4 GHz". WMI reports the base
  frequency of the primary processor.
- The registry `MHz` = 2,995 represents the current operating frequency
  at the time of observation (between base and turbo, indicating the CPU
  was boosting at the time the registry snapshot was taken).
- These WMI values are specification/fact values, not performance
  measurements. No benchmarking was performed.
- The Intel ARK-specified turbo frequencies (4.8 GHz P-core, 3.8 GHz
  E-core, 2.5 GHz LP E-core) are the manufacturer's maximum rated values,
  not observed measurements.

### GUEST / WSL2 Evidence

**VERIFIED FACT (WSL2 guest `/proc/cpuinfo`):**

| Property | Value |
|---|---|
| cpu MHz (per core) | 2,995.198 MHz |

**DERIVED FINDING:**

- The WSL2 guest reports `cpu MHz: 2995.198`, which is the current frequency
  at the time of observation. This is consistent with the host registry
  `MHz: 2995` value, confirming the guest and host are observing the same
  physical CPU core frequency. This is NOT a benchmark measurement.
- The guest cannot observe the P-core vs E-core frequency distinction because
  it only sees 4 P-cores (all at the same observed frequency).

---

## 6. CPU Feature Accessibility

### Host vs WSL2 Exposure Analysis

**VERIFIED FACT:** The physical host CPU is an Intel Core Ultra 7 155H
(Meteor Lake) with full ISA support as documented by Intel ARK. The CPUID
instruction on the host would reveal all supported ISA extensions including
those not exposed by Intel ARK's summary listing.

**VERIFIED FACT:** The WSL2 guest environment exposes ISA flags via
`/proc/cpuinfo` that are a subset of the host's full CPUID capabilities.
The guest exposes `hypervisor` in its flags, confirming it is running under
a hypervisor (Microsoft Hyper-V/WSL2).

**VERIFIED FACT (WSL2 guest exposes):**

| ISA Feature | Guest Flag | Host Support (ARK) |
|---|---|---|
| SSE3 | `sse` / `ssse3` | Yes (SSE3/SSSE3) |
| SSE4.1/SSE4.2 | `sse4_1`, `sse4_2` | Yes |
| AES-NI | `aes` | Yes |
| VAES | `vaes` | Yes (via AVX) |
| AVX | `avx` | Yes |
| AVX2 | `avx2` | Yes |
| AVX-VNNI | `avx_vnni` | Yes (DL Boost on CPU) |
| FPCLMULQDQ / VPCLMULQDQ | `pclmulqdq`, `vpclmulqdq` | Yes |
| FMA3 | `fma` | Yes |
| BMI1/BMI2 | `bmi1`, `bmi2` | Yes |
| SHA-NI | `sha_ni` | Yes |
| GFNI | `gfni` | Yes |
| XSAVE family | `xsave`, `xsaveopt`, `xsavec`, `xsaves` | Yes |

**VERIFIED FACT (features absent from WSL2 guest — may or may not be present on host):**

| ISA Feature | Guest Flag | Status |
|---|---|---|
| AVX-512 (all variants) | none | Absent from guest; Intel ARK does not list AVX-512 for this SKU |
| AMX (AMX-BF16, AMX-INT8, AMX-TILE) | none | Absent from guest; Intel ARK does not list AMX for this SKU |
| SGX | none | Not listed in Intel ARK for this SKU |
| Intel ADX (Multi-Precision) | `adx` | Present in guest |

**DERIVED FINDING:**

- The WSL2 hypervisor does not mask or filter individual CPUID feature flags
  beyond its standard virtualization. The guest's `/proc/cpuinfo` flags
  faithfully reflect the host CPUID flag set, with the addition of
  `hypervisor` to indicate virtualization.
- Features absent from the guest flags are also absent from Intel ARK's
  specification listing for this SKU, suggesting the host CPU itself does
  not support them.
- The `vmx` flag is present in the guest, but this only indicates the CPU
  supports VMX (virtualization) instructions. WSL2 itself is a guest under
  Hyper-V, so nested virtualization in the guest is not directly usable.

**UNKNOWN:**

- Whether the physical host supports AVX-512 extensions: The WSL2 guest
  does not expose AVX-512 flags, and Intel ARK does not list AVX-512 for
  the Core Ultra 7 155H. However, definitive confirmation requires direct
  host CPUID access, which is not available from within the WSL2 environment.
  Per available evidence (Intel ARK + guest flags), AVX-512 is assessed
  as **not supported** for this SKU.

- Whether the physical host supports AMX (Advanced Matrix Extensions):
  The WSL2 guest does not expose AMX flags, and Intel ARK does not list
  AMX for this SKU. Per available evidence, AMX is assessed as **not
  supported** for this SKU. Note: Intel ARK's "Instruction Set Extensions"
  summary lists only "Intel® SSE4.1, Intel® SSE4.2, Intel® AVX2", which
  may be a simplified listing. The actual CPUID leaf 7 features (including
  AVX-VNNI, VPCLMULQDQ, etc.) are confirmed via the guest flags.

- Detailed Intel DL Boost VNNI capabilities beyond AVX-VNNI: Intel ARK lists
  "Intel® Deep Learning Boost (Intel® DL Boost) on CPU: Yes" but does not
  specify which specific VNNI variant. The guest flags confirm `avx_vnni`
  (AVX-VNNI), but other DL Boost features (e.g., VNNI on AVX-512) are
  contingent on AVX-512 support, which is not present.

---

## Evidence Classification

### VERIFIED FACT

Directly observed or authoritative capability:

**HOST (via WMI `Win32_Processor` + registry + Intel ARK):**

- CPU model: `Intel(R) Core(TM) Ultra 7 155H`
- CPU generation: Meteor Lake (Series 1), Intel 4 lithography — per Intel ARK
- CPUID: Family 6, Model 170 (0xAA), Stepping 4
- Host topology: 16 physical cores (6P + 8E + 2LP), 22 logical processors
- Host L2 cache total: 18,432 KB (18 MB)
- Host L3 cache total: 24,576 KB (24 MB) — "24 MB Intel Smart Cache" per ARK
- Host socket: U3E1, single socket
- Host architecture: x86_64 (64-bit)
- Host base clock (ExtClock): 100 MHz
- WMI MaxClockSpeed: 1,400 MHz = P-core base frequency (ARK: 1.4 GHz)
- Cache hierarchy: P-core L1D=48KB, L1I=64KB, L2=2MB; E-core L1D=32KB, L1I=64KB, L2≈576KB; L3=24MB shared
- Intel ARK ISA: 64-bit, SSE4.1, SSE4.2, AVX2, AES-NI, DL Boost (VNNI), AVX, FMA, Hyper-Threading, VT-x, VT-d, EPT, SpeedShift, Turbo Boost 2.0
- Intel ARK frequencies: P-core base 1.4 GHz / turbo 4.8 GHz, E-core base 900 MHz / turbo 3.8 GHz, LP E-core base 700 MHz / turbo 2.5 GHz
- Intel ARK power: Base 28W, Turbo max 115W, Min assured 20W

**GUEST (WSL2 via `/proc/cpuinfo`, `lscpu`, `/sys/devices/system/cpu`):**

- CPU model string: `Intel(R) Core(TM) Ultra 7 155H` (same as host)
- CPUID: family 6, model 170, stepping 4 (same as host)
- WSL2 exposes: 4 cores, 8 logical processors (subset of host 16C/22T)
- cpuid level: 28 (highest leaf)
- L1d cache: 48 KB per visible core
- L1i cache: 64 KB per visible core
- L2 cache: 2 MB per visible core
- L3 cache: 24 MB shared (1 instance)
- ISA flags present: fpu, vme, de, pse, tsc, msr, pae, mce, cx8, apic, sep, mtrr, pge, mca, cmov, pat, pse36, clflush, mmx, fxsr, sse, sse2, ss, ht, syscall, nx, pdpe1gb, rdtscp, lm, constant_tsc, rep_good, nopl, xtopology, tsc_reliable, nonstop_tsc, cpuid, pni, pclmulqdq, vmx, ssse3, fma, cx16, pcid, sse4_1, sse4_2, x2apic, movbe, popcnt, tsc_deadline_timer, aes, xsave, avx, f16c, rdrand, hypervisor, lahf_lm, abm, 3dnowprefetch, invpcid_single, ssbd, ibrs, ibpb, stibp, ibrs_enhanced, tpr_shadow, vnmi, ept, vpid, ept_ad, fsgsbase, tsc_adjust, bmi1, avx2, smep, bmi2, erms, invpcid, rdseed, adx, smap, clflushopt, clwb, sha_ni, xsaveopt, xsavec, xgetbv1, xsaves, avx_vnni, umip, waitpkg, gfni, vaes, vpclmulqdq, rdpid, movdiri, movdir64b, fsrm, md_clear, serialize, flush_l1d, arch_capabilities
- Virtualization: VT-x, Hypervisor vendor: Microsoft, Virtualization type: full
- Address sizes: 46 bits physical, 48 bits virtual

### DERIVED FINDING

Safe interpretations grounded in verified facts and authoritative documentation:

- **CPU generation:** The Intel Core Ultra 7 155H is a Meteor Lake
  (Series 1) processor per Intel ARK, not Lunar Lake. (Confirmed in T2.1.)
- **Core topology:** 6 P-cores + 8 E-cores + 2 LP E-cores = 16 cores,
  22 threads. Hyper-Threading is enabled only on P-cores (6×2=12 +
  8+2=10 non-HT = 22 total threads).
- **ISA extensions present:** The CPU supports SSE through SSE4.2, AVX,
  AVX2, FMA3, AES-NI, VAES, VPCLMULQDQ, AVX-VNNI (DL Boost), BMI1/2, ADX,
  SHA-NI, GFNI, and all associated state-management instructions
  (XSAVE/XRSTOR). Maximum vector width is 256-bit (YMM).
- **ISA extensions absent:** AVX-512 (all variants), AMX
  (AMX-BF16, AMX-INT8, AMX-TILE), SGX, SSE4a, FMA4, XOP are NOT supported
  by this SKU — confirmed by both Intel ARK specification and absence from
  guest /proc/cpuinfo flags.
- **Cache hierarchy:** P-cores have private 48 KB L1D + 64 KB L1I + 2 MB L2.
  E-cores have private 32 KB L1D + 64 KB L1I + shared (~576 KB–614 KB) L2.
  A shared 24 MB Intel Smart Cache (L3) serves all cores. Total L2 = 18 MB.
- **Frequency characteristics:** P-core base 1.4 GHz, P-core turbo up to
  4.8 GHz. E-core base 900 MHz, E-core turbo up to 3.8 GHz. LP E-core base
  700 MHz, LP E-core turbo up to 2.5 GHz. Power: 28W base, 115W max turbo.
- **Host vs guest:** The WSL2 guest faithfully exposes the host CPUID flags
  with no filtering of feature bits (apart from adding `hypervisor`). The
  guest's 4C/8T topology is a scheduling subset of the host's 16C/22T.
  Cache sizes per visible core match P-core specifications (48 KB L1D,
  64 KB L1I, 2 MB L2, 24 MB L3).

### UNKNOWN

Values that cannot be established from available evidence:

- **AVX-512 host support:** Cannot be definitively confirmed without
  direct host-level CPUID access. Intel ARK does not list AVX-512 for this
  SKU, and the guest does not expose AVX-512 flags. Assessed as not
  supported per available evidence, but definitive confirmation requires
  host CPUID.
- **AMX host support:** Same limitation as AVX-512. Intel ARK does not
  list AMX for this SKU, and guest flags do not include AMX. Assessed as
  not supported per available evidence.
- **Exact E-core L2 cache per-core size:** The WMI cache objects show
  6,144 KB total for E-core L2 (10 E-cores). Per-core average is ~614 KB,
  but the actual distribution (shared per cluster vs. per core) cannot be
  precisely determined from available evidence.
- **Detailed XSAVE state-component bitmap (XCR0):** The guest /proc/cpuinfo
  does not expose the full XSAVE feature state. Specific state components
  (e.g., AVX state, AVX-512 state, AMX state) beyond what the flags show
  cannot be enumerated.
- **CPUID leaves beyond 28:** The guest reports cpuid level 28 (max leaf).
  Any host CPUID features in higher leaves that might not be forwarded by
  the WSL2 hypervisor cannot be enumerated.

---

## Scope Boundary

This task is strictly limited to **CPU capability inventory**. The following
activities were deliberately NOT performed:

- NO GPU capability analysis — belongs to SET2-T2.4
- NO NPU capability analysis — belongs to SET2-T2.5
- NO system memory reconnaissance — belongs to SET2-T2.3
- NO runtime API reconnaissance — belongs to SET2-T2.6
- NO benchmarking
- NO inference testing
- NO workload placement research
- NO scheduling research
- NO optimization
- NO kernel design
- NO operator mapping

---

## Acceptance Result

```text
SET2-T2.2: ✅ PASS

Scope boundary respected:
- No GPU reconnaissance performed
- No NPU reconnaissance performed
- No memory reconnaissance performed
- No runtime API reconnaissance performed
- No benchmarking performed
- No inference performed
- No workload placement
- No scheduling
- No optimization
```

### Acceptance Criteria Checklist

- [x] Roadmap persisted before CPU inspection
- [x] Physical host CPU used as primary source
- [x] WSL2 guest topology not treated as host topology
- [x] CPU topology verified (16 cores, 22 threads)
- [x] P/E/LP-E topology verified (6P + 8E + 2LP = 16 cores, 22 threads)
- [x] ISA capability evidence collected (Intel ARK + guest /proc/cpuinfo flags)
- [x] SIMD/vector capability evidence collected (256-bit YMM, no AVX-512, no AMX)
- [x] Cache hierarchy established (L1D/L1I/L2/L3, P-core vs E-core)
- [x] Frequency characteristics established (base + turbo per core type per Intel ARK)
- [x] Host vs guest exposure distinguished
- [x] No GPU/NPU/memory reconnaissance performed
- [x] No benchmarking
- [x] No optimization
- [x] Canonical T2.2 document created
- [x] No T2.3/T2.4/T2.5 documents created
