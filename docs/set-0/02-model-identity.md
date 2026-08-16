# SET 0 — Model Identity

## Document Status

- Document: `02-model-identity.md`
- SET: `SET 0 — Model Reconnaissance`
- Source Task: `SET0-T05`
- Status: VERIFIED
- Purpose: Canonical record of the verified identity of the Qwen3.8-27B artifact

---

## 1. Model Identity

The official model under investigation is:

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

This identity has already been established through the official
repository provenance and verified local artifact.

---

## 2. Configuration Identity

The verified `config.json` contains:

```text
architectures:
Qwen3_5ForConditionalGeneration
```

Top-level model type:

```text
model_type:
qwen3_5
```

Language configuration model type:

```text
text_config.model_type:
qwen3_5_text
```

Transformers version recorded by the artifact:

```text
transformers_version:
5.8.0.dev0
```

Additional top-level identity-related fields:

```text
language_model_only:
false

tie_word_embeddings:
false
```

---

## 3. Multimodal Identity Indicators

The configuration explicitly contains multimodal identity fields:

```text
image_token_id:
248056

video_token_id:
248057

vision_start_token_id:
248053

vision_end_token_id:
248054
```

A dedicated `vision_config` is also present.

Therefore the artifact is not represented as a language-only configuration.

### Verified Finding

```text
language_model_only = false
vision_config = present
image/video token metadata = present
```

These fields establish the presence of multimodal configuration at the
model identity/configuration level.

They do not, by themselves, define the complete vision architecture.

---

## 4. Release Identity vs Configuration Identity

Two distinct identities must be preserved:

### Release / Model Identity

```text
Qwen3.8-27B
```

This identifies the model release represented by the official repository.

### Implementation / Configuration Identity

```text
Qwen3_5ForConditionalGeneration
qwen3_5
qwen3_5_text
```

These identify the implementation/configuration lineage represented by
the model artifact.

These two identities are related but must not be conflated.

---

## 5. Identity Reconciliation

An apparent discrepancy was identified:

```text
Repository:
Qwen3.8-27B
```

versus:

```text
architectures:
Qwen3_5ForConditionalGeneration

model_type:
qwen3_5
```

This discrepancy was independently investigated in:

```text
model/official/IDENTITY-RECONCILIATION.md
```

The official Qwen3.8 artifact documentation states that Qwen3.8 is built
on the architectural foundation of Qwen3.5.

Therefore the presence of the Qwen3.5-derived implementation identifiers
is consistent with the official artifact and is not evidence that the
wrong model repository was obtained.

### Reconciliation Result

```text
CONSISTENT WITH ARCHITECTURAL / IMPLEMENTATION LINEAGE
```

Confidence:

```text
HIGH
```

---

## 6. Important Identity Boundary

The following distinction is mandatory:

```text
Qwen3.8-27B
        ≠
"Qwen3.5 model"
```

The correct interpretation is:

```text
Qwen3.8-27B
        ↓
uses a Qwen3.5-derived implementation/configuration foundation
        ↓
but remains a distinct Qwen3.8 model artifact
```

Therefore, architecture details must be derived from the Qwen3.8-27B
artifact itself.

The Qwen3.5 configuration must not be substituted for missing Qwen3.8
configuration data.

---

## 7. VERIFIED FACTS

The following are established facts from the verified official artifact:

```text
✅ Model identity = Qwen3.8-27B
✅ Repository = Qwen/Qwen3.8-27B
✅ Pinned revision = 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
✅ architectures = Qwen3_5ForConditionalGeneration
✅ model_type = qwen3_5
✅ text_config.model_type = qwen3_5_text
✅ transformers_version = 5.8.0.dev0
✅ language_model_only = false
✅ tie_word_embeddings = false
✅ image_token_id = 248056
✅ video_token_id = 248057
✅ vision_start_token_id = 248053
✅ vision_end_token_id = 248054
✅ vision_config is present
```

---

## 8. Evidence-Based Inferences

The following conclusions are supported by the verified artifact and the
identity reconciliation:

### 8.1 Implementation lineage

Qwen3.8-27B uses a Qwen3.5-derived implementation/configuration
foundation.

### 8.2 Multimodal configuration

The model configuration represents a multimodal model rather than a
language-only model.

### 8.3 Identity consistency

The repository name and internal configuration identifiers are
consistent once release identity is distinguished from implementation
lineage.

---

## 9. Unknown / Deferred

This document intentionally does NOT establish:

```text
❓ Complete language architecture
❓ Exact attention mechanism
❓ Exact linear-attention algorithm
❓ Exact full-attention implementation
❓ MLP formulation
❓ Layer topology
❓ Tensor structure
❓ Parameter count
❓ Checkpoint size
❓ Runtime memory requirements
❓ Hardware workload placement
❓ CPU/GPU/NPU execution strategy
```

Those topics belong to later research tasks.

---

## 10. Relationship to Other SET 0 Documents

### Upstream Evidence

```text
docs/set-0/01-artifact-provenance.md
```

Establishes where the artifact came from and how its integrity was
verified.

### Identity Reconciliation Evidence

```text
model/official/IDENTITY-RECONCILIATION.md
```

Provides the detailed evidence resolving the Qwen3.8 vs Qwen3.5 naming
relationship.

### Downstream Analysis

```text
docs/set-0/04-core-architecture.md
```

will document the architecture derived from the Qwen3.8-specific
configuration.

This document therefore acts as the identity boundary between
provenance and architecture analysis.

---

## 11. Research Boundary

The following rule applies to subsequent SET 0 work:

> Use Qwen3.8-27B-specific artifact data as the ground truth for its
> architecture.

The existence of a Qwen3.5-derived implementation identity permits
lineage analysis, but does not authorize importing undocumented
Qwen3.5 architecture details into Qwen3.8-27B.

When a required architectural fact is not established by Qwen3.8-specific
evidence:

```text
UNKNOWN / REQUIRES FURTHER VERIFICATION
```

Do not silently substitute another model.

---

## 12. Final Acceptance

```text
MODEL IDENTITY:
VERIFIED

CONFIGURATION IDENTITY:
VERIFIED

IDENTITY RECONCILIATION:
VERIFIED

ARTIFACT:
Qwen3.8-27B

IMPLEMENTATION LINEAGE:
Qwen3.5-derived configuration identity

IDENTITY STATUS:
PASS
```

SET0-T05 Model Identity is complete and suitable for use as a persistent
research checkpoint.