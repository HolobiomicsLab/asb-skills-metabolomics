---
name: qiime2-artifact-import-export
description: Use when starting from an MZmine2-produced MGF and quant/feature table
  (e.g. MassIVE MSV000082969 Cheetah Quant table) and you need typed QIIME 2 artifacts
  (FeatureTable[Frequency], MassSpectrometryFeatures) as Qemistree inputs.
when_to_use_negative:
- Data are already QIIME 2 artifacts (no need to re-import — go straight to Qemistree
  methods).
- Only MS1 features are present without MS/MS fragmentation spectra — these are not
  the MGF/feature-table inputs Qemistree expects.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: QIIME 2
  role: Provides `qiime tools import/export` commands that create typed artifacts
    from raw metabolomics files and export artifacts to files for downstream analysis.
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:d2f155bf7a0bff8d2fc0e2edb75ddb119723b50521230f59492190674c938f91
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/biocore/q2-qemistree
  repo_url: https://github.com/biocore/q2-qemistree
  tools:
  - CSI:FingerID
  - GNPS
  - MZmine2
  - Python
  - QIIME 2
  - Qemistree
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qiime2-artifact-import-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import raw MGF/feature-table files (e.g. from MZmine2/GNPS quant tables) into QIIME 2 artifacts of the correct semantic types and export them back to disk, so that metabolomics data can feed Qemistree and other QIIME 2 workflows.

## When to use

Use when starting from an MZmine2-produced MGF and quant/feature table (e.g. MassIVE MSV000082969 Cheetah Quant table) and you need typed QIIME 2 artifacts (FeatureTable[Frequency], MassSpectrometryFeatures) as Qemistree inputs.

## When NOT to use

- Data are already QIIME 2 artifacts (no need to re-import — go straight to Qemistree methods).
- Only MS1 features are present without MS/MS fragmentation spectra — these are not the MGF/feature-table inputs Qemistree expects.

## Inputs

- MZmine2-processed MGF of MS/MS fragmentation spectra
- Quant/feature table (BIOM, e.g. feature-table.biom from the q2-qemistree demo, or GNPS quant tables such as Cheetah Quant table or Evaluation dataset quant table)

## Outputs

- QIIME 2 artifact of type FeatureTable[Frequency]
- QIIME 2 artifact of type MassSpectrometryFeatures
- Exported plain files (e.g. BIOM TSV, MGF) from artifacts

## How to apply

Choose the semantic type that matches the data: a sample-by-feature abundance table (BIOM format) maps to FeatureTable[Frequency], while the MGF of MS/MS spectra maps to MassSpectrometryFeatures. Use `qiime tools import --input-path ... --type FeatureTable[Frequency]` (or MassSpectrometryFeatures) to create artifacts, and `qiime tools export` to dump them back to plain files for downstream tools (e.g. Emperor, ClassyFire contingency tables). Verify with `qiime tools peek` that each artifact carries the expected type before passing it to Qemistree methods — the tree is generated from the feature table, the MGF, and the MS/MS library matches together.

## Related tools

- **QIIME 2** (Provides `qiime tools import/export` commands that create typed artifacts from raw metabolomics files and export artifacts to files for downstream analysis.) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime tools import --input-path sirius.mgf --type 'MassSpectrometryFeatures' --input-format MassSpectrometryFeaturesFormat --output-path sirius.qza
```

## Evaluation signals

- `qiime tools peek` reports the expected type (e.g. FeatureTable[Frequency] or MassSpectrometryFeatures) for every imported artifact.
- Exported BIOM/MGF files round-trip to the same row/column counts as the source quant table.
- Downstream Qemistree commands (make-hierarchy, compute-fragmentation-trees) accept the artifacts without type errors.
- Artifact UUIDs match between imported and exported copies.

## Limitations

- Only chemical features with MS/MS data are included; features with only MS1 are not considered.
- MS1 features without fingerprints are filtered out of the merged feature table.

## Evidence

- [methods] Import feature table: [evidence span withheld — anchor status: not_found]
