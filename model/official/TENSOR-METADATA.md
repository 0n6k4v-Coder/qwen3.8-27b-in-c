# Qwen3.8-27B — Tensor Metadata

> **Authoritative tensor metadata acquired directly from safetensors shard headers**
> 
> This document contains raw tensor metadata (name, shape, dtype, source shard)
> acquired from the official checkpoint at the pinned revision. It is intended
> for downstream parameter and byte-accounting analysis by Luna.
> 
> This document performs **evidence acquisition only**. It does not compute
> parameter counts, byte totals, or memory footprints.

---

## 1. Source and Provenance

```text
Model:           Qwen3.8-27B
Repository:      Qwen/Qwen3.8-27B
Pinned revision: 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
Index artifact:  model/official/model.safetensors.index.json
Config artifact: model/official/config.json
```

## 2. Metadata Acquisition Method

Tensor metadata was acquired by reading **safetensors shard headers only**,
**without downloading or materializing tensor payloads**.

The safetensors binary format stores a JSON header at the beginning of each
shard file. The header begins with an 8-byte little-endian unsigned integer
(uint64) specifying the JSON header length, immediately followed by the JSON
text. The JSON header maps each tensor name to:

```json
{"dtype": "BF16", "shape": [5120], "data_offsets": [0, 10240]}
```

Each shard header was read via a bounded range request
(`huggingface_hub.HfFileSystem.read_bytes` with `start`/`end`) that fetches
only the 8-byte length prefix and the JSON header bytes. No tensor payload
data was transferred or loaded into memory.

The acquired metadata was cross-checked against the local
`model.safetensors.index.json` to verify tensor→shard mapping, shape, and
dtype completeness.

---

## 3. Checkpoint Summary

| Property | Value |
|---|---|
| Total indexed tensors | 1199 |
| Total header-resolved tensors | 1199 |
| Shard count | 18 |
| Index `total_size` | 55,562,855,904 bytes |
| Universal dtype | {'BF16': 1199} |
| Data offsets type | Per-shard byte offsets (relative to shard data start) |

### Shard Inventory

| Shard | Tensors (header) | Tensors (index) | File Size (bytes) |
|---|---|---|---|
| model-00001-of-00018.safetensors | 392 | 392 | 3,966,730,552 |
| model-00002-of-00018.safetensors | 47 | 47 | 3,043,080,328 |
| model-00003-of-00018.safetensors | 1 | 1 | 2,542,796,952 |
| model-00004-of-00018.safetensors | 69 | 69 | 3,988,973,152 |
| model-00005-of-00018.safetensors | 37 | 37 | 2,099,339,864 |
| model-00006-of-00018.safetensors | 76 | 76 | 3,979,553,696 |
| model-00007-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00008-of-00018.safetensors | 76 | 76 | 3,979,553,696 |
| model-00009-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00010-of-00018.safetensors | 76 | 76 | 3,979,553,696 |
| model-00011-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00012-of-00018.safetensors | 76 | 76 | 3,979,553,696 |
| model-00013-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00014-of-00018.safetensors | 76 | 76 | 3,979,553,696 |
| model-00015-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00016-of-00018.safetensors | 77 | 77 | 3,979,564,040 |
| model-00017-of-00018.safetensors | 30 | 30 | 2,108,759,344 |
| model-00018-of-00018.safetensors | 16 | 16 | 3,392,197,344 |
| **Total** | **1199** | **1199** | **55,563,006,776** |

> The sum of shard file sizes (55,563,006,776) exceeds the index `total_size`
> (55,562,855,904) by 150,872 bytes. This difference
> is safetensors shard trailing padding (page-alignment) and is expected. The
> index `total_size` is the authoritative checkpoint payload size.

---

## 4. Cross-Check Verification

The acquired header metadata was cross-checked against
`model/official/model.safetensors.index.json`.

| Check | Result |
|---|---|
| 1. Every indexed tensor resolves to a shard | PASS |
| 2. Every resolved tensor has metadata | PASS |
| 3. Tensor names agree with index | PASS |
| 4. Shapes available for all tensors | PASS |
| 5. Dtypes available for all tensors | PASS |
| 6. No unexpected tensor names discovered | PASS |
| 7. Indexed shard set remains 18 shards | PASS |
| 8. Pinned revision is metadata source | PASS |

- Indexed tensors missing from headers: **none**
- Unexpected tensors discovered in headers: **none**

```text
INDEX CROSS-CHECK: PASS
  Indexed tensors:        1199
  Header-resolved tensors: 1199
  Missing from headers:    0
  Unexpected in headers:   0
  Shard count:             18
  Shard set match:         True
```

---

## 5. Storage Dtype

| DType | Count |
|---|---|
| `BF16` | 1199 |

> All 1199 tensors are stored as `BF16` (bfloat16). This matches the
> `dtype: bfloat16` setting in `config.json` (`text_config.dtype`).

---

## 6. Language Global Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.language_model.embed_tokens.weight` | [248320, 5120] | `BF16` | `model-00003-of-00018.safetensors` |
| `model.language_model.norm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `lm_head.weight` | [248320, 5120] | `BF16` | `model-00018-of-00018.safetensors` |

---

## 7. Language MLP Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.language_model.layers.0.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.10.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.15.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.15.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.16.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.2.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.20.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.21.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.21.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.24.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.29.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.29.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.3.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.30.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.32.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.37.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.37.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.4.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.4.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.4.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.40.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.45.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.45.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.48.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.5.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.50.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.53.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.53.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.56.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.6.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.60.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.61.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.61.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.7.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.8.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `mtp.layers.0.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |

### MLP Shape Summary (unique shapes)

| Shape | Dtype | Tensor Count |
|---|---|---|
| [5120, 17408] | `BF16` | 65 |
| [17408, 5120] | `BF16` | 130 |

---

## 8. Full-Attention Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.language_model.layers.11.self_attn.k_norm.weight` | [256] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.self_attn.q_norm.weight` | [256] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.k_norm.weight` | [256] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.q_norm.weight` | [256] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.k_norm.weight` | [256] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.q_norm.weight` | [256] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.k_norm.weight` | [256] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.q_norm.weight` | [256] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.k_norm.weight` | [256] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.q_norm.weight` | [256] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.k_norm.weight` | [256] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.q_norm.weight` | [256] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.k_norm.weight` | [256] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.q_norm.weight` | [256] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.k_norm.weight` | [256] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.q_norm.weight` | [256] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.k_norm.weight` | [256] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.q_norm.weight` | [256] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.k_norm.weight` | [256] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.q_norm.weight` | [256] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.k_norm.weight` | [256] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.q_norm.weight` | [256] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.k_norm.weight` | [256] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.q_norm.weight` | [256] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.k_norm.weight` | [256] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.q_norm.weight` | [256] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.k_norm.weight` | [256] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.q_norm.weight` | [256] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.k_norm.weight` | [256] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.q_norm.weight` | [256] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.k_norm.weight` | [256] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.q_norm.weight` | [256] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `mtp.layers.0.self_attn.k_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.q_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |

### Full-Attention Shape Summary (unique shapes)

| Shape | Dtype | Tensor Count |
|---|---|---|
| [256] | `BF16` | 34 |
| [1024, 5120] | `BF16` | 34 |
| [5120, 6144] | `BF16` | 17 |
| [12288, 5120] | `BF16` | 17 |

---

## 9. Linear-Attention Tensors (Qwen3_5GatedDeltaNet)

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.language_model.layers.0.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.A_log` | [48] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.dt_bias` | [48] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.norm.weight` | [128] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.A_log` | [48] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.dt_bias` | [48] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.norm.weight` | [128] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.A_log` | [48] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.dt_bias` | [48] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.norm.weight` | [128] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.4.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.A_log` | [48] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.dt_bias` | [48] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.norm.weight` | [128] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.A_log` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.dt_bias` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.A_log` | [48] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.dt_bias` | [48] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.norm.weight` | [128] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.A_log` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.dt_bias` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.A_log` | [48] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.dt_bias` | [48] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.norm.weight` | [128] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.A_log` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.dt_bias` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.norm.weight` | [128] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.A_log` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.dt_bias` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.norm.weight` | [128] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |

### Linear-Attention Shape Summary (unique shapes)

| Shape | Dtype | Tensor Count |
|---|---|---|
| [48] | `BF16` | 96 |
| [48, 5120] | `BF16` | 96 |
| [128] | `BF16` | 48 |
| [5120, 6144] | `BF16` | 48 |
| [6144, 5120] | `BF16` | 48 |
| [10240, 1, 4] | `BF16` | 48 |
| [10240, 5120] | `BF16` | 48 |

---

## 10. Language Normalization Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.language_model.layers.0.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.0.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.1.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.10.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.10.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.11.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.12.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.13.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.14.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.15.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.15.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| `model.language_model.layers.16.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.16.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.17.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.18.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.19.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.2.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.2.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.20.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.20.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| `model.language_model.layers.21.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.input_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.22.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.input_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.23.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| `model.language_model.layers.24.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.24.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.25.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.26.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.27.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.28.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| `model.language_model.layers.29.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.3.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.3.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.30.input_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.30.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.input_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.31.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| `model.language_model.layers.32.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.32.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.33.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.34.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.35.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.36.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| `model.language_model.layers.37.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.input_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.38.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.input_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.39.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| `model.language_model.layers.4.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.language_model.layers.4.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.40.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.40.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.41.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.42.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.43.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.44.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| `model.language_model.layers.45.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.input_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.46.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.input_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.47.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| `model.language_model.layers.48.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.48.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.49.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.5.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.5.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.50.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.50.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.51.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.52.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| `model.language_model.layers.53.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.input_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.54.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.input_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.55.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| `model.language_model.layers.56.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.56.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.57.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.58.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.59.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.6.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.6.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.60.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.60.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| `model.language_model.layers.61.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.input_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.62.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.input_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.63.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| `model.language_model.layers.7.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.7.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| `model.language_model.layers.8.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.8.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `model.language_model.layers.9.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| `mtp.layers.0.input_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |

---

## 11. Vision Subsystem Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `model.visual.blocks.0.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.0.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.1.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.10.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.11.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.12.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.13.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.14.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.15.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.16.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.17.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.18.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.19.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.2.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.20.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.21.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.22.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.23.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.24.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.25.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.26.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.3.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.4.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.5.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.6.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.7.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.8.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.blocks.9.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.linear_fc1.bias` | [4608] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.linear_fc1.weight` | [4608, 4608] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.linear_fc2.bias` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.linear_fc2.weight` | [5120, 4608] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.norm.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.merger.norm.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.patch_embed.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.patch_embed.proj.weight` | [1152, 3, 2, 16, 16] | `BF16` | `model-00001-of-00018.safetensors` |
| `model.visual.pos_embed.weight` | [2304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |

### Vision Tensor Count by Family

| Family | Tensor Count |
|---|---|
| blocks.attn | 108 |
| blocks.mlp | 108 |
| blocks.norm | 108 |
| merger | 6 |
| patch_embed | 2 |
| pos_embed | 1 |

---

## 12. MTP (Multi-Token Prediction) Tensors

| Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|
| `mtp.fc.weight` | [5120, 10240] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.input_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.k_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.q_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.layers.0.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.norm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.pre_fc_norm_embedding.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| `mtp.pre_fc_norm_hidden.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |

---

## 13. Complete Tensor Inventory

The following table lists every indexed tensor with its authoritative shape,
dtype, and source shard as resolved from the safetensors headers.

| # | Tensor Name | Shape | Dtype | Source Shard |
|---|---|---|---|---|
| 1 | `lm_head.weight` | [248320, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 2 | `model.language_model.embed_tokens.weight` | [248320, 5120] | `BF16` | `model-00003-of-00018.safetensors` |
| 3 | `model.language_model.layers.0.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 4 | `model.language_model.layers.0.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 5 | `model.language_model.layers.0.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| 6 | `model.language_model.layers.0.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 7 | `model.language_model.layers.0.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 8 | `model.language_model.layers.0.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 9 | `model.language_model.layers.0.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 10 | `model.language_model.layers.0.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 11 | `model.language_model.layers.0.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| 12 | `model.language_model.layers.0.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| 13 | `model.language_model.layers.0.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| 14 | `model.language_model.layers.0.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 15 | `model.language_model.layers.0.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 16 | `model.language_model.layers.0.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 17 | `model.language_model.layers.1.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 18 | `model.language_model.layers.1.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 19 | `model.language_model.layers.1.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| 20 | `model.language_model.layers.1.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 21 | `model.language_model.layers.1.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 22 | `model.language_model.layers.1.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 23 | `model.language_model.layers.1.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 24 | `model.language_model.layers.1.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 25 | `model.language_model.layers.1.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| 26 | `model.language_model.layers.1.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| 27 | `model.language_model.layers.1.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| 28 | `model.language_model.layers.1.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 29 | `model.language_model.layers.1.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 30 | `model.language_model.layers.1.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 31 | `model.language_model.layers.10.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 32 | `model.language_model.layers.10.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 33 | `model.language_model.layers.10.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| 34 | `model.language_model.layers.10.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 35 | `model.language_model.layers.10.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 36 | `model.language_model.layers.10.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 37 | `model.language_model.layers.10.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 38 | `model.language_model.layers.10.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 39 | `model.language_model.layers.10.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| 40 | `model.language_model.layers.10.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| 41 | `model.language_model.layers.10.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| 42 | `model.language_model.layers.10.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 43 | `model.language_model.layers.10.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 44 | `model.language_model.layers.10.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 45 | `model.language_model.layers.11.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 46 | `model.language_model.layers.11.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| 47 | `model.language_model.layers.11.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 48 | `model.language_model.layers.11.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 49 | `model.language_model.layers.11.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 50 | `model.language_model.layers.11.self_attn.k_norm.weight` | [256] | `BF16` | `model-00004-of-00018.safetensors` |
| 51 | `model.language_model.layers.11.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 52 | `model.language_model.layers.11.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| 53 | `model.language_model.layers.11.self_attn.q_norm.weight` | [256] | `BF16` | `model-00004-of-00018.safetensors` |
| 54 | `model.language_model.layers.11.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 55 | `model.language_model.layers.11.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 56 | `model.language_model.layers.12.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 57 | `model.language_model.layers.12.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 58 | `model.language_model.layers.12.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| 59 | `model.language_model.layers.12.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 60 | `model.language_model.layers.12.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 61 | `model.language_model.layers.12.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 62 | `model.language_model.layers.12.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 63 | `model.language_model.layers.12.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 64 | `model.language_model.layers.12.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| 65 | `model.language_model.layers.12.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| 66 | `model.language_model.layers.12.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| 67 | `model.language_model.layers.12.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 68 | `model.language_model.layers.12.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 69 | `model.language_model.layers.12.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 70 | `model.language_model.layers.13.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 71 | `model.language_model.layers.13.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 72 | `model.language_model.layers.13.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| 73 | `model.language_model.layers.13.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 74 | `model.language_model.layers.13.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 75 | `model.language_model.layers.13.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 76 | `model.language_model.layers.13.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 77 | `model.language_model.layers.13.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 78 | `model.language_model.layers.13.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| 79 | `model.language_model.layers.13.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| 80 | `model.language_model.layers.13.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| 81 | `model.language_model.layers.13.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 82 | `model.language_model.layers.13.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 83 | `model.language_model.layers.13.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 84 | `model.language_model.layers.14.input_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 85 | `model.language_model.layers.14.linear_attn.A_log` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 86 | `model.language_model.layers.14.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00004-of-00018.safetensors` |
| 87 | `model.language_model.layers.14.linear_attn.dt_bias` | [48] | `BF16` | `model-00004-of-00018.safetensors` |
| 88 | `model.language_model.layers.14.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 89 | `model.language_model.layers.14.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 90 | `model.language_model.layers.14.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 91 | `model.language_model.layers.14.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 92 | `model.language_model.layers.14.linear_attn.norm.weight` | [128] | `BF16` | `model-00004-of-00018.safetensors` |
| 93 | `model.language_model.layers.14.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00004-of-00018.safetensors` |
| 94 | `model.language_model.layers.14.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00004-of-00018.safetensors` |
| 95 | `model.language_model.layers.14.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 96 | `model.language_model.layers.14.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 97 | `model.language_model.layers.14.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 98 | `model.language_model.layers.15.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 99 | `model.language_model.layers.15.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| 100 | `model.language_model.layers.15.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 101 | `model.language_model.layers.15.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 102 | `model.language_model.layers.15.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00004-of-00018.safetensors` |
| 103 | `model.language_model.layers.15.self_attn.k_norm.weight` | [256] | `BF16` | `model-00005-of-00018.safetensors` |
| 104 | `model.language_model.layers.15.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 105 | `model.language_model.layers.15.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |
| 106 | `model.language_model.layers.15.self_attn.q_norm.weight` | [256] | `BF16` | `model-00005-of-00018.safetensors` |
| 107 | `model.language_model.layers.15.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 108 | `model.language_model.layers.15.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 109 | `model.language_model.layers.16.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 110 | `model.language_model.layers.16.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 111 | `model.language_model.layers.16.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| 112 | `model.language_model.layers.16.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 113 | `model.language_model.layers.16.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 114 | `model.language_model.layers.16.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 115 | `model.language_model.layers.16.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 116 | `model.language_model.layers.16.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 117 | `model.language_model.layers.16.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| 118 | `model.language_model.layers.16.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| 119 | `model.language_model.layers.16.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| 120 | `model.language_model.layers.16.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 121 | `model.language_model.layers.16.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 122 | `model.language_model.layers.16.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 123 | `model.language_model.layers.17.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 124 | `model.language_model.layers.17.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 125 | `model.language_model.layers.17.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| 126 | `model.language_model.layers.17.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 127 | `model.language_model.layers.17.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 128 | `model.language_model.layers.17.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 129 | `model.language_model.layers.17.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 130 | `model.language_model.layers.17.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 131 | `model.language_model.layers.17.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| 132 | `model.language_model.layers.17.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| 133 | `model.language_model.layers.17.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| 134 | `model.language_model.layers.17.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 135 | `model.language_model.layers.17.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 136 | `model.language_model.layers.17.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 137 | `model.language_model.layers.18.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 138 | `model.language_model.layers.18.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 139 | `model.language_model.layers.18.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| 140 | `model.language_model.layers.18.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 141 | `model.language_model.layers.18.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 142 | `model.language_model.layers.18.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 143 | `model.language_model.layers.18.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 144 | `model.language_model.layers.18.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 145 | `model.language_model.layers.18.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| 146 | `model.language_model.layers.18.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| 147 | `model.language_model.layers.18.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| 148 | `model.language_model.layers.18.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 149 | `model.language_model.layers.18.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 150 | `model.language_model.layers.18.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 151 | `model.language_model.layers.19.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 152 | `model.language_model.layers.19.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| 153 | `model.language_model.layers.19.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 154 | `model.language_model.layers.19.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 155 | `model.language_model.layers.19.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 156 | `model.language_model.layers.19.self_attn.k_norm.weight` | [256] | `BF16` | `model-00006-of-00018.safetensors` |
| 157 | `model.language_model.layers.19.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 158 | `model.language_model.layers.19.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| 159 | `model.language_model.layers.19.self_attn.q_norm.weight` | [256] | `BF16` | `model-00006-of-00018.safetensors` |
| 160 | `model.language_model.layers.19.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 161 | `model.language_model.layers.19.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 162 | `model.language_model.layers.2.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 163 | `model.language_model.layers.2.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 164 | `model.language_model.layers.2.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| 165 | `model.language_model.layers.2.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 166 | `model.language_model.layers.2.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 167 | `model.language_model.layers.2.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 168 | `model.language_model.layers.2.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 169 | `model.language_model.layers.2.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 170 | `model.language_model.layers.2.linear_attn.norm.weight` | [128] | `BF16` | `model-00001-of-00018.safetensors` |
| 171 | `model.language_model.layers.2.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| 172 | `model.language_model.layers.2.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| 173 | `model.language_model.layers.2.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 174 | `model.language_model.layers.2.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 175 | `model.language_model.layers.2.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 176 | `model.language_model.layers.20.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 177 | `model.language_model.layers.20.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 178 | `model.language_model.layers.20.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| 179 | `model.language_model.layers.20.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 180 | `model.language_model.layers.20.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 181 | `model.language_model.layers.20.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 182 | `model.language_model.layers.20.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 183 | `model.language_model.layers.20.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 184 | `model.language_model.layers.20.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| 185 | `model.language_model.layers.20.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00006-of-00018.safetensors` |
| 186 | `model.language_model.layers.20.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00006-of-00018.safetensors` |
| 187 | `model.language_model.layers.20.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 188 | `model.language_model.layers.20.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 189 | `model.language_model.layers.20.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 190 | `model.language_model.layers.21.input_layernorm.weight` | [5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 191 | `model.language_model.layers.21.linear_attn.A_log` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 192 | `model.language_model.layers.21.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00006-of-00018.safetensors` |
| 193 | `model.language_model.layers.21.linear_attn.dt_bias` | [48] | `BF16` | `model-00006-of-00018.safetensors` |
| 194 | `model.language_model.layers.21.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 195 | `model.language_model.layers.21.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 196 | `model.language_model.layers.21.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 197 | `model.language_model.layers.21.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00006-of-00018.safetensors` |
| 198 | `model.language_model.layers.21.linear_attn.norm.weight` | [128] | `BF16` | `model-00006-of-00018.safetensors` |
| 199 | `model.language_model.layers.21.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| 200 | `model.language_model.layers.21.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| 201 | `model.language_model.layers.21.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 202 | `model.language_model.layers.21.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 203 | `model.language_model.layers.21.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 204 | `model.language_model.layers.22.input_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 205 | `model.language_model.layers.22.linear_attn.A_log` | [48] | `BF16` | `model-00007-of-00018.safetensors` |
| 206 | `model.language_model.layers.22.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00007-of-00018.safetensors` |
| 207 | `model.language_model.layers.22.linear_attn.dt_bias` | [48] | `BF16` | `model-00007-of-00018.safetensors` |
| 208 | `model.language_model.layers.22.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 209 | `model.language_model.layers.22.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 210 | `model.language_model.layers.22.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 211 | `model.language_model.layers.22.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 212 | `model.language_model.layers.22.linear_attn.norm.weight` | [128] | `BF16` | `model-00007-of-00018.safetensors` |
| 213 | `model.language_model.layers.22.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| 214 | `model.language_model.layers.22.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| 215 | `model.language_model.layers.22.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 216 | `model.language_model.layers.22.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 217 | `model.language_model.layers.22.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 218 | `model.language_model.layers.23.input_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 219 | `model.language_model.layers.23.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00007-of-00018.safetensors` |
| 220 | `model.language_model.layers.23.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 221 | `model.language_model.layers.23.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 222 | `model.language_model.layers.23.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 223 | `model.language_model.layers.23.self_attn.k_norm.weight` | [256] | `BF16` | `model-00007-of-00018.safetensors` |
| 224 | `model.language_model.layers.23.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 225 | `model.language_model.layers.23.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00007-of-00018.safetensors` |
| 226 | `model.language_model.layers.23.self_attn.q_norm.weight` | [256] | `BF16` | `model-00007-of-00018.safetensors` |
| 227 | `model.language_model.layers.23.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 228 | `model.language_model.layers.23.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00007-of-00018.safetensors` |
| 229 | `model.language_model.layers.24.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 230 | `model.language_model.layers.24.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 231 | `model.language_model.layers.24.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| 232 | `model.language_model.layers.24.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 233 | `model.language_model.layers.24.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 234 | `model.language_model.layers.24.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 235 | `model.language_model.layers.24.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 236 | `model.language_model.layers.24.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 237 | `model.language_model.layers.24.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| 238 | `model.language_model.layers.24.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| 239 | `model.language_model.layers.24.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| 240 | `model.language_model.layers.24.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 241 | `model.language_model.layers.24.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 242 | `model.language_model.layers.24.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 243 | `model.language_model.layers.25.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 244 | `model.language_model.layers.25.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 245 | `model.language_model.layers.25.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| 246 | `model.language_model.layers.25.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 247 | `model.language_model.layers.25.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 248 | `model.language_model.layers.25.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 249 | `model.language_model.layers.25.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 250 | `model.language_model.layers.25.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 251 | `model.language_model.layers.25.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| 252 | `model.language_model.layers.25.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| 253 | `model.language_model.layers.25.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| 254 | `model.language_model.layers.25.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 255 | `model.language_model.layers.25.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 256 | `model.language_model.layers.25.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 257 | `model.language_model.layers.26.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 258 | `model.language_model.layers.26.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 259 | `model.language_model.layers.26.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| 260 | `model.language_model.layers.26.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 261 | `model.language_model.layers.26.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 262 | `model.language_model.layers.26.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 263 | `model.language_model.layers.26.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 264 | `model.language_model.layers.26.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 265 | `model.language_model.layers.26.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| 266 | `model.language_model.layers.26.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| 267 | `model.language_model.layers.26.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| 268 | `model.language_model.layers.26.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 269 | `model.language_model.layers.26.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 270 | `model.language_model.layers.26.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 271 | `model.language_model.layers.27.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 272 | `model.language_model.layers.27.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| 273 | `model.language_model.layers.27.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 274 | `model.language_model.layers.27.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 275 | `model.language_model.layers.27.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 276 | `model.language_model.layers.27.self_attn.k_norm.weight` | [256] | `BF16` | `model-00008-of-00018.safetensors` |
| 277 | `model.language_model.layers.27.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 278 | `model.language_model.layers.27.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| 279 | `model.language_model.layers.27.self_attn.q_norm.weight` | [256] | `BF16` | `model-00008-of-00018.safetensors` |
| 280 | `model.language_model.layers.27.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 281 | `model.language_model.layers.27.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 282 | `model.language_model.layers.28.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 283 | `model.language_model.layers.28.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 284 | `model.language_model.layers.28.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| 285 | `model.language_model.layers.28.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 286 | `model.language_model.layers.28.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 287 | `model.language_model.layers.28.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 288 | `model.language_model.layers.28.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 289 | `model.language_model.layers.28.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 290 | `model.language_model.layers.28.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| 291 | `model.language_model.layers.28.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00008-of-00018.safetensors` |
| 292 | `model.language_model.layers.28.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00008-of-00018.safetensors` |
| 293 | `model.language_model.layers.28.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 294 | `model.language_model.layers.28.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 295 | `model.language_model.layers.28.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 296 | `model.language_model.layers.29.input_layernorm.weight` | [5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 297 | `model.language_model.layers.29.linear_attn.A_log` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 298 | `model.language_model.layers.29.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00008-of-00018.safetensors` |
| 299 | `model.language_model.layers.29.linear_attn.dt_bias` | [48] | `BF16` | `model-00008-of-00018.safetensors` |
| 300 | `model.language_model.layers.29.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 301 | `model.language_model.layers.29.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 302 | `model.language_model.layers.29.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 303 | `model.language_model.layers.29.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00008-of-00018.safetensors` |
| 304 | `model.language_model.layers.29.linear_attn.norm.weight` | [128] | `BF16` | `model-00008-of-00018.safetensors` |
| 305 | `model.language_model.layers.29.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| 306 | `model.language_model.layers.29.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| 307 | `model.language_model.layers.29.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 308 | `model.language_model.layers.29.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 309 | `model.language_model.layers.29.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 310 | `model.language_model.layers.3.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 311 | `model.language_model.layers.3.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00001-of-00018.safetensors` |
| 312 | `model.language_model.layers.3.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 313 | `model.language_model.layers.3.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 314 | `model.language_model.layers.3.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 315 | `model.language_model.layers.3.self_attn.k_norm.weight` | [256] | `BF16` | `model-00001-of-00018.safetensors` |
| 316 | `model.language_model.layers.3.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 317 | `model.language_model.layers.3.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00001-of-00018.safetensors` |
| 318 | `model.language_model.layers.3.self_attn.q_norm.weight` | [256] | `BF16` | `model-00001-of-00018.safetensors` |
| 319 | `model.language_model.layers.3.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 320 | `model.language_model.layers.3.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 321 | `model.language_model.layers.30.input_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 322 | `model.language_model.layers.30.linear_attn.A_log` | [48] | `BF16` | `model-00009-of-00018.safetensors` |
| 323 | `model.language_model.layers.30.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00009-of-00018.safetensors` |
| 324 | `model.language_model.layers.30.linear_attn.dt_bias` | [48] | `BF16` | `model-00009-of-00018.safetensors` |
| 325 | `model.language_model.layers.30.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 326 | `model.language_model.layers.30.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 327 | `model.language_model.layers.30.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 328 | `model.language_model.layers.30.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 329 | `model.language_model.layers.30.linear_attn.norm.weight` | [128] | `BF16` | `model-00009-of-00018.safetensors` |
| 330 | `model.language_model.layers.30.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| 331 | `model.language_model.layers.30.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| 332 | `model.language_model.layers.30.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 333 | `model.language_model.layers.30.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 334 | `model.language_model.layers.30.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 335 | `model.language_model.layers.31.input_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 336 | `model.language_model.layers.31.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00009-of-00018.safetensors` |
| 337 | `model.language_model.layers.31.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 338 | `model.language_model.layers.31.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 339 | `model.language_model.layers.31.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 340 | `model.language_model.layers.31.self_attn.k_norm.weight` | [256] | `BF16` | `model-00009-of-00018.safetensors` |
| 341 | `model.language_model.layers.31.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 342 | `model.language_model.layers.31.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00009-of-00018.safetensors` |
| 343 | `model.language_model.layers.31.self_attn.q_norm.weight` | [256] | `BF16` | `model-00009-of-00018.safetensors` |
| 344 | `model.language_model.layers.31.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 345 | `model.language_model.layers.31.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00009-of-00018.safetensors` |
| 346 | `model.language_model.layers.32.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 347 | `model.language_model.layers.32.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 348 | `model.language_model.layers.32.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| 349 | `model.language_model.layers.32.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 350 | `model.language_model.layers.32.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 351 | `model.language_model.layers.32.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 352 | `model.language_model.layers.32.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 353 | `model.language_model.layers.32.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 354 | `model.language_model.layers.32.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| 355 | `model.language_model.layers.32.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| 356 | `model.language_model.layers.32.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| 357 | `model.language_model.layers.32.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 358 | `model.language_model.layers.32.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 359 | `model.language_model.layers.32.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 360 | `model.language_model.layers.33.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 361 | `model.language_model.layers.33.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 362 | `model.language_model.layers.33.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| 363 | `model.language_model.layers.33.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 364 | `model.language_model.layers.33.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 365 | `model.language_model.layers.33.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 366 | `model.language_model.layers.33.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 367 | `model.language_model.layers.33.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 368 | `model.language_model.layers.33.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| 369 | `model.language_model.layers.33.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| 370 | `model.language_model.layers.33.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| 371 | `model.language_model.layers.33.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 372 | `model.language_model.layers.33.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 373 | `model.language_model.layers.33.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 374 | `model.language_model.layers.34.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 375 | `model.language_model.layers.34.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 376 | `model.language_model.layers.34.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| 377 | `model.language_model.layers.34.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 378 | `model.language_model.layers.34.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 379 | `model.language_model.layers.34.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 380 | `model.language_model.layers.34.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 381 | `model.language_model.layers.34.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 382 | `model.language_model.layers.34.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| 383 | `model.language_model.layers.34.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| 384 | `model.language_model.layers.34.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| 385 | `model.language_model.layers.34.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 386 | `model.language_model.layers.34.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 387 | `model.language_model.layers.34.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 388 | `model.language_model.layers.35.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 389 | `model.language_model.layers.35.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| 390 | `model.language_model.layers.35.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 391 | `model.language_model.layers.35.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 392 | `model.language_model.layers.35.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 393 | `model.language_model.layers.35.self_attn.k_norm.weight` | [256] | `BF16` | `model-00010-of-00018.safetensors` |
| 394 | `model.language_model.layers.35.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 395 | `model.language_model.layers.35.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| 396 | `model.language_model.layers.35.self_attn.q_norm.weight` | [256] | `BF16` | `model-00010-of-00018.safetensors` |
| 397 | `model.language_model.layers.35.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 398 | `model.language_model.layers.35.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 399 | `model.language_model.layers.36.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 400 | `model.language_model.layers.36.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 401 | `model.language_model.layers.36.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| 402 | `model.language_model.layers.36.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 403 | `model.language_model.layers.36.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 404 | `model.language_model.layers.36.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 405 | `model.language_model.layers.36.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 406 | `model.language_model.layers.36.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 407 | `model.language_model.layers.36.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| 408 | `model.language_model.layers.36.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00010-of-00018.safetensors` |
| 409 | `model.language_model.layers.36.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00010-of-00018.safetensors` |
| 410 | `model.language_model.layers.36.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 411 | `model.language_model.layers.36.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 412 | `model.language_model.layers.36.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 413 | `model.language_model.layers.37.input_layernorm.weight` | [5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 414 | `model.language_model.layers.37.linear_attn.A_log` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 415 | `model.language_model.layers.37.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00010-of-00018.safetensors` |
| 416 | `model.language_model.layers.37.linear_attn.dt_bias` | [48] | `BF16` | `model-00010-of-00018.safetensors` |
| 417 | `model.language_model.layers.37.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 418 | `model.language_model.layers.37.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 419 | `model.language_model.layers.37.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 420 | `model.language_model.layers.37.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00010-of-00018.safetensors` |
| 421 | `model.language_model.layers.37.linear_attn.norm.weight` | [128] | `BF16` | `model-00010-of-00018.safetensors` |
| 422 | `model.language_model.layers.37.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| 423 | `model.language_model.layers.37.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| 424 | `model.language_model.layers.37.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 425 | `model.language_model.layers.37.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 426 | `model.language_model.layers.37.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 427 | `model.language_model.layers.38.input_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 428 | `model.language_model.layers.38.linear_attn.A_log` | [48] | `BF16` | `model-00011-of-00018.safetensors` |
| 429 | `model.language_model.layers.38.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00011-of-00018.safetensors` |
| 430 | `model.language_model.layers.38.linear_attn.dt_bias` | [48] | `BF16` | `model-00011-of-00018.safetensors` |
| 431 | `model.language_model.layers.38.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 432 | `model.language_model.layers.38.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 433 | `model.language_model.layers.38.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 434 | `model.language_model.layers.38.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 435 | `model.language_model.layers.38.linear_attn.norm.weight` | [128] | `BF16` | `model-00011-of-00018.safetensors` |
| 436 | `model.language_model.layers.38.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| 437 | `model.language_model.layers.38.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| 438 | `model.language_model.layers.38.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 439 | `model.language_model.layers.38.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 440 | `model.language_model.layers.38.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 441 | `model.language_model.layers.39.input_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 442 | `model.language_model.layers.39.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00011-of-00018.safetensors` |
| 443 | `model.language_model.layers.39.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 444 | `model.language_model.layers.39.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 445 | `model.language_model.layers.39.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 446 | `model.language_model.layers.39.self_attn.k_norm.weight` | [256] | `BF16` | `model-00011-of-00018.safetensors` |
| 447 | `model.language_model.layers.39.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 448 | `model.language_model.layers.39.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00011-of-00018.safetensors` |
| 449 | `model.language_model.layers.39.self_attn.q_norm.weight` | [256] | `BF16` | `model-00011-of-00018.safetensors` |
| 450 | `model.language_model.layers.39.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 451 | `model.language_model.layers.39.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00011-of-00018.safetensors` |
| 452 | `model.language_model.layers.4.input_layernorm.weight` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 453 | `model.language_model.layers.4.linear_attn.A_log` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 454 | `model.language_model.layers.4.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00001-of-00018.safetensors` |
| 455 | `model.language_model.layers.4.linear_attn.dt_bias` | [48] | `BF16` | `model-00001-of-00018.safetensors` |
| 456 | `model.language_model.layers.4.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 457 | `model.language_model.layers.4.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 458 | `model.language_model.layers.4.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 459 | `model.language_model.layers.4.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 460 | `model.language_model.layers.4.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| 461 | `model.language_model.layers.4.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| 462 | `model.language_model.layers.4.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| 463 | `model.language_model.layers.4.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 464 | `model.language_model.layers.4.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 465 | `model.language_model.layers.4.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 466 | `model.language_model.layers.40.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 467 | `model.language_model.layers.40.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 468 | `model.language_model.layers.40.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| 469 | `model.language_model.layers.40.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 470 | `model.language_model.layers.40.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 471 | `model.language_model.layers.40.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 472 | `model.language_model.layers.40.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 473 | `model.language_model.layers.40.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 474 | `model.language_model.layers.40.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| 475 | `model.language_model.layers.40.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| 476 | `model.language_model.layers.40.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| 477 | `model.language_model.layers.40.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 478 | `model.language_model.layers.40.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 479 | `model.language_model.layers.40.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 480 | `model.language_model.layers.41.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 481 | `model.language_model.layers.41.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 482 | `model.language_model.layers.41.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| 483 | `model.language_model.layers.41.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 484 | `model.language_model.layers.41.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 485 | `model.language_model.layers.41.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 486 | `model.language_model.layers.41.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 487 | `model.language_model.layers.41.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 488 | `model.language_model.layers.41.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| 489 | `model.language_model.layers.41.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| 490 | `model.language_model.layers.41.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| 491 | `model.language_model.layers.41.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 492 | `model.language_model.layers.41.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 493 | `model.language_model.layers.41.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 494 | `model.language_model.layers.42.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 495 | `model.language_model.layers.42.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 496 | `model.language_model.layers.42.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| 497 | `model.language_model.layers.42.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 498 | `model.language_model.layers.42.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 499 | `model.language_model.layers.42.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 500 | `model.language_model.layers.42.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 501 | `model.language_model.layers.42.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 502 | `model.language_model.layers.42.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| 503 | `model.language_model.layers.42.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| 504 | `model.language_model.layers.42.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| 505 | `model.language_model.layers.42.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 506 | `model.language_model.layers.42.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 507 | `model.language_model.layers.42.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 508 | `model.language_model.layers.43.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 509 | `model.language_model.layers.43.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| 510 | `model.language_model.layers.43.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 511 | `model.language_model.layers.43.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 512 | `model.language_model.layers.43.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 513 | `model.language_model.layers.43.self_attn.k_norm.weight` | [256] | `BF16` | `model-00012-of-00018.safetensors` |
| 514 | `model.language_model.layers.43.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 515 | `model.language_model.layers.43.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| 516 | `model.language_model.layers.43.self_attn.q_norm.weight` | [256] | `BF16` | `model-00012-of-00018.safetensors` |
| 517 | `model.language_model.layers.43.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 518 | `model.language_model.layers.43.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 519 | `model.language_model.layers.44.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 520 | `model.language_model.layers.44.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 521 | `model.language_model.layers.44.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| 522 | `model.language_model.layers.44.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 523 | `model.language_model.layers.44.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 524 | `model.language_model.layers.44.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 525 | `model.language_model.layers.44.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 526 | `model.language_model.layers.44.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 527 | `model.language_model.layers.44.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| 528 | `model.language_model.layers.44.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00012-of-00018.safetensors` |
| 529 | `model.language_model.layers.44.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00012-of-00018.safetensors` |
| 530 | `model.language_model.layers.44.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 531 | `model.language_model.layers.44.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 532 | `model.language_model.layers.44.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 533 | `model.language_model.layers.45.input_layernorm.weight` | [5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 534 | `model.language_model.layers.45.linear_attn.A_log` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 535 | `model.language_model.layers.45.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00012-of-00018.safetensors` |
| 536 | `model.language_model.layers.45.linear_attn.dt_bias` | [48] | `BF16` | `model-00012-of-00018.safetensors` |
| 537 | `model.language_model.layers.45.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 538 | `model.language_model.layers.45.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 539 | `model.language_model.layers.45.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 540 | `model.language_model.layers.45.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00012-of-00018.safetensors` |
| 541 | `model.language_model.layers.45.linear_attn.norm.weight` | [128] | `BF16` | `model-00012-of-00018.safetensors` |
| 542 | `model.language_model.layers.45.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| 543 | `model.language_model.layers.45.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| 544 | `model.language_model.layers.45.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 545 | `model.language_model.layers.45.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 546 | `model.language_model.layers.45.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 547 | `model.language_model.layers.46.input_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 548 | `model.language_model.layers.46.linear_attn.A_log` | [48] | `BF16` | `model-00013-of-00018.safetensors` |
| 549 | `model.language_model.layers.46.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00013-of-00018.safetensors` |
| 550 | `model.language_model.layers.46.linear_attn.dt_bias` | [48] | `BF16` | `model-00013-of-00018.safetensors` |
| 551 | `model.language_model.layers.46.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 552 | `model.language_model.layers.46.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 553 | `model.language_model.layers.46.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 554 | `model.language_model.layers.46.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 555 | `model.language_model.layers.46.linear_attn.norm.weight` | [128] | `BF16` | `model-00013-of-00018.safetensors` |
| 556 | `model.language_model.layers.46.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| 557 | `model.language_model.layers.46.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| 558 | `model.language_model.layers.46.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 559 | `model.language_model.layers.46.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 560 | `model.language_model.layers.46.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 561 | `model.language_model.layers.47.input_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 562 | `model.language_model.layers.47.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00013-of-00018.safetensors` |
| 563 | `model.language_model.layers.47.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 564 | `model.language_model.layers.47.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 565 | `model.language_model.layers.47.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 566 | `model.language_model.layers.47.self_attn.k_norm.weight` | [256] | `BF16` | `model-00013-of-00018.safetensors` |
| 567 | `model.language_model.layers.47.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 568 | `model.language_model.layers.47.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00013-of-00018.safetensors` |
| 569 | `model.language_model.layers.47.self_attn.q_norm.weight` | [256] | `BF16` | `model-00013-of-00018.safetensors` |
| 570 | `model.language_model.layers.47.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 571 | `model.language_model.layers.47.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00013-of-00018.safetensors` |
| 572 | `model.language_model.layers.48.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 573 | `model.language_model.layers.48.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 574 | `model.language_model.layers.48.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| 575 | `model.language_model.layers.48.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 576 | `model.language_model.layers.48.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 577 | `model.language_model.layers.48.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 578 | `model.language_model.layers.48.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 579 | `model.language_model.layers.48.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 580 | `model.language_model.layers.48.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| 581 | `model.language_model.layers.48.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| 582 | `model.language_model.layers.48.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| 583 | `model.language_model.layers.48.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 584 | `model.language_model.layers.48.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 585 | `model.language_model.layers.48.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 586 | `model.language_model.layers.49.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 587 | `model.language_model.layers.49.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 588 | `model.language_model.layers.49.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| 589 | `model.language_model.layers.49.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 590 | `model.language_model.layers.49.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 591 | `model.language_model.layers.49.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 592 | `model.language_model.layers.49.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 593 | `model.language_model.layers.49.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 594 | `model.language_model.layers.49.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| 595 | `model.language_model.layers.49.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| 596 | `model.language_model.layers.49.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| 597 | `model.language_model.layers.49.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 598 | `model.language_model.layers.49.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 599 | `model.language_model.layers.49.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 600 | `model.language_model.layers.5.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 601 | `model.language_model.layers.5.linear_attn.A_log` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| 602 | `model.language_model.layers.5.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00002-of-00018.safetensors` |
| 603 | `model.language_model.layers.5.linear_attn.dt_bias` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| 604 | `model.language_model.layers.5.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 605 | `model.language_model.layers.5.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 606 | `model.language_model.layers.5.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 607 | `model.language_model.layers.5.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 608 | `model.language_model.layers.5.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| 609 | `model.language_model.layers.5.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| 610 | `model.language_model.layers.5.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| 611 | `model.language_model.layers.5.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 612 | `model.language_model.layers.5.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 613 | `model.language_model.layers.5.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 614 | `model.language_model.layers.50.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 615 | `model.language_model.layers.50.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 616 | `model.language_model.layers.50.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| 617 | `model.language_model.layers.50.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 618 | `model.language_model.layers.50.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 619 | `model.language_model.layers.50.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 620 | `model.language_model.layers.50.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 621 | `model.language_model.layers.50.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 622 | `model.language_model.layers.50.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| 623 | `model.language_model.layers.50.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| 624 | `model.language_model.layers.50.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| 625 | `model.language_model.layers.50.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 626 | `model.language_model.layers.50.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 627 | `model.language_model.layers.50.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 628 | `model.language_model.layers.51.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 629 | `model.language_model.layers.51.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| 630 | `model.language_model.layers.51.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 631 | `model.language_model.layers.51.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 632 | `model.language_model.layers.51.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 633 | `model.language_model.layers.51.self_attn.k_norm.weight` | [256] | `BF16` | `model-00014-of-00018.safetensors` |
| 634 | `model.language_model.layers.51.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 635 | `model.language_model.layers.51.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| 636 | `model.language_model.layers.51.self_attn.q_norm.weight` | [256] | `BF16` | `model-00014-of-00018.safetensors` |
| 637 | `model.language_model.layers.51.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 638 | `model.language_model.layers.51.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 639 | `model.language_model.layers.52.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 640 | `model.language_model.layers.52.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 641 | `model.language_model.layers.52.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| 642 | `model.language_model.layers.52.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 643 | `model.language_model.layers.52.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 644 | `model.language_model.layers.52.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 645 | `model.language_model.layers.52.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 646 | `model.language_model.layers.52.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 647 | `model.language_model.layers.52.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| 648 | `model.language_model.layers.52.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00014-of-00018.safetensors` |
| 649 | `model.language_model.layers.52.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00014-of-00018.safetensors` |
| 650 | `model.language_model.layers.52.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 651 | `model.language_model.layers.52.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 652 | `model.language_model.layers.52.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 653 | `model.language_model.layers.53.input_layernorm.weight` | [5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 654 | `model.language_model.layers.53.linear_attn.A_log` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 655 | `model.language_model.layers.53.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00014-of-00018.safetensors` |
| 656 | `model.language_model.layers.53.linear_attn.dt_bias` | [48] | `BF16` | `model-00014-of-00018.safetensors` |
| 657 | `model.language_model.layers.53.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 658 | `model.language_model.layers.53.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 659 | `model.language_model.layers.53.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 660 | `model.language_model.layers.53.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00014-of-00018.safetensors` |
| 661 | `model.language_model.layers.53.linear_attn.norm.weight` | [128] | `BF16` | `model-00014-of-00018.safetensors` |
| 662 | `model.language_model.layers.53.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| 663 | `model.language_model.layers.53.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| 664 | `model.language_model.layers.53.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 665 | `model.language_model.layers.53.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 666 | `model.language_model.layers.53.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 667 | `model.language_model.layers.54.input_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 668 | `model.language_model.layers.54.linear_attn.A_log` | [48] | `BF16` | `model-00015-of-00018.safetensors` |
| 669 | `model.language_model.layers.54.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00015-of-00018.safetensors` |
| 670 | `model.language_model.layers.54.linear_attn.dt_bias` | [48] | `BF16` | `model-00015-of-00018.safetensors` |
| 671 | `model.language_model.layers.54.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 672 | `model.language_model.layers.54.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 673 | `model.language_model.layers.54.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 674 | `model.language_model.layers.54.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 675 | `model.language_model.layers.54.linear_attn.norm.weight` | [128] | `BF16` | `model-00015-of-00018.safetensors` |
| 676 | `model.language_model.layers.54.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| 677 | `model.language_model.layers.54.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| 678 | `model.language_model.layers.54.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 679 | `model.language_model.layers.54.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 680 | `model.language_model.layers.54.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 681 | `model.language_model.layers.55.input_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 682 | `model.language_model.layers.55.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00015-of-00018.safetensors` |
| 683 | `model.language_model.layers.55.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 684 | `model.language_model.layers.55.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 685 | `model.language_model.layers.55.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 686 | `model.language_model.layers.55.self_attn.k_norm.weight` | [256] | `BF16` | `model-00015-of-00018.safetensors` |
| 687 | `model.language_model.layers.55.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 688 | `model.language_model.layers.55.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00015-of-00018.safetensors` |
| 689 | `model.language_model.layers.55.self_attn.q_norm.weight` | [256] | `BF16` | `model-00015-of-00018.safetensors` |
| 690 | `model.language_model.layers.55.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 691 | `model.language_model.layers.55.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00015-of-00018.safetensors` |
| 692 | `model.language_model.layers.56.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 693 | `model.language_model.layers.56.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 694 | `model.language_model.layers.56.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| 695 | `model.language_model.layers.56.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 696 | `model.language_model.layers.56.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 697 | `model.language_model.layers.56.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 698 | `model.language_model.layers.56.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 699 | `model.language_model.layers.56.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 700 | `model.language_model.layers.56.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| 701 | `model.language_model.layers.56.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| 702 | `model.language_model.layers.56.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| 703 | `model.language_model.layers.56.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 704 | `model.language_model.layers.56.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 705 | `model.language_model.layers.56.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 706 | `model.language_model.layers.57.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 707 | `model.language_model.layers.57.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 708 | `model.language_model.layers.57.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| 709 | `model.language_model.layers.57.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 710 | `model.language_model.layers.57.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 711 | `model.language_model.layers.57.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 712 | `model.language_model.layers.57.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 713 | `model.language_model.layers.57.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 714 | `model.language_model.layers.57.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| 715 | `model.language_model.layers.57.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| 716 | `model.language_model.layers.57.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| 717 | `model.language_model.layers.57.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 718 | `model.language_model.layers.57.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 719 | `model.language_model.layers.57.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 720 | `model.language_model.layers.58.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 721 | `model.language_model.layers.58.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 722 | `model.language_model.layers.58.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| 723 | `model.language_model.layers.58.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 724 | `model.language_model.layers.58.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 725 | `model.language_model.layers.58.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 726 | `model.language_model.layers.58.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 727 | `model.language_model.layers.58.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 728 | `model.language_model.layers.58.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| 729 | `model.language_model.layers.58.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| 730 | `model.language_model.layers.58.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| 731 | `model.language_model.layers.58.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 732 | `model.language_model.layers.58.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 733 | `model.language_model.layers.58.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 734 | `model.language_model.layers.59.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 735 | `model.language_model.layers.59.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| 736 | `model.language_model.layers.59.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 737 | `model.language_model.layers.59.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 738 | `model.language_model.layers.59.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 739 | `model.language_model.layers.59.self_attn.k_norm.weight` | [256] | `BF16` | `model-00016-of-00018.safetensors` |
| 740 | `model.language_model.layers.59.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 741 | `model.language_model.layers.59.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| 742 | `model.language_model.layers.59.self_attn.q_norm.weight` | [256] | `BF16` | `model-00016-of-00018.safetensors` |
| 743 | `model.language_model.layers.59.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 744 | `model.language_model.layers.59.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 745 | `model.language_model.layers.6.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 746 | `model.language_model.layers.6.linear_attn.A_log` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| 747 | `model.language_model.layers.6.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00002-of-00018.safetensors` |
| 748 | `model.language_model.layers.6.linear_attn.dt_bias` | [48] | `BF16` | `model-00002-of-00018.safetensors` |
| 749 | `model.language_model.layers.6.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 750 | `model.language_model.layers.6.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 751 | `model.language_model.layers.6.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 752 | `model.language_model.layers.6.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 753 | `model.language_model.layers.6.linear_attn.norm.weight` | [128] | `BF16` | `model-00002-of-00018.safetensors` |
| 754 | `model.language_model.layers.6.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| 755 | `model.language_model.layers.6.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| 756 | `model.language_model.layers.6.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 757 | `model.language_model.layers.6.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 758 | `model.language_model.layers.6.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 759 | `model.language_model.layers.60.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 760 | `model.language_model.layers.60.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 761 | `model.language_model.layers.60.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| 762 | `model.language_model.layers.60.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 763 | `model.language_model.layers.60.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 764 | `model.language_model.layers.60.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 765 | `model.language_model.layers.60.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 766 | `model.language_model.layers.60.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 767 | `model.language_model.layers.60.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| 768 | `model.language_model.layers.60.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00016-of-00018.safetensors` |
| 769 | `model.language_model.layers.60.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00016-of-00018.safetensors` |
| 770 | `model.language_model.layers.60.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 771 | `model.language_model.layers.60.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 772 | `model.language_model.layers.60.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 773 | `model.language_model.layers.61.input_layernorm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 774 | `model.language_model.layers.61.linear_attn.A_log` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 775 | `model.language_model.layers.61.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00016-of-00018.safetensors` |
| 776 | `model.language_model.layers.61.linear_attn.dt_bias` | [48] | `BF16` | `model-00016-of-00018.safetensors` |
| 777 | `model.language_model.layers.61.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 778 | `model.language_model.layers.61.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 779 | `model.language_model.layers.61.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 780 | `model.language_model.layers.61.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 781 | `model.language_model.layers.61.linear_attn.norm.weight` | [128] | `BF16` | `model-00016-of-00018.safetensors` |
| 782 | `model.language_model.layers.61.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| 783 | `model.language_model.layers.61.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| 784 | `model.language_model.layers.61.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 785 | `model.language_model.layers.61.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 786 | `model.language_model.layers.61.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 787 | `model.language_model.layers.62.input_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 788 | `model.language_model.layers.62.linear_attn.A_log` | [48] | `BF16` | `model-00017-of-00018.safetensors` |
| 789 | `model.language_model.layers.62.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00017-of-00018.safetensors` |
| 790 | `model.language_model.layers.62.linear_attn.dt_bias` | [48] | `BF16` | `model-00017-of-00018.safetensors` |
| 791 | `model.language_model.layers.62.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 792 | `model.language_model.layers.62.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 793 | `model.language_model.layers.62.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 794 | `model.language_model.layers.62.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 795 | `model.language_model.layers.62.linear_attn.norm.weight` | [128] | `BF16` | `model-00017-of-00018.safetensors` |
| 796 | `model.language_model.layers.62.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| 797 | `model.language_model.layers.62.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| 798 | `model.language_model.layers.62.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 799 | `model.language_model.layers.62.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 800 | `model.language_model.layers.62.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 801 | `model.language_model.layers.63.input_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 802 | `model.language_model.layers.63.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00017-of-00018.safetensors` |
| 803 | `model.language_model.layers.63.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 804 | `model.language_model.layers.63.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 805 | `model.language_model.layers.63.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 806 | `model.language_model.layers.63.self_attn.k_norm.weight` | [256] | `BF16` | `model-00017-of-00018.safetensors` |
| 807 | `model.language_model.layers.63.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 808 | `model.language_model.layers.63.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00017-of-00018.safetensors` |
| 809 | `model.language_model.layers.63.self_attn.q_norm.weight` | [256] | `BF16` | `model-00017-of-00018.safetensors` |
| 810 | `model.language_model.layers.63.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 811 | `model.language_model.layers.63.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00017-of-00018.safetensors` |
| 812 | `model.language_model.layers.7.input_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 813 | `model.language_model.layers.7.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00002-of-00018.safetensors` |
| 814 | `model.language_model.layers.7.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 815 | `model.language_model.layers.7.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 816 | `model.language_model.layers.7.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 817 | `model.language_model.layers.7.self_attn.k_norm.weight` | [256] | `BF16` | `model-00002-of-00018.safetensors` |
| 818 | `model.language_model.layers.7.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 819 | `model.language_model.layers.7.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00002-of-00018.safetensors` |
| 820 | `model.language_model.layers.7.self_attn.q_norm.weight` | [256] | `BF16` | `model-00002-of-00018.safetensors` |
| 821 | `model.language_model.layers.7.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 822 | `model.language_model.layers.7.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00002-of-00018.safetensors` |
| 823 | `model.language_model.layers.8.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 824 | `model.language_model.layers.8.linear_attn.A_log` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| 825 | `model.language_model.layers.8.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00005-of-00018.safetensors` |
| 826 | `model.language_model.layers.8.linear_attn.dt_bias` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| 827 | `model.language_model.layers.8.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 828 | `model.language_model.layers.8.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 829 | `model.language_model.layers.8.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 830 | `model.language_model.layers.8.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 831 | `model.language_model.layers.8.linear_attn.norm.weight` | [128] | `BF16` | `model-00005-of-00018.safetensors` |
| 832 | `model.language_model.layers.8.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |
| 833 | `model.language_model.layers.8.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| 834 | `model.language_model.layers.8.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 835 | `model.language_model.layers.8.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 836 | `model.language_model.layers.8.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 837 | `model.language_model.layers.9.input_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 838 | `model.language_model.layers.9.linear_attn.A_log` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| 839 | `model.language_model.layers.9.linear_attn.conv1d.weight` | [10240, 1, 4] | `BF16` | `model-00005-of-00018.safetensors` |
| 840 | `model.language_model.layers.9.linear_attn.dt_bias` | [48] | `BF16` | `model-00005-of-00018.safetensors` |
| 841 | `model.language_model.layers.9.linear_attn.in_proj_a.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 842 | `model.language_model.layers.9.linear_attn.in_proj_b.weight` | [48, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 843 | `model.language_model.layers.9.linear_attn.in_proj_qkv.weight` | [10240, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 844 | `model.language_model.layers.9.linear_attn.in_proj_z.weight` | [6144, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 845 | `model.language_model.layers.9.linear_attn.norm.weight` | [128] | `BF16` | `model-00005-of-00018.safetensors` |
| 846 | `model.language_model.layers.9.linear_attn.out_proj.weight` | [5120, 6144] | `BF16` | `model-00005-of-00018.safetensors` |
| 847 | `model.language_model.layers.9.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00005-of-00018.safetensors` |
| 848 | `model.language_model.layers.9.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 849 | `model.language_model.layers.9.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 850 | `model.language_model.layers.9.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00005-of-00018.safetensors` |
| 851 | `model.language_model.norm.weight` | [5120] | `BF16` | `model-00016-of-00018.safetensors` |
| 852 | `model.visual.blocks.0.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 853 | `model.visual.blocks.0.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 854 | `model.visual.blocks.0.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 855 | `model.visual.blocks.0.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 856 | `model.visual.blocks.0.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 857 | `model.visual.blocks.0.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 858 | `model.visual.blocks.0.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 859 | `model.visual.blocks.0.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 860 | `model.visual.blocks.0.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 861 | `model.visual.blocks.0.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 862 | `model.visual.blocks.0.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 863 | `model.visual.blocks.0.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 864 | `model.visual.blocks.1.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 865 | `model.visual.blocks.1.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 866 | `model.visual.blocks.1.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 867 | `model.visual.blocks.1.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 868 | `model.visual.blocks.1.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 869 | `model.visual.blocks.1.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 870 | `model.visual.blocks.1.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 871 | `model.visual.blocks.1.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 872 | `model.visual.blocks.1.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 873 | `model.visual.blocks.1.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 874 | `model.visual.blocks.1.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 875 | `model.visual.blocks.1.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 876 | `model.visual.blocks.10.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 877 | `model.visual.blocks.10.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 878 | `model.visual.blocks.10.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 879 | `model.visual.blocks.10.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 880 | `model.visual.blocks.10.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 881 | `model.visual.blocks.10.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 882 | `model.visual.blocks.10.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 883 | `model.visual.blocks.10.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 884 | `model.visual.blocks.10.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 885 | `model.visual.blocks.10.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 886 | `model.visual.blocks.10.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 887 | `model.visual.blocks.10.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 888 | `model.visual.blocks.11.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 889 | `model.visual.blocks.11.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 890 | `model.visual.blocks.11.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 891 | `model.visual.blocks.11.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 892 | `model.visual.blocks.11.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 893 | `model.visual.blocks.11.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 894 | `model.visual.blocks.11.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 895 | `model.visual.blocks.11.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 896 | `model.visual.blocks.11.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 897 | `model.visual.blocks.11.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 898 | `model.visual.blocks.11.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 899 | `model.visual.blocks.11.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 900 | `model.visual.blocks.12.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 901 | `model.visual.blocks.12.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 902 | `model.visual.blocks.12.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 903 | `model.visual.blocks.12.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 904 | `model.visual.blocks.12.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 905 | `model.visual.blocks.12.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 906 | `model.visual.blocks.12.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 907 | `model.visual.blocks.12.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 908 | `model.visual.blocks.12.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 909 | `model.visual.blocks.12.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 910 | `model.visual.blocks.12.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 911 | `model.visual.blocks.12.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 912 | `model.visual.blocks.13.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 913 | `model.visual.blocks.13.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 914 | `model.visual.blocks.13.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 915 | `model.visual.blocks.13.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 916 | `model.visual.blocks.13.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 917 | `model.visual.blocks.13.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 918 | `model.visual.blocks.13.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 919 | `model.visual.blocks.13.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 920 | `model.visual.blocks.13.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 921 | `model.visual.blocks.13.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 922 | `model.visual.blocks.13.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 923 | `model.visual.blocks.13.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 924 | `model.visual.blocks.14.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 925 | `model.visual.blocks.14.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 926 | `model.visual.blocks.14.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 927 | `model.visual.blocks.14.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 928 | `model.visual.blocks.14.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 929 | `model.visual.blocks.14.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 930 | `model.visual.blocks.14.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 931 | `model.visual.blocks.14.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 932 | `model.visual.blocks.14.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 933 | `model.visual.blocks.14.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 934 | `model.visual.blocks.14.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 935 | `model.visual.blocks.14.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 936 | `model.visual.blocks.15.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 937 | `model.visual.blocks.15.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 938 | `model.visual.blocks.15.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 939 | `model.visual.blocks.15.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 940 | `model.visual.blocks.15.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 941 | `model.visual.blocks.15.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 942 | `model.visual.blocks.15.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 943 | `model.visual.blocks.15.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 944 | `model.visual.blocks.15.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 945 | `model.visual.blocks.15.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 946 | `model.visual.blocks.15.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 947 | `model.visual.blocks.15.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 948 | `model.visual.blocks.16.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 949 | `model.visual.blocks.16.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 950 | `model.visual.blocks.16.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 951 | `model.visual.blocks.16.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 952 | `model.visual.blocks.16.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 953 | `model.visual.blocks.16.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 954 | `model.visual.blocks.16.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 955 | `model.visual.blocks.16.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 956 | `model.visual.blocks.16.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 957 | `model.visual.blocks.16.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 958 | `model.visual.blocks.16.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 959 | `model.visual.blocks.16.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 960 | `model.visual.blocks.17.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 961 | `model.visual.blocks.17.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 962 | `model.visual.blocks.17.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 963 | `model.visual.blocks.17.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 964 | `model.visual.blocks.17.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 965 | `model.visual.blocks.17.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 966 | `model.visual.blocks.17.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 967 | `model.visual.blocks.17.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 968 | `model.visual.blocks.17.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 969 | `model.visual.blocks.17.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 970 | `model.visual.blocks.17.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 971 | `model.visual.blocks.17.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 972 | `model.visual.blocks.18.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 973 | `model.visual.blocks.18.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 974 | `model.visual.blocks.18.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 975 | `model.visual.blocks.18.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 976 | `model.visual.blocks.18.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 977 | `model.visual.blocks.18.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 978 | `model.visual.blocks.18.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 979 | `model.visual.blocks.18.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 980 | `model.visual.blocks.18.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 981 | `model.visual.blocks.18.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 982 | `model.visual.blocks.18.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 983 | `model.visual.blocks.18.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 984 | `model.visual.blocks.19.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 985 | `model.visual.blocks.19.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 986 | `model.visual.blocks.19.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 987 | `model.visual.blocks.19.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 988 | `model.visual.blocks.19.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 989 | `model.visual.blocks.19.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 990 | `model.visual.blocks.19.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 991 | `model.visual.blocks.19.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 992 | `model.visual.blocks.19.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 993 | `model.visual.blocks.19.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 994 | `model.visual.blocks.19.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 995 | `model.visual.blocks.19.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 996 | `model.visual.blocks.2.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 997 | `model.visual.blocks.2.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 998 | `model.visual.blocks.2.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 999 | `model.visual.blocks.2.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1000 | `model.visual.blocks.2.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1001 | `model.visual.blocks.2.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1002 | `model.visual.blocks.2.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1003 | `model.visual.blocks.2.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1004 | `model.visual.blocks.2.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1005 | `model.visual.blocks.2.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1006 | `model.visual.blocks.2.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1007 | `model.visual.blocks.2.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1008 | `model.visual.blocks.20.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1009 | `model.visual.blocks.20.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1010 | `model.visual.blocks.20.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1011 | `model.visual.blocks.20.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1012 | `model.visual.blocks.20.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1013 | `model.visual.blocks.20.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1014 | `model.visual.blocks.20.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1015 | `model.visual.blocks.20.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1016 | `model.visual.blocks.20.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1017 | `model.visual.blocks.20.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1018 | `model.visual.blocks.20.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1019 | `model.visual.blocks.20.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1020 | `model.visual.blocks.21.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1021 | `model.visual.blocks.21.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1022 | `model.visual.blocks.21.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1023 | `model.visual.blocks.21.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1024 | `model.visual.blocks.21.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1025 | `model.visual.blocks.21.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1026 | `model.visual.blocks.21.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1027 | `model.visual.blocks.21.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1028 | `model.visual.blocks.21.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1029 | `model.visual.blocks.21.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1030 | `model.visual.blocks.21.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1031 | `model.visual.blocks.21.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1032 | `model.visual.blocks.22.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1033 | `model.visual.blocks.22.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1034 | `model.visual.blocks.22.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1035 | `model.visual.blocks.22.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1036 | `model.visual.blocks.22.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1037 | `model.visual.blocks.22.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1038 | `model.visual.blocks.22.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1039 | `model.visual.blocks.22.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1040 | `model.visual.blocks.22.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1041 | `model.visual.blocks.22.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1042 | `model.visual.blocks.22.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1043 | `model.visual.blocks.22.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1044 | `model.visual.blocks.23.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1045 | `model.visual.blocks.23.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1046 | `model.visual.blocks.23.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1047 | `model.visual.blocks.23.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1048 | `model.visual.blocks.23.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1049 | `model.visual.blocks.23.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1050 | `model.visual.blocks.23.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1051 | `model.visual.blocks.23.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1052 | `model.visual.blocks.23.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1053 | `model.visual.blocks.23.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1054 | `model.visual.blocks.23.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1055 | `model.visual.blocks.23.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1056 | `model.visual.blocks.24.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1057 | `model.visual.blocks.24.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1058 | `model.visual.blocks.24.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1059 | `model.visual.blocks.24.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1060 | `model.visual.blocks.24.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1061 | `model.visual.blocks.24.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1062 | `model.visual.blocks.24.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1063 | `model.visual.blocks.24.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1064 | `model.visual.blocks.24.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1065 | `model.visual.blocks.24.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1066 | `model.visual.blocks.24.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1067 | `model.visual.blocks.24.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1068 | `model.visual.blocks.25.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1069 | `model.visual.blocks.25.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1070 | `model.visual.blocks.25.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1071 | `model.visual.blocks.25.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1072 | `model.visual.blocks.25.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1073 | `model.visual.blocks.25.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1074 | `model.visual.blocks.25.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1075 | `model.visual.blocks.25.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1076 | `model.visual.blocks.25.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1077 | `model.visual.blocks.25.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1078 | `model.visual.blocks.25.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1079 | `model.visual.blocks.25.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1080 | `model.visual.blocks.26.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1081 | `model.visual.blocks.26.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1082 | `model.visual.blocks.26.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1083 | `model.visual.blocks.26.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1084 | `model.visual.blocks.26.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1085 | `model.visual.blocks.26.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1086 | `model.visual.blocks.26.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1087 | `model.visual.blocks.26.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1088 | `model.visual.blocks.26.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1089 | `model.visual.blocks.26.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1090 | `model.visual.blocks.26.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1091 | `model.visual.blocks.26.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1092 | `model.visual.blocks.3.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1093 | `model.visual.blocks.3.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1094 | `model.visual.blocks.3.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1095 | `model.visual.blocks.3.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1096 | `model.visual.blocks.3.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1097 | `model.visual.blocks.3.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1098 | `model.visual.blocks.3.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1099 | `model.visual.blocks.3.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1100 | `model.visual.blocks.3.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1101 | `model.visual.blocks.3.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1102 | `model.visual.blocks.3.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1103 | `model.visual.blocks.3.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1104 | `model.visual.blocks.4.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1105 | `model.visual.blocks.4.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1106 | `model.visual.blocks.4.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1107 | `model.visual.blocks.4.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1108 | `model.visual.blocks.4.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1109 | `model.visual.blocks.4.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1110 | `model.visual.blocks.4.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1111 | `model.visual.blocks.4.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1112 | `model.visual.blocks.4.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1113 | `model.visual.blocks.4.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1114 | `model.visual.blocks.4.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1115 | `model.visual.blocks.4.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1116 | `model.visual.blocks.5.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1117 | `model.visual.blocks.5.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1118 | `model.visual.blocks.5.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1119 | `model.visual.blocks.5.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1120 | `model.visual.blocks.5.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1121 | `model.visual.blocks.5.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1122 | `model.visual.blocks.5.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1123 | `model.visual.blocks.5.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1124 | `model.visual.blocks.5.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1125 | `model.visual.blocks.5.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1126 | `model.visual.blocks.5.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1127 | `model.visual.blocks.5.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1128 | `model.visual.blocks.6.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1129 | `model.visual.blocks.6.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1130 | `model.visual.blocks.6.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1131 | `model.visual.blocks.6.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1132 | `model.visual.blocks.6.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1133 | `model.visual.blocks.6.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1134 | `model.visual.blocks.6.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1135 | `model.visual.blocks.6.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1136 | `model.visual.blocks.6.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1137 | `model.visual.blocks.6.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1138 | `model.visual.blocks.6.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1139 | `model.visual.blocks.6.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1140 | `model.visual.blocks.7.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1141 | `model.visual.blocks.7.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1142 | `model.visual.blocks.7.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1143 | `model.visual.blocks.7.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1144 | `model.visual.blocks.7.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1145 | `model.visual.blocks.7.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1146 | `model.visual.blocks.7.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1147 | `model.visual.blocks.7.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1148 | `model.visual.blocks.7.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1149 | `model.visual.blocks.7.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1150 | `model.visual.blocks.7.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1151 | `model.visual.blocks.7.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1152 | `model.visual.blocks.8.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1153 | `model.visual.blocks.8.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1154 | `model.visual.blocks.8.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1155 | `model.visual.blocks.8.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1156 | `model.visual.blocks.8.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1157 | `model.visual.blocks.8.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1158 | `model.visual.blocks.8.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1159 | `model.visual.blocks.8.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1160 | `model.visual.blocks.8.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1161 | `model.visual.blocks.8.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1162 | `model.visual.blocks.8.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1163 | `model.visual.blocks.8.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1164 | `model.visual.blocks.9.attn.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1165 | `model.visual.blocks.9.attn.proj.weight` | [1152, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1166 | `model.visual.blocks.9.attn.qkv.bias` | [3456] | `BF16` | `model-00001-of-00018.safetensors` |
| 1167 | `model.visual.blocks.9.attn.qkv.weight` | [3456, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1168 | `model.visual.blocks.9.mlp.linear_fc1.bias` | [4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1169 | `model.visual.blocks.9.mlp.linear_fc1.weight` | [4304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1170 | `model.visual.blocks.9.mlp.linear_fc2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1171 | `model.visual.blocks.9.mlp.linear_fc2.weight` | [1152, 4304] | `BF16` | `model-00001-of-00018.safetensors` |
| 1172 | `model.visual.blocks.9.norm1.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1173 | `model.visual.blocks.9.norm1.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1174 | `model.visual.blocks.9.norm2.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1175 | `model.visual.blocks.9.norm2.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1176 | `model.visual.merger.linear_fc1.bias` | [4608] | `BF16` | `model-00001-of-00018.safetensors` |
| 1177 | `model.visual.merger.linear_fc1.weight` | [4608, 4608] | `BF16` | `model-00001-of-00018.safetensors` |
| 1178 | `model.visual.merger.linear_fc2.bias` | [5120] | `BF16` | `model-00001-of-00018.safetensors` |
| 1179 | `model.visual.merger.linear_fc2.weight` | [5120, 4608] | `BF16` | `model-00001-of-00018.safetensors` |
| 1180 | `model.visual.merger.norm.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1181 | `model.visual.merger.norm.weight` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1182 | `model.visual.patch_embed.proj.bias` | [1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1183 | `model.visual.patch_embed.proj.weight` | [1152, 3, 2, 16, 16] | `BF16` | `model-00001-of-00018.safetensors` |
| 1184 | `model.visual.pos_embed.weight` | [2304, 1152] | `BF16` | `model-00001-of-00018.safetensors` |
| 1185 | `mtp.fc.weight` | [5120, 10240] | `BF16` | `model-00018-of-00018.safetensors` |
| 1186 | `mtp.layers.0.input_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1187 | `mtp.layers.0.mlp.down_proj.weight` | [5120, 17408] | `BF16` | `model-00018-of-00018.safetensors` |
| 1188 | `mtp.layers.0.mlp.gate_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1189 | `mtp.layers.0.mlp.up_proj.weight` | [17408, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1190 | `mtp.layers.0.post_attention_layernorm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1191 | `mtp.layers.0.self_attn.k_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| 1192 | `mtp.layers.0.self_attn.k_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1193 | `mtp.layers.0.self_attn.o_proj.weight` | [5120, 6144] | `BF16` | `model-00018-of-00018.safetensors` |
| 1194 | `mtp.layers.0.self_attn.q_norm.weight` | [256] | `BF16` | `model-00018-of-00018.safetensors` |
| 1195 | `mtp.layers.0.self_attn.q_proj.weight` | [12288, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1196 | `mtp.layers.0.self_attn.v_proj.weight` | [1024, 5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1197 | `mtp.norm.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1198 | `mtp.pre_fc_norm_embedding.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |
| 1199 | `mtp.pre_fc_norm_hidden.weight` | [5120] | `BF16` | `model-00018-of-00018.safetensors` |

---

## 14. Cross-Reference to Existing Research Documents

This authoritative header metadata confirms the tensor shapes previously
(SET0-T15) classified as "DERIVED FINDING" — they are now directly verified
from the checkpoint headers:

| Tensor Family | SET0-T15 Status | SET0-T16 Status |
|---|---|---|
| Global language tensors | VERIFIED FACT | CONFIRMED from headers |
| Language MLP | VERIFIED FACT | CONFIRMED from headers |
| Full-attention | VERIFIED FACT | CONFIRMED from headers |
| Linear-attention | VERIFIED FACT | CONFIRMED from headers |
| Language normalization | VERIFIED FACT | CONFIRMED from headers |
| Vision | VERIFIED FACT | CONFIRMED from headers |
| MTP tensors presence | VERIFIED (15 tensors) | CONFIRMED from headers |
| MTP exact shapes | DERIVED FINDING | **CONFIRMED from headers** |
| Exact per-tensor dtype (`BF16`) | UNKNOWN | **CONFIRMED from headers** |

> The SET0-T15 document (`09-tensor-shape-mapping.md`) listed exact storage
> dtype as UNKNOWN. This document resolves that: all 1199 tensors are `BF16`
> as confirmed by direct header inspection.

---

## 15. Scope Boundary

This document is evidence acquisition only. It does NOT contain:

- Parameter counts
- Byte totals per tensor family
- Checkpoint memory footprint
- Runtime memory calculations
- Hardware placement design
- Kernel implementations

These topics are deferred to subsequent tasks (T17+).

---

## 16. Acceptance

```text
TASK:                    SET0-T16
SOURCE:                  Qwen/Qwen3.8-27B
REVISION:                1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
METADATA METHOD:         Safetensors shard header range-read via huggingface_hub
TENSOR COUNT:            1199
SHARD COUNT:             18
SHAPES:                  AVAILABLE
DTYPES:                  AVAILABLE
INDEX CROSS-CHECK:       PASS
- Indexed tensors:        1199
- Header-resolved:        1199
- Missing from headers:    0
- Unexpected in headers:   0
- Shard count:            18
- Shard set match:        True
UNEXPECTED TENSORS:        none

SCOPE:                   COMPLIANT
```
