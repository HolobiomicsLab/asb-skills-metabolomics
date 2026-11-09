---
name: bioassay-data-integration-into-feature-tables
description: Use when a feature quantification matrix from MZmine2 or Optimus must
  be merged with per-sample bioassay values (e.g. antiviral EC50 or inhibition) prior
  to running the bioactive molecular networking Pearson-correlation notebook.
when_to_use_negative:
- The input is already a bioactivity-augmented table (features_quantification_matrix_edited_bioactivity.csv)
  — re-appending bioassay columns would duplicate or overwrite the correlation input.
- The goal is plain feature-based molecular networking without bioactivity mapping;
  the standard GNPS FBMN workflow consumes the unedited features_quantification_matrix.csv
  and .MGF directly.
- Bioassay data are not matched to the LC-MS/MS-analyzed fractions (different samples
  or incompatible identifiers), since the Pearson correlation would then pair feature
  intensities with the wrong bioactivity values.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine2
  role: Produces the feature quantification table (features_quantification_matrix.csv)
    via the 'Export to CSV' module that the bioassay values are appended to; version
    2.31 was used in the study.
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative LC-MS/MS processing tool (KNIME/OpenMS based) that generates an
    equivalent quantification table; the 'MS/MS filter' option must be activated.
  repo: https://github.com/MolecularCartography/Optimus
- name: Jupyter notebook
  role: Bioactive_molecular_networks_v1.0.ipynb (R kernel) consumes the edited table
    to compute Pearson correlations of each feature against the bioassay results.
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: IRkernel
  role: R kernel for Jupyter required to execute the bioactive molecular networking
    notebook.
  repo: https://github.com/IRkernel/IRkernel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:79abacac009b432baf7525b218697c10f70f14875f605adcab96a14d97a9e05a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - IRkernel
  - Jupyter notebook
  - MZmine2
  - OpenMS
  - Optimus
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bioassay-data-integration-into-feature-tables

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Augments the aligned LC-MS/MS feature quantification table (e.g. features_quantification_matrix.csv) with the bioassay measurement recorded for each fraction (e.g. antiviral activity), yielding features_quantification_matrix_edited_bioactivity.csv so that the Bioactive_Molecular_Networks Jupyter notebook can compute Pearson correlations between every detected feature and the bioactivity outcome.

## When to use

Use when a feature quantification matrix from MZmine2 or Optimus must be merged with per-sample bioassay values (e.g. antiviral EC50 or inhibition) prior to running the bioactive molecular networking Pearson-correlation notebook.

## When NOT to use

- The input is already a bioactivity-augmented table (features_quantification_matrix_edited_bioactivity.csv) — re-appending bioassay columns would duplicate or overwrite the correlation input.
- The goal is plain feature-based molecular networking without bioactivity mapping; the standard GNPS FBMN workflow consumes the unedited features_quantification_matrix.csv and .MGF directly.
- Bioassay data are not matched to the LC-MS/MS-analyzed fractions (different samples or incompatible identifiers), since the Pearson correlation would then pair feature intensities with the wrong bioactivity values.

## Inputs

- features_quantification_matrix.csv — aligned feature list (row ID, row m/z, row retention time, Peak area) exported from MZmine2
- Per-sample bioassay measurement values (e.g. antiviral activity) for each LC-MS/MS-analyzed fraction
- Optional: .MGF spectral summary file and aligned feature table for the downstream GNPS FBMN job when Optimus was used

## Outputs

- features_quantification_matrix_edited_bioactivity.csv — feature table augmented with the bioassay column per sample, ready for the Bioactive_molecular_networks_v1.0.ipynb notebook

## How to apply

After exporting the aligned feature quantification table from MZmine2 (via the 'Export to CSV' module with 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area') or Optimus, open features_quantification_matrix.csv and append one new column per fraction holding that sample's bioassay result, producing features_quantification_matrix_edited_bioactivity.csv. The bioassay values must be aligned by sample identifier so each feature row's intensity vector and the corresponding bioactivity vector are correctly paired, because the notebook computes the Pearson correlation score and its significance for every detected feature against the bioassay results. When the raw LC-MS/MS was instead processed with Optimus, activate the 'MS/MS filter' option and keep the .MGF spectral summary alongside the quantification table, since both are needed downstream for GNPS feature-based molecular networking. This edited table plus the .MGF file are the two inputs the Jupyter notebook (R kernel via IRkernel) and the subsequent GNPS Data Analysis job consume, so file naming and column ordering should follow the repository conventions. Validation is straightforward: every sample present in the quantification matrix should have a non-missing bioassay value, and the edited file should retain the original feature rows and peak-area columns unchanged.

## Related tools

- **MZmine2** (Produces the feature quantification table (features_quantification_matrix.csv) via the 'Export to CSV' module that the bioassay values are appended to; version 2.31 was used in the study.) — http://mzmine.github.io/
- **Optimus** (Alternative LC-MS/MS processing tool (KNIME/OpenMS based) that generates an equivalent quantification table; the 'MS/MS filter' option must be activated.) — https://github.com/MolecularCartography/Optimus
- **Jupyter notebook** (Bioactive_molecular_networks_v1.0.ipynb (R kernel) consumes the edited table to compute Pearson correlations of each feature against the bioassay results.) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **IRkernel** (R kernel for Jupyter required to execute the bioactive molecular networking notebook.) — https://github.com/IRkernel/IRkernel

## Evaluation signals

- The edited file features_quantification_matrix_edited_bioactivity.csv contains one bioassay value for every sample that appears as a column in the original features_quantification_matrix.csv.
- Original feature rows and their row ID / row m/z / row retention time / Peak area columns are unchanged relative to the unedited export.
- Sample identifiers used for the bioassay columns match the sample names in the MZmine2/Optimus export exactly (no re-ordering or renaming).
- The downstream Jupyter notebook runs without missing-value errors and outputs features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv with per-feature Pearson r and corrected p-values.
- GNPS feature-based molecular networking job is launched with the .MGF and the edited feature table, and the resulting Cytoscape network flags bioactive nodes at r2 > 0.8 and p < 0.02.

## Limitations

- The method assumes a scalar bioassay readout per fraction; multi-endpoint or dose–response assays would need explicit handling before appending columns.
- Pearson correlation significance is computed in the notebook with correction, so the bioassay values should be on a scale where linear correlation is meaningful (the study used antiviral assay results).
- Optimus-processed data require the 'MS/MS filter' option to be activated to retain MS/MS-bearing features for the GNPS workflow.
- MZmine2 exports must include exactly the row ID, row m/z, row retention time, and Peak area columns for the notebook and GNPS ingestion to work.

## Evidence

- [methods] Feature table must include bioassay values per sample before notebook correlation: "Add the value of the bioassay experiments for each sample analyzed by LC-MS/MS in the table (features_quantification_matrix.csv to features_quantification_matrix_edited_bioactivity.csv)"
- [methods] Notebook computes Pearson correlation of every feature against bioassay results: "The step 2 will calculate the Pearson correlation score and the its significance for every detected features, against the result of the bioassay"
- [methods] MZmine2 export columns required for the quantification matrix: "a table exported with the 'Export to CSV' module in MZmine, that includes 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'"
- [methods] Optimus requires MS/MS filter activation for GNPS-compatible output: "Note that the option 'MS/MS filter' has to be activated."
- [readme] README describes the workflow integrating MS/MS molecular networking and bioassay-guided fractionation: "The bioactive molecular network workflow integrates MS/MS molecular networking and bioassay-guided fractionation into the concept of bioactive molecular networking."
