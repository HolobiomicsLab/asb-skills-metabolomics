---
name: mass-spectrometry-feature-table-import
description: 'Use when starting a Qemistree workflow: untargeted MS data (mzXML, mzML
  or mzDATA) has been pre-processed in MZmine2 into an MGF and a quant/biom feature
  table that must be loaded as QIIME 2 artifacts before fragmentation-tree computation
  or hierarchy building.'
when_to_use_negative:
- The input is already a QIIME 2 artifact (MassSpectrometryFeatures or FeatureTable[Frequency])
  — re-importing is unnecessary.
- 'The data have not yet been feature-detected: raw vendor files should first be processed
  in MZmine2 (or equivalent) to generate the MGF and feature table rather than imported
  directly.'
- Only MS1 data are available with no MS/MS spectra — Qemistree fingerprint prediction
  requires MS/MS, so features with only MS1 cannot be used to build the hierarchy.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QIIME 2
  role: Hosts the q2-qemistree plugin; 'qiime tools import' converts MGF and BIOM
    feature-table files into typed artifacts (MassSpectrometryFeatures, FeatureTable[Frequency])
    consumed by all qemistree methods.
  repo: https://github.com/biocore/q2-qemistree
- name: q2-qemistree
  role: Provides the plugin commands (compute-fragmentation-trees, rerank-molecular-formulas,
    predict-fingerprints, make-hierarchy, prune-hierarchy) that operate on the imported
    artifacts; includes demo inputs feature-table.biom and sirius.mgf.
  repo: https://github.com/biocore/q2-qemistree
- name: MZmine2
  role: 'Upstream feature detection: pre-processes raw mzXML/mzML/mzDATA files to
    produce the MGF and the feature table that this import skill ingests.'
- name: SIRIUS
  role: Called by compute-fragmentation-trees after import to determine molecular
    formulas and fragmentation trees from the imported MS/MS spectra; q2-qemistree
    requires a SIRIUS binary (e.g. sirius-4.9.3) in the environment.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:22d14c6ec25e4700ca9884937a5619f59537ea99e5f2c0b647ee441966776fda
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
  - QIIME 2
  - Qemistree
  - SIRIUS
  - q2-qemistree
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectrometry-feature-table-import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Import LC-MS/MS feature detection outputs (an MGF file and a feature table) into QIIME 2 artifacts of type MassSpectrometryFeatures and FeatureTable[Frequency] so they can be used as inputs to downstream Qemistree commands such as compute-fragmentation-trees and make-hierarchy.

## When to use

Use when starting a Qemistree workflow: untargeted MS data (mzXML, mzML or mzDATA) has been pre-processed in MZmine2 into an MGF and a quant/biom feature table that must be loaded as QIIME 2 artifacts before fragmentation-tree computation or hierarchy building.

## When NOT to use

- The input is already a QIIME 2 artifact (MassSpectrometryFeatures or FeatureTable[Frequency]) — re-importing is unnecessary.
- The data have not yet been feature-detected: raw vendor files should first be processed in MZmine2 (or equivalent) to generate the MGF and feature table rather than imported directly.
- Only MS1 data are available with no MS/MS spectra — Qemistree fingerprint prediction requires MS/MS, so features with only MS1 cannot be used to build the hierarchy.

## Inputs

- MGF file of MS/MS fragmentation spectra (one spectrum per chemical feature, e.g. sirius.mgf)
- Feature table of MS1 peak areas (BIOM, e.g. feature-table.biom or a dataset-specific quant table such as the Cheetah Quant table)
- Raw mzXML / mzML / mzDATA files for upstream MZmine2 feature detection
- Feature metadata (featureID, parent_mass, retention_time) as sample-metadata/metadata TSV when attaching annotations

## Outputs

- QIIME 2 artifact of type MassSpectrometryFeatures (imported MGF)
- QIIME 2 artifact of type FeatureTable[Frequency] (imported feature table)
- Feature data artifact with featureID, parent_mass, retention_time columns (produced downstream by make-hierarchy)

## How to apply

Pre-process the raw mass-spectrometry files (.mzXML, .mzML or .mzDATA) in MZmine2 to produce an MGF file of MS/MS fragmentation spectra and a feature table of observed ions with peak areas, as done for the Cheetah Quant table from MassIVE MSV000082969. Import the MGF with 'qiime tools import' using the qemistree MassSpectrometryFeatures format to obtain a MassSpectrometryFeatures artifact, and import the feature table (BIOM format, e.g. feature-table.biom from the q2-qemistree demo) as a FeatureTable[Frequency] artifact. Only chemical features with MS/MS data are retained — features with only MS1 are not considered for fragmentation-tree computation. These artifacts are then fed to qiime qemistree compute-fragmentation-trees (SIRIUS-based, e.g. --p-ppm-max 15, --p-profile orbitrap) and ultimately make-hierarchy, where MS1 features without fingerprints are filtered out of the merged feature table.

## Related tools

- **QIIME 2** (Hosts the q2-qemistree plugin; 'qiime tools import' converts MGF and BIOM feature-table files into typed artifacts (MassSpectrometryFeatures, FeatureTable[Frequency]) consumed by all qemistree methods.) — https://github.com/biocore/q2-qemistree
- **q2-qemistree** (Provides the plugin commands (compute-fragmentation-trees, rerank-molecular-formulas, predict-fingerprints, make-hierarchy, prune-hierarchy) that operate on the imported artifacts; includes demo inputs feature-table.biom and sirius.mgf.) — https://github.com/biocore/q2-qemistree
- **MZmine2** (Upstream feature detection: pre-processes raw mzXML/mzML/mzDATA files to produce the MGF and the feature table that this import skill ingests.)
- **SIRIUS** (Called by compute-fragmentation-trees after import to determine molecular formulas and fragmentation trees from the imported MS/MS spectra; q2-qemistree requires a SIRIUS binary (e.g. sirius-4.9.3) in the environment.)

## Examples

```
qiime tools import --input-path sirius.mgf --type MassSpectrometryFeatures; qiime tools import --input-path feature-table.biom --type FeatureTable[Frequency]
```

## Evaluation signals

- The imported MGF artifact reports QIIME 2 semantic type MassSpectrometryFeatures ('qiime tools peek' or 'qiime tools import' log).
- The imported feature-table artifact reports semantic type FeatureTable[Frequency] and preserves the original ion identifiers from the quant table.
- Feature IDs in the MGF and the BIOM table correspond one-to-one; mismatched IDs cause downstream make-hierarchy to produce a feature table lacking parent_mass/retention_time columns.
- Only features with MS/MS spectra are present in the MGF — MS1-only features are excluded before fragmentation-tree computation.
- Downstream qiime qemistree make-hierarchy runs without error and outputs a Phylogeny[Rooted] tree plus a hashed feature table in which MS1 features without fingerprints have been filtered out.

## Limitations

- Typically only 1–20% of features obtain spectral library matches, so most imported features enter the workflow without direct structural annotation.
- In negative ionization mode fewer fingerprints can be confidently predicted (fewer public reference spectra), yielding less-extensive trees from the imported features.
- Qemistree was initially developed under SIRIUS 4.0.1; versions >4.4.29 are supported by the recent adaptation, but very old/new SIRIUS releases may break import–compute compatibility.
- Imported feature tables depend on upstream MZmine2 processing quality (feature detection and RT alignment); Qemistree itself does not re-detect or re-align features across datasets.

## Evidence

- [readme] we need to pre-process mass-spectrometry data (.mzXML, .mzML or .mzDATA files) using MZmine2 and produce the following inputs: 1. An MGF: [evidence span withheld — anchor status: not_found]
