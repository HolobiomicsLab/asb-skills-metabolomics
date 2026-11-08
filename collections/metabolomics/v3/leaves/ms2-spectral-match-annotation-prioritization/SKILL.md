---
name: ms2-spectral-match-annotation-prioritization
description: Use when you have a Qemistree hierarchy and merged feature data and need
  chemical class labels (e.g., acyl-carnitines, flavonoids) to trace class-specific
  subtrees against sample metadata.
when_to_use_negative:
- Features lack both MS/MS spectral library matches and CSI:FingerID structural predictions,
  so no SMILES exist to submit to ClassyFire
- The hierarchy has not yet been built — run compute-fragmentation-trees, predict-fingerprints,
  and make-hierarchy first, as this taxonomy step annotates existing features rather
  than relating them
- Only MS1 features without fingerprints are available; these are filtered out and
  cannot be classified
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Qemistree
  role: Provides the get-classyfire-taxonomy method that submits feature SMILES to
    ClassyFire and records structure_source
  repo: https://github.com/biocore/q2-qemistree
- name: ClassyFire
  role: Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass,
    direct parent) to molecules annotated via spectral library matching and in silico
    prediction
- name: GNPS
  role: Source of MS/MS spectral library matches (ms2_smiles) that are prioritized
    for taxonomy assignment
- name: CSI:FingerID
  role: Fallback source of predicted structures (csi_smiles) used when no spectral
    library match exists
- name: QIIME 2
  role: Plugin framework in which q2-qemistree's get-classyfire-taxonomy is executed
    as `qiime qemistree get-classyfire-taxonomy`
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:596bcd93358138cf223ebc9e9de3d36f6377b663ff91a5e9de05cf9d09a2d6e6
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
  - ClassyFire
  - GNPS
  - Python
  - QIIME 2
  - Qemistree
  - iTOL
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ms2-spectral-match-annotation-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each molecular feature in a Qemistree chemical hierarchy a 5-level ClassyFire taxonomy (kingdom, superclass, class, subclass, direct parent), prioritizing MS/MS spectral library matches over CSI:FingerID in silico predictions while recording each assignment's origin in a structure_source column.

## When to use

Use when you have a Qemistree hierarchy and merged feature data and need chemical class labels (e.g., acyl-carnitines, flavonoids) to trace class-specific subtrees against sample metadata.

## When NOT to use

- Features lack both MS/MS spectral library matches and CSI:FingerID structural predictions, so no SMILES exist to submit to ClassyFire
- The hierarchy has not yet been built — run compute-fragmentation-trees, predict-fingerprints, and make-hierarchy first, as this taxonomy step annotates existing features rather than relating them
- Only MS1 features without fingerprints are available; these are filtered out and cannot be classified

## Inputs

- Merged chemical hierarchy tree (qemistree.qza) from make-hierarchy
- Merged feature data (merged-feature-data.qza) containing molecular formulas and SMILES from spectral matching or CSI:FingerID

## Outputs

- Classified feature data (classified-merged-feature-data.qza) with kingdom, superclass, class, subclass, direct_parent, and structure_source columns

## How to apply

Load the merged chemical hierarchy tree (qemistree.qza) and merged feature data produced by the upstream make-hierarchy step, then run Qemistree's get-classyfire-taxonomy, which uses ClassyFire to assign a 5-level chemical taxonomy (chemical kingdom, superclass, class, subclass, and direct parent ontology) to each molecule. The tool prioritizes MS/MS spectral library matches (ms2_smiles) for taxonomy assignment and falls back to CSI:FingerID-predicted structures (csi_smiles) only when no library match exists; the structure_source column records which origin was used for each feature. Export the classified feature data and inspect the 'class' column to extract features annotated as classes of interest (e.g., flavonoids, alkaloids, phospholipids, acyl-carnitines, O-glycosyl compounds), then check their membership against sample metadata to trace class-specific subtrees. Save the annotated output as a .qza artifact for downstream visualization in iTOL or Empress.

## Related tools

- **Qemistree** (Provides the get-classyfire-taxonomy method that submits feature SMILES to ClassyFire and records structure_source) — https://github.com/biocore/q2-qemistree
- **ClassyFire** (Assigns the 5-level chemical taxonomy (kingdom, superclass, class, subclass, direct parent) to molecules annotated via spectral library matching and in silico prediction)
- **GNPS** (Source of MS/MS spectral library matches (ms2_smiles) that are prioritized for taxonomy assignment)
- **CSI:FingerID** (Fallback source of predicted structures (csi_smiles) used when no spectral library match exists)
- **QIIME 2** (Plugin framework in which q2-qemistree's get-classyfire-taxonomy is executed as `qiime qemistree get-classyfire-taxonomy`) — https://github.com/biocore/q2-qemistree

## Examples

```
qiime qemistree get-classyfire-taxonomy --i-feature-data merged-feature-data.qza --o-classified-feature-data classified-merged-feature-data.qza
```

## Evaluation signals

- The classified feature data contains all five taxonomy columns: kingdom, superclass, class, subclass, and direct_parent
- Every classified feature carries a structure_source value indicating whether its SMILES came from ms2_smiles (spectral library match) or csi_smiles (CSI:FingerID prediction)
- Features with MS/MS spectral library matches should systematically have structure_source = ms2_smiles, reflecting the prioritization rule
- Class-level subtree membership matches expected biology, e.g., acyl-carnitine branches restricted to animal products and a flavonoid clade shared by plant products and honey (Fig. 3a pattern)
- Output is a valid QIIME 2 artifact (.qza) loadable for downstream tree visualization

## Limitations

- Only 1–20% of features typically have spectral library matches, depending on how well the sample type has been investigated, so most features rely on less-accurate CSI:FingerID predictions
- Annotation coverage from spectral matches alone is too low to adequately summarize the chemical content of complex samples
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases used to train the predictive models

## Evidence
