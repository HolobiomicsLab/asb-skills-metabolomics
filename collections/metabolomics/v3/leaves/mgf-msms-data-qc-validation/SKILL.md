---
name: mgf-msms-data-qc-validation
description: Use when preparing MGF files and feature tables from MZmine2 pre-processing
  as QIIME 2 artifacts, before compute-fragmentation-trees, and when checking that
  make-hierarchy outputs (rooted Phylogeny, filtered hashed table, feature data columns)
  are correctly formed.
when_to_use_negative:
- Input is raw mzXML/mzML/mzDATA data — MZmine2 feature detection must run first to
  produce the MGF and feature table
- 'Only MS1 data with no MS/MS spectra are present: ''Only chemical features with
  MS/MS data are included; features with only MS1 are not considered.'''
- The goal is fingerprint prediction or hierarchy construction itself, not checking
  that inputs and outputs of those steps are well formed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Imports the MGF and feature table as MassSpectrometryFeatures and FeatureTable[Frequency]
    artifacts consumed by q2-qemistree
  repo: https://github.com/biocore/q2-qemistree
- name: q2-qemistree
  role: Provides the methods (compute-fragmentation-trees, predict-fingerprints, make-hierarchy)
    whose inputs and outputs are validated here
  repo: https://github.com/biocore/q2-qemistree
- name: MZmine2
  role: Upstream pre-processing tool that must produce the MGF and feature table consumed
    by this QC step
  repo: http://mzmine.github.io
- name: GNPS
  role: Source of MS/MS library matches optionally attached as --i-ms2-matches to
    make-hierarchy
- name: SIRIUS
  role: Downstream consumer of the validated MGF; requires the correct ion-mode and
    instrument profile parameters to be set during validation
  repo: https://bio.informatik.uni-jena.de/sirius/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41589-020-00677-3
    title: Chemically informed analyses of metabolomics mass spectrometry data with
      Qemistree
  source_content_hash: sha256:0c122e0913ef73b36a8a1f5528f7908d133f7c9eb6ea0fc56bae6562381c3302
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# mgf-msms-data-qc-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control and validation of MGF-format MS/MS feature files and paired feature tables before building Qemistree chemical hierarchies: confirming artifact types (MassSpectrometryFeatures, FeatureTable[Frequency]), correct ion-mode and instrument profiles for SIRIUS, and that downstream outputs (rooted tree, hashed table, feature data columns) are structurally valid.

## When to use

Use when preparing MGF files and feature tables from MZmine2 pre-processing as QIIME 2 artifacts, before compute-fragmentation-trees, and when checking that make-hierarchy outputs (rooted Phylogeny, filtered hashed table, feature data columns) are correctly formed.

## When NOT to use

- Input is raw mzXML/mzML/mzDATA data — MZmine2 feature detection must run first to produce the MGF and feature table
- Only MS1 data with no MS/MS spectra are present: 'Only chemical features with MS/MS data are included; features with only MS1 are not considered.'
- The goal is fingerprint prediction or hierarchy construction itself, not checking that inputs and outputs of those steps are well formed

## Inputs

- MGF file of MS/MS fragmentation spectra from MZmine2 pre-processing (e.g. sirius.mgf)
- Feature table with peak areas as BIOM (e.g. feature-table.biom)
- QIIME 2 artifact of type MassSpectrometryFeatures
- QIIME 2 artifact of type FeatureTable[Frequency]
- MS/MS library matches from GNPS (optional, for --i-ms2-matches)

## Outputs

- Validated QIIME 2 artifacts ready for q2-qemistree methods
- Rooted chemical hierarchy of type Phylogeny[Rooted]
- Hashed feature table with features lacking fingerprints removed
- Feature data with columns featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, table_number

## How to apply

After MZmine2 feature detection produces an MGF file with associated peak areas and MS/MS fragmentation spectra, import it as a QIIME 2 artifact of type MassSpectrometryFeatures together with the feature table as FeatureTable[Frequency], matching the demo inputs sirius.mgf and feature-table.biom. At each Qemistree step, validate the intermediate: compute-fragmentation-trees must run with the correct ion mode (e.g. --p-ions-considered '[M+H]'), instrument profile (--p-profile orbitrap), and mass tolerance (--p-ppm-max 15; fingerprints predicted with --p-ppm-max 20); rerank-molecular-formulas uses --p-zodiac-threshold 0.95 to select the top-scoring formula per feature. Confirm make-hierarchy returns a tree of type Phylogeny[Rooted], a hashed feature table in which MS1 features without fingerprints are filtered out, and feature data containing the columns featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, and table_number. If MS/MS spectral library matches from GNPS are available, check they are passed as --i-ms2-matches so library hits are preserved in the hierarchy.

## Related tools

- **QIIME 2** (Imports the MGF and feature table as MassSpectrometryFeatures and FeatureTable[Frequency] artifacts consumed by q2-qemistree) — https://github.com/biocore/q2-qemistree
- **q2-qemistree** (Provides the methods (compute-fragmentation-trees, predict-fingerprints, make-hierarchy) whose inputs and outputs are validated here) — https://github.com/biocore/q2-qemistree
- **MZmine2** (Upstream pre-processing tool that must produce the MGF and feature table consumed by this QC step) — http://mzmine.github.io
- **GNPS** (Source of MS/MS library matches optionally attached as --i-ms2-matches to make-hierarchy)
- **SIRIUS** (Downstream consumer of the validated MGF; requires the correct ion-mode and instrument profile parameters to be set during validation) — https://bio.informatik.uni-jena.de/sirius/

## Examples

```
qiime tools import --input-path sirius.mgf --output-path msfeatures.qza --type 'MassSpectrometryFeatures'
```

## Evaluation signals

- Tree artifact type is exactly Phylogeny[Rooted] after make-hierarchy
- Hashed feature table excludes MS1 features without fingerprints
- Feature data contains featureID, parent_mass, retention_time, csi_smiles, ms2_smiles, table_number
- compute-fragmentation-trees ran with --p-ppm-max 15, --p-profile orbitrap, and the correct --p-ions-considered for the dataset's ionization mode
- predict-fingerprints uses --p-ppm-max 20 and predicts 2936 molecular properties per feature

## Limitations

- Only chemical features with MS/MS data are included; features with only MS1 are not considered.
- Fingerprint prediction depends on the quality and coverage of MS/MS spectral databases available for training the predictive models
- Qemistree was initially developed under Sirius 4.0.1; since Sirius 4.0.1 reached end of life it was adapted to work with new Sirius versions (>4.4.29)

## Evidence

- [results] Validation of hierarchy outputs: [evidence span withheld — anchor status: not_found]
- [results] Parameter validation for SIRIUS runs: [evidence span withheld — anchor status: not_found]
