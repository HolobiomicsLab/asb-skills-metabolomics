---
name: ms2-representative-spectra-selection
description: Use when raw LC-MS/MS mzML files contain multiple MS2 scans per chromatographic
  feature and you need one representative MS2 spectrum per feature (m/z, RT) to feed
  formula assignment or ML-based annotation.
when_to_use_negative:
- Input is already a processed feature table with per-feature MS2 spectra — the original
  meat study case where processed spectra were missing required full replication,
  but a ready feature list makes this skill redundant
- Data are in negative ion mode or a non-LC-MS/MS acquisition, since the demonstrated
  evaluation focuses on positive ion mode LC–MS/MS data
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MZmine3
  role: Performs feature extraction (mass detection, chromatogram building, deconvolution)
    and representative spectra selection to assign one MS2 spectrum per feature
- name: SIRIUS v6.1.0
  role: Consumes the selected representative spectra downstream for formula and adduct
    assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:c5e72e6547a5b3faf7f40f9b0af503d3358b8d4a18be9f11f0c9a585aa956469
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ms2-representative-spectra-selection

## Summary

Selects a single representative MS2 spectrum for each chromatographic feature detected from LC-MS/MS mzML files using MZmine3, so downstream formula/adduct assignment (SIRIUS v6.1.0) and metabolite annotation operate on one canonical spectrum per feature. Validated on the cultured-vs-conventional meat Metabolomics Workbench deposit, where ~900 features were detected.

## When to use

Use when raw LC-MS/MS mzML files contain multiple MS2 scans per chromatographic feature and you need one representative MS2 spectrum per feature (m/z, RT) to feed formula assignment or ML-based annotation.

## When NOT to use

- Input is already a processed feature table with per-feature MS2 spectra — the original meat study case where processed spectra were missing required full replication, but a ready feature list makes this skill redundant
- Data are in negative ion mode or a non-LC-MS/MS acquisition, since the demonstrated evaluation focuses on positive ion mode LC–MS/MS data

## Inputs

- Raw LC-MS/MS mzML files (positive ion mode) from a public deposit such as Metabolomics Workbench
- MZmine3 installation with mass detection, chromatogram building, feature deconvolution, and representative spectra selection modules

## Outputs

- feature_table.csv containing per-feature m/z, retention time, and the linked representative MS2 spectrum
- MZmine3 feature list with chromatographic features characterized by m/z and retention time

## How to apply

Import the positive ion mode mzML files (e.g., the meat metabolomics files from the Metabolomics Workbench) into MZmine3 and run mass detection followed by chromatogram building to construct extracted ion chromatograms. Execute MZmine3 feature detection/deconvolution to obtain chromatographic features characterized by m/z and retention time, then run MZmine3's representative spectra selection module to assign a single MS2 spectrum to each feature. Export the feature list (m/z, retention time, linked representative MS2 spectrum) to feature_table.csv. In the MVP meat-study application this workflow detected ~900 features, 866 of which subsequently received formula and adduct assignments from SIRIUS v6.1.0; note the original meat study deposited only raw mzML, so processing must be replicated rather than reused.

## Related tools

- **MZmine3** (Performs feature extraction (mass detection, chromatogram building, deconvolution) and representative spectra selection to assign one MS2 spectrum per feature)
- **SIRIUS v6.1.0** (Consumes the selected representative spectra downstream for formula and adduct assignment)

## Evaluation signals

- feature_table.csv exists and parses as valid CSV
- Detected feature count matches the approximately 900 features reported for the meat study
- Every feature row has a m/z and retention time, and a single linked representative MS2 spectrum (no feature without a spectrum)
- Downstream SIRIUS formula/adduct assignment succeeds for the majority of features (866 of ~900 in the reference run)

## Limitations

- Neither the processed spectra nor the mapping to identified metabolites was provided by the original meat study, so results must be replicated from raw mzML rather than validated directly
- Annotations downstream of representative spectra selection are putative and require experimental validation
- Representation quality could benefit from more accurate peak labeling and additional views

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3, followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] Raw data availability on Metabolomics Workbench: "LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench"
- [results] Need to replicate because processed outputs were not deposited: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
- [results] Positive ion mode scope: "For this evaluation, we focus on the positive ion mode data"
