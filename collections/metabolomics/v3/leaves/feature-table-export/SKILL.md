---
name: feature-table-export
description: Use when raw positive-ion-mode LC-MS/MS mzML files must be converted
  into a per-feature CSV (m/z, RT, representative MS2 spectrum) to feed SIRIUS formula/adduct
  assignment and MVP candidate ranking.
when_to_use_negative:
- Input is already a processed feature table or identified-metabolite mapping — the
  original meat study lacked these, which is why this export step was needed.
- Working with negative ion mode data when the downstream evaluation focuses on positive
  ion mode (the article states 'For this evaluation, we focus on the positive ion
  mode data').
- Raw data is not mzML (e.g. proprietary vendor formats) — MZmine3 import in this
  workflow assumes mzML.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine3
  role: Feature extraction (mass detection, chromatogram building, feature detection/deconvolution),
    representative spectra selection, and feature-table export
- name: SIRIUS
  role: 'Downstream consumer: formula and adduct assignment on the exported feature
    table (SIRIUS v6.1.0)'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:1dcc14024daf6b628be5e44b2ca7c1a672163701be20466f63244b5d8f75e873
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine3
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# feature-table-export

## Summary

Export MZmine3-detected LC-MS/MS features (m/z, retention time, and linked representative MS2 spectra) to a feature_table.csv for downstream formula/adduct assignment and candidate ranking.

## When to use

Use when raw positive-ion-mode LC-MS/MS mzML files must be converted into a per-feature CSV (m/z, RT, representative MS2 spectrum) to feed SIRIUS formula/adduct assignment and MVP candidate ranking.

## When NOT to use

- Input is already a processed feature table or identified-metabolite mapping — the original meat study lacked these, which is why this export step was needed.
- Working with negative ion mode data when the downstream evaluation focuses on positive ion mode (the article states 'For this evaluation, we focus on the positive ion mode data').
- Raw data is not mzML (e.g. proprietary vendor formats) — MZmine3 import in this workflow assumes mzML.

## Inputs

- LC-MS/MS mzML raw files (positive ion mode) downloaded from a public deposit such as Metabolomics Workbench

## Outputs

- feature_table.csv containing per-feature m/z, retention time, and linked representative MS2 spectra

## How to apply

Import the positive ion mode mzML files into MZmine3, run mass detection followed by chromatogram building to construct extracted ion chromatograms, then execute MZmine3 feature detection/deconvolution to obtain chromatographic features characterized by m/z and retention time. Apply MZmine3's representative spectra selection module so each feature receives a single representative MS2 spectrum. Export the feature list (m/z, retention time, and linked representative MS2 spectra per feature) to feature_table.csv. This table then feeds formula and adduct assignment with SIRIUS v6.1.0 and candidate retrieval within 0.01 Da tolerance from FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library. The task card validation expects feature_table.csv to exist, parse as CSV, and contain a feature count matching the study's reported number (900 features, 866 with formula/adduct assignments via SIRIUS).

## Related tools

- **MZmine3** (Feature extraction (mass detection, chromatogram building, feature detection/deconvolution), representative spectra selection, and feature-table export)
- **SIRIUS** (Downstream consumer: formula and adduct assignment on the exported feature table (SIRIUS v6.1.0))

## Examples

```
mzmine3 -c mzmine_batch.xml --input mzml/*.mzML --output feature_table.csv
```

## Evaluation signals

- feature_table.csv exists and parses as a valid CSV file.
- The number of features in feature_table.csv matches the expected count from the study (approximately 900 features).
- Each row carries three elements: an m/z value, a retention time, and a linked representative MS2 spectrum.
- Downstream SIRIUS formula/adduct assignment succeeds on the exported table (866 of 900 features received assignments in the reference run).
- A single representative MS2 spectrum is assigned per feature (no features lacking a linked representative spectrum).

## Limitations

- Neither the processed spectra nor the mapping to the identified metabolites was provided by the original study — this export is a replication, not a direct reuse of prior output.
- Reported MVP annotations derived from this table are putative and require experimental validation.
- Feature counts depend on MZmine3 parameters and the specific mzML inputs; the ~900 figure is study-specific and will vary with different datasets.

## Evidence

- [results] Feature extraction and representative spectra selection using MZmine3: "Feature extraction and representative spectra selection are performed using MZmine3"
- [results] Export format of the feature table: "followed by formula and adduct assignment with SIRIUS v6.1.0.35 Candidate structures are retrieved within a 0.01 Da tolerance"
- [results] Exported table and expected feature count: [evidence span withheld — anchor status: not_found]
- [results] Why the export was needed (original data unavailable): "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
- [results] Positive ion mode focus: "For this evaluation, we focus on the positive ion mode data"
- [readme] README pointing to the MassSpecGym dataset used for training/evaluation: "Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym). Follow their instruction to download the spectra and candidate dataset."
