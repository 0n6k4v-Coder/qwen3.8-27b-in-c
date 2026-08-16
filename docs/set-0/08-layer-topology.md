# SET 0 — Exact Language Layer Topology

## Document Status

- Document: `08-layer-topology.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T10`
- Status: VERIFIED
- Purpose: Canonical, deterministic layer-by-layer topology of the
  Qwen3.8-27B language model
- Responsibility: 🧠 LUNA

---

# 1. Source and Provenance

Model:

```text
Qwen3.8-27B
````

Official repository:

```text
Qwen/Qwen3.8-27B
```

Pinned revision:

```text
1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
```

Primary configuration artifact:

```text
model/official/config.json
```

Authoritative topology field:

```text
text_config.layer_types
```

Additional consistency field:

```text
text_config.full_attention_interval = 4
```

Related research checkpoints:

```text
docs/set-0/01-artifact-provenance.md
docs/set-0/02-model-identity.md
docs/set-0/04-core-architecture.md
docs/set-0/05-attention-architecture.md
docs/set-0/06-mlp-architecture.md
docs/set-0/07-vision-and-mtp.md
```

The local `config.json` has already been verified byte-for-byte against the
official artifact at the pinned revision.

---

# 2. Topology Summary

The Qwen3.8-27B language model contains:

```text
64 total language layers
```

The explicit `layer_types` array defines:

```text
48 × linear_attention
16 × full_attention
```

The exact repeating topology is:

```text
[ linear_attention
  linear_attention
  linear_attention
  full_attention ] × 16
```

Equivalent shorthand:

```text
[LA → LA → LA → FA] × 16
```

where:

```text
LA = linear_attention
FA = full_attention
```

---

# 3. Canonical Layer Table

| Layer | Layer Type         | Operator               | Common MLP           |
| ----: | ------------------ | ---------------------- | -------------------- |
|     0 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     1 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     2 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     3 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|     4 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     5 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     6 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     7 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|     8 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|     9 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    10 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    11 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    12 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    13 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    14 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    15 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    16 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    17 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    18 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    19 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    20 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    21 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    22 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    23 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    24 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    25 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    26 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    27 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    28 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    29 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    30 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    31 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    32 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    33 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    34 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    35 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    36 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    37 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    38 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    39 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    40 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    41 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    42 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    43 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    44 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    45 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    46 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    47 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    48 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    49 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    50 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    51 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    52 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    53 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    54 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    55 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    56 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    57 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    58 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    59 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |
|    60 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    61 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    62 | `linear_attention` | `Qwen3_5GatedDeltaNet` | Dense Gated SiLU MLP |
|    63 | `full_attention`   | `Qwen3_5Attention`     | Dense Gated SiLU MLP |

---

# 4. Full-Attention Layer Indices

The complete full-attention index set is:

```text
3, 7, 11, 15,
19, 23, 27, 31,
35, 39, 43, 47,
51, 55, 59, 63
```

Count:

```text
16
```

---

# 5. Linear-Attention Layer Indices

The complete linear-attention index set is:

```text
0, 1, 2,
4, 5, 6,
8, 9, 10,
12, 13, 14,
16, 17, 18,
20, 21, 22,
24, 25, 26,
28, 29, 30,
32, 33, 34,
36, 37, 38,
40, 41, 42,
44, 45, 46,
48, 49, 50,
52, 53, 54,
56, 57, 58,
60, 61, 62
```

Count:

```text
48
```

---

# 6. Topology Validation

Total:

```text
48 + 16 = 64
```

Therefore every language layer is accounted for.

Validation requirements:

```text
Layer indices:
0 through 63       → complete

Duplicate indices:
none                → PASS

Missing indices:
none                → PASS

Layer types:
exactly one/layer   → PASS
```

Overall topology accounting:

```text
PASS
```

---

# 7. Placement Rule

The explicit `layer_types` array yields:

```text
[LA → LA → LA → FA] × 16
```

where full attention occurs at:

```text
layer_index % 4 == 3
```

and linear attention occurs at:

```text
layer_index % 4 != 3
```

This rule is a **derived shorthand**.

The authoritative source remains the explicit 64-entry `layer_types`
array.

---

# 8. Full-Attention Interval Consistency

Configuration:

```text
full_attention_interval = 4
```

Observed full-attention indices:

```text
3, 7, 11, 15, ..., 63
```

Difference between consecutive full-attention indices:

```text
7  - 3  = 4
11 - 7  = 4
15 - 11 = 4
...
63 - 59 = 4
```

Therefore:

```text
Explicit layer_types
        +
full_attention_interval = 4
        ↓
CONSISTENT
```

The interval field is therefore a successful consistency check against the
explicit topology.

---

# 9. Layer Operator Mapping

The verified architecture/implementation mapping is:

```text
linear_attention
        ↓
Qwen3_5GatedDeltaNet
```

and:

```text
full_attention
        ↓
Qwen3_5Attention
```

Therefore each language layer is represented conceptually as:

```text
Layer N
├── Token Mixer
│   ├── Qwen3_5GatedDeltaNet
│   └── Qwen3_5Attention
│
└── Dense Gated SiLU MLP
```

Only one token-mixer branch applies to a given layer.

---

# 10. Common MLP

The verified language architecture uses the same dense gated SiLU MLP
structure throughout the language decoder:

```text
gate_proj
up_proj
↓
SiLU(gate_proj(x)) ⊙ up_proj(x)
↓
down_proj
```

Therefore both layer families have:

```text
Token Mixer
+
Dense Gated SiLU MLP
```

The token mixer changes according to `layer_types`; the common MLP does not.

---

# 11. Topology Diagram

```text
                    Qwen3.8-27B
                         │
                 64 Language Layers
                         │
          ┌──────────────┴──────────────┐
          │                             │
      48 × Linear                   16 × Full
       Attention                     Attention
          │                             │
 Qwen3_5GatedDeltaNet            Qwen3_5Attention
          │                             │
          └──────────────┬──────────────┘
                         │
              Dense Gated SiLU MLP
```

Sequentially:

```text
L00 LA ─┐
L01 LA  │
L02 LA  │
L03 FA  ┘ Group 0

L04 LA ─┐
L05 LA  │
L06 LA  │
L07 FA  ┘ Group 1

...

L60 LA ─┐
L61 LA  │
L62 LA  │
L63 FA  ┘ Group 15
```

Canonical pattern:

```text
[LA → LA → LA → FA] × 16
```

---

# 12. Quantitative Topology

Linear-attention proportion:

```text
48 / 64 = 75%
```

Full-attention proportion:

```text
16 / 64 = 25%
```

Thus:

```text
Linear Attention : Full Attention
48 : 16
= 3 : 1
```

These are **topology ratios only**.

They must not be interpreted as:

* FLOP ratios
* parameter ratios
* memory ratios
* inference-time ratios

---

# 13. Verified Facts

```text
✅ num_hidden_layers = 64

✅ layer_types contains exactly 64 entries

✅ Every layer index 0–63 is represented

✅ No duplicate layer index

✅ No missing layer index

✅ 48 linear_attention layers

✅ 16 full_attention layers

✅ Full-attention indices explicitly identified

✅ Linear-attention indices explicitly identified

✅ full_attention_interval = 4

✅ Explicit topology agrees with the interval configuration

✅ linear_attention maps to Qwen3_5GatedDeltaNet

✅ full_attention maps to Qwen3_5Attention

✅ Common Dense Gated SiLU MLP is present across language layers
```

---

# 14. Derived Findings

```text
48 + 16 = 64
```

```text
48 / 16 = 3
```

Therefore:

```text
Linear : Full = 3 : 1
```

and:

```text
Linear = 75% of language layers
Full   = 25% of language layers
```

The deterministic placement shorthand is:

```text
layer_index % 4 == 3
    → full_attention
```

---

# 15. Inferences

### I1 — Operator dispatch can be deterministic

Once the explicit artifact topology has been verified, a runtime can use
the layer index to select the token-mixer family.

### I2 — Attention state management can be layer-type aware

The topology allows the runtime to know ahead of execution whether a given
layer requires the state model associated with:

```text
Gated DeltaNet
```

or:

```text
Full Attention
```

### I3 — Tensor mapping can be organized by layer family

Later tensor audit can classify tensors into:

```text
Linear-Attention layer tensors
Full-Attention layer tensors
Common MLP tensors
```

using this verified topology as its layer map.

---

# 16. Anomalies / Unknowns

No anomaly was found in the verified 64-layer topology.

```text
Topology anomaly:
NONE
```

Remaining questions are outside this document's scope:

```text
- exact tensor inventory per layer
- exact normalization tensor placement
- exact parameter distribution
- exact checkpoint shard mapping
- runtime memory requirements
- hardware workload placement
```

---

# 17. Research Boundary

This document establishes the **exact language-layer topology**.

It does not establish:

```text
- tensor names and shapes
- parameter counts by tensor
- checkpoint size
- tensor-to-shard mapping
- runtime memory
- execution scheduling
- performance
- hardware placement
```

Those topics require subsequent research tasks.

---

# 18. Canonical Topology Statement

> **Qwen3.8-27B contains exactly 64 language layers. The authoritative
> `layer_types` array defines 48 `linear_attention` layers and 16
> `full_attention` layers in the exact repeating pattern
> `[linear_attention, linear_attention, linear_attention, full_attention] ×
> 16`. Full-attention layers occur at indices 3, 7, 11, 15, ..., 63,
> exactly matching `full_attention_interval = 4`. Every language layer
> contains the common dense gated SiLU MLP component.**

---

# 19. Final Acceptance

```text
TOPOLOGY STATUS:
VERIFIED

TOTAL LANGUAGE LAYERS:
64

LINEAR-ATTENTION:
48

FULL-ATTENTION:
16

PATTERN:
[LA → LA → LA → FA] × 16

FULL-ATTENTION INDICES:
3, 7, 11, 15, 19, 23, 27, 31,
35, 39, 43, 47, 51, 55, 59, 63

TOPOLOGY CONSISTENCY:
PASS

SET0-T10:
PASS
```

This document is the canonical `SET0-T10` exact language-layer topology
checkpoint.