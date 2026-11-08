---
name: mzmine-csv-and-mgf-export-configuration
description: Use when raw fractionated LC-MS/MS files have been feature-detected,
  deconvoluted and aligned in MZmine2 (v2.31) and a feature table plus MS/MS spectra
  must be exported for GNPS FBMN or bioactive molecular networking.
when_to_use_negative:
- The feature quantification table already exists and matches the TEMPLATE_FILES format
  — proceed directly to bioactivity-score editing or the Jupyter notebook step
- The processing is already done in Optimus, where the 'MS/MS filter' option replaces
  MZmine2's 'Peak list row filter' and export path
- Direct-infusion or imaging data that will not feed the GNPS feature-based molecular
  networking workflow, since the MS/MS-scan-only filter would discard most features
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine2
  role: Performs the 'Peak list row filter' (keep only features with MS/MS scan (GNPS);
    reset the peak number) and the 'Export to CSV' module that produces the feature
    quantification table and the .MGF spectral summary
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative OpenMS/KNIME-based workflow for the same LC-MS/MS processing;
    its 'MS/MS filter' option must be activated in place of MZmine2's row filter
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the underlying LC-MS feature detection and quantification algorithms
    executed by Optimus
  repo: http://www.openms.de
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:cb02ca01d67b73b110831738e247e14815ab59afdc00f1ed2fba2bcbefcd2fed
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MZmine2
  - OpenMS
  - Optimus
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# MZmine CSV and MGF export configuration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Configures MZmine2's export modules so that LC-MS/MS feature processing yields both a feature quantification table (features_quantification_matrix.csv) and an .MGF file with MS/MS spectral summaries, the paired inputs required for downstream bioactivity scoring and GNPS feature-based molecular networking.

## When to use

Use when raw fractionated LC-MS/MS files have been feature-detected, deconvoluted and aligned in MZmine2 (v2.31) and a feature table plus MS/MS spectra must be exported for GNPS FBMN or bioactive molecular networking.

## When NOT to use

- The feature quantification table already exists and matches the TEMPLATE_FILES format — proceed directly to bioactivity-score editing or the Jupyter notebook step
- The processing is already done in Optimus, where the 'MS/MS filter' option replaces MZmine2's 'Peak list row filter' and export path
- Direct-infusion or imaging data that will not feed the GNPS feature-based molecular networking workflow, since the MS/MS-scan-only filter would discard most features

## Inputs

- MZmine2 (v2.31) peak list after feature detection, deconvolution and alignment on fractionated LC-MS/MS raw files
- Raw LC-MS/MS files of bioassay-guided fractionation samples (e.g. Euphorbia dendroides fractions)

## Outputs

- features_quantification_matrix.csv with columns row ID, row m/z, row retention time, and Peak area
- .MGF file containing the MS/MS spectral summary for GNPS FBMN

## How to apply

After feature detection, deconvolution and alignment in MZmine2 version 2.31, first apply the 'Peak list row filter' module with the two required options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number'; the MS/MS-scan filter guarantees every retained feature carries an MS/MS spectrum for later GNPS library matching, and resetting the peak number keeps row identifiers consistent. Then run the 'Export to CSV' module ensuring the output includes 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', and also generate the .MGF file containing the MS/MS spectral summary. Save the table as features_quantification_matrix.csv, which holds the aligned list of features and their intensity across the analyzed fractions (e.g. Euphorbia dendroidest fractions). The equivalent Optimus path uses the OpenMS-based KNIME workflow with the 'MS/MS filter' option activated to achieve the same filtering. Validate the output against the TEMPLATE_FILES folder of the Bioactive_Molecular_Networks GitHub repository before handing the CSV and MGF to the bioactivity-scoring step (Pearson correlation via Jupyter notebook) and the GNPS FBMN job.

## Related tools

- **MZmine2** (Performs the 'Peak list row filter' (keep only features with MS/MS scan (GNPS); reset the peak number) and the 'Export to CSV' module that produces the feature quantification table and the .MGF spectral summary) — http://mzmine.github.io/
- **Optimus** (Alternative OpenMS/KNIME-based workflow for the same LC-MS/MS processing; its 'MS/MS filter' option must be activated in place of MZmine2's row filter) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the underlying LC-MS feature detection and quantification algorithms executed by Optimus) — http://www.openms.de

## Evaluation signals

- The exported CSV contains exactly the required columns: 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'
- The CSV structure matches the template file format provided in the TEMPLATE_FILES folder of the Bioactive_Molecular_Networks GitHub repository
- Every retained feature row has an associated MS/MS scan, because the 'Keep only features with MS/MS scan (GNPS)' filter option was applied
- The companion .MGF file exists and contains an MS/MS spectral summary for the same feature set (row IDs consistent with the CSV after 'Reset the peak number')
- Feature intensities are present across all fractions analyzed by LC-MS/MS, allowing the table to be extended with bioassay values (features_quantification_matrix.csv → features_quantification_matrix_edited_bioactivity.csv)

## Limitations

- The two 'Peak list row filter' options are explicitly required; skipping 'Keep only features with MS/MS scan (GNPS)' leaves features without spectra that GNPS FBMN cannot use, and skipping 'Reset the peak number' can desynchronize row IDs between CSV and MGF
- MZmine2 version 2.31 is the version specified by the protocol; other versions may differ in module behavior
- If Optimus is used instead, the 'MS/MS filter' option must be activated — the MZmine2 filter names do not apply directly

## Evidence
