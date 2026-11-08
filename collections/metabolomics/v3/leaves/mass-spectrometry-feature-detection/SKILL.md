---
name: mass-spectrometry-feature-detection
description: Use when starting from raw LC-MS/MS files and the analysis goal is an
  untargeted metabolomics workflow requiring an MGF of MS1/MS2 entries plus a feature
  table of MS1 peak areas per sample as inputs to fingerprinting or networking tools.
when_to_use_negative:
- The input is already a processed feature table (e.g., a quant table or FeatureData[Molecules]
  artifact) — feature detection has already been done and repeating it would change
  feature IDs.
- The goal is only to re-network existing GNPS FBMN output rather than re-detect features;
  Qemistree can incorporate multiple datasets iteratively without repeated feature
  detection or retention time alignment.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine2
  role: Feature detection on raw mass-spectrometry files to produce the MGF and feature
    table consumed by Qemistree and GNPS
- name: GNPS
  role: Feature-Based Molecular Networking consumes the feature detection outputs
    to obtain MS2 spectral library matches
- name: QIIME 2 / q2-qemistree
  role: Receives the feature table (FeatureData[Molecules]) and MGF as inputs to compute-fragmentation-trees
    and downstream hierarchy building
  repo: https://github.com/biocore/q2-qemistree
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:7c72cfb6e0db0f29e3c0ce24a2567ce6960e650b01d744ec92efd3ebe838e00b
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
  - MZmine2
  - QIIME 2
  - QIIME 2 / q2-qemistree
  - Qemistree
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-feature-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-process raw LC-MS/MS data (mzXML, mzML, or mzDATA) into chemical features — a list of observed ions with peak areas and MS/MS fragmentation spectra — so that downstream fingerprint prediction and chemical tree building (e.g., Qemistree) can operate.

## When to use

Use when starting from raw LC-MS/MS files and the analysis goal is an untargeted metabolomics workflow requiring an MGF of MS1/MS2 entries plus a feature table of MS1 peak areas per sample as inputs to fingerprinting or networking tools.

## When NOT to use

- The input is already a processed feature table (e.g., a quant table or FeatureData[Molecules] artifact) — feature detection has already been done and repeating it would change feature IDs.
- The goal is only to re-network existing GNPS FBMN output rather than re-detect features; Qemistree can incorporate multiple datasets iteratively without repeated feature detection or retention time alignment.

## Inputs

- raw LC-MS/MS files (.mzXML, .mzML, or .mzDATA) from a repository such as MassIVE (e.g., MSV000085226)
- MZmine2 batch file (e.g., batchQE-MZmine) specifying feature detection parameters

## Outputs

- MGF file containing MS1 and MS2 entries (e.g., sirius.mgf format)
- feature table of MS1 peak areas per sample (e.g., feature-table.biom)
- list of chemical features with associated peak areas and MS/MS fragmentation spectra

## How to apply

Run feature detection (e.g., MZmine2, guided by an example batch file such as batchQE-MZmine) on the raw files from MassIVE accessions (e.g., MSV000085226 for the Global FoodOmics subset) to generate two coupled outputs: an MGF containing MS1 and MS2 entries, and a feature table of MS1 peak areas per sample. Only chemical features with MS/MS data are carried forward; features with only MS1 are not considered. The feature table and MGF then feed directly into FBMN/GNPS for spectral library matching and into SIRUS-based workflows (compute-fragmentation-trees) via QIIME 2. For the Qemistree demo path, the same formats are expected — feature-table.biom and sirius.mgf. Quality filtering of the resulting features (e.g., in the food dataset leaving 663 features) determines the denominator for downstream annotation-rate calculations.

## Related tools

- **MZmine2** (Feature detection on raw mass-spectrometry files to produce the MGF and feature table consumed by Qemistree and GNPS)
- **GNPS** (Feature-Based Molecular Networking consumes the feature detection outputs to obtain MS2 spectral library matches)
- **QIIME 2 / q2-qemistree** (Receives the feature table (FeatureData[Molecules]) and MGF as inputs to compute-fragmentation-trees and downstream hierarchy building) — https://github.com/biocore/q2-qemistree

## Examples

```
wget https://raw.githubusercontent.com/biocore/q2-qemistree/master/q2_qemistree/demo/sirius.mgf
```

## Evaluation signals

- The MGF contains paired MS1 and MS2 entries and the feature table holds MS1 peak areas for each sample — format validity is the primary invariant.
- Features lacking MS/MS data (MS1-only) are excluded from downstream steps per the Qemistree filter rule.
- After quality filtering, the feature count should land in a defensible range for the dataset (e.g., 663 features in the Global FoodOmics food dataset).
- Subsequent FBMN spectral-library match rate should fall in the typical 1–20% range reported for features with spectral matches.
- The outputs import cleanly as QIIME 2 artifacts (e.g., feature-table.biom loads as FeatureTable, clusterinfo_summary tsv loads as FeatureData[Molecules]).

## Limitations

- Typically only 1–20% of features have spectral library matches, depending on how well the sample type has been investigated.
- Fingerprint prediction quality depends on the coverage of MS/MS spectral databases used to train the predictive models; in negative ionization mode fewer molecular fingerprints can be confidently predicted due to fewer publicly available reference spectra.
- Qemistree was initially developed under SIRIUS 4.0.1 and later adapted for SIRIUS >4.4.29 — version mismatches can affect behavior.

## Evidence
