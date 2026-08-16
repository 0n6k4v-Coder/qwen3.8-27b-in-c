# Qwen3.8-27B config.json Verification

## Artifact

* Local path: model/official/config.json
* Repository: Qwen/Qwen3.8-27B
* Revision: 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0

## Integrity

* JSON syntax: VALID
* Local SHA-256: 191e0af232104ed8b65258cf3fb2b842e288008baca7633c11b82a1ac7203aab
* Official comparison method: Byte-level diff (diff) after authenticated fetch of config.json at pinned revision to /tmp
* Integrity result: VERIFIED — local file and official file are byte-identical (4312 bytes each)

## Provenance

* SOURCE.md revision: 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
* Verified revision: 1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0
* Provenance result: VERIFIED — SOURCE.md revision matches the pinned revision used for config.json retrieval

## Finding

VERIFIED

## Reason

The local config.json was fetched from https://huggingface.co/Qwen/Qwen3.8-27B/resolve/1d4bf0f2ff6012fd82039f2fa52739d0dd7c60c0/config.json (HTTP 200) and compared byte-for-byte with diff. The two files are identical (4312 bytes, no differences). JSON syntax validates successfully. The revision in SOURCE.md matches the pinned revision. The official ETag from HuggingFace headers is "706cebd746c4b6f2b1d1f892630867acfdfd3df8" (Git blob SHA-1). No architecture fields were interpreted during this verification.

## Scope

SET0-T04 ONLY
