---
name: mass-spectrometry-data-preprocessing-for-bioactivity-workflow
description: Use when raw LC-MS/MS files of bioassay-guided fractionation fractions
  must be converted into the GNPS FBMN feature quantification table (features_quantification_matrix.csv)
  and its .MGF MS/MS summary before bioactivity scoring.
when_to_use_negative:
- A feature quantification table already exists — proceed directly to Step 2 (Jupyter
  notebook Pearson correlation) instead of re-preprocessing raw files
- The dataset lacks MS/MS scans — the required 'Keep only features with MS/MS scan
  (GNPS)' filter would remove every feature from the table
- The goal is running the GNPS FBMN Data Analysis job itself (Step 3) — that step
  consumes this skill's CSV/MGF outputs rather than producing them
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MZmine2
  role: GUI feature detection, deconvolution, and alignment of fraction LC-MS/MS files
    (version 2.31); applies the 'Peak list row filter' (Keep only features with MS/MS
    scan (GNPS), Reset the peak number) and exports the feature table via 'Export
    to CSV' plus the .MGF MS/MS summary
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative KNIME workflow for LC-MS feature detection, quantification, and
    alignment of fractions; the 'MS/MS filter' option must be activated to keep only
    MS/MS-acquired features for FBMN
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Supplies the underlying state-of-the-art LC-MS feature detection and quantification
    algorithms executed inside the Optimus pipeline
  repo: http://www.openms.de
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:4cd0fce3867028ab63ec3ae2d85044dfda7ba0b8568fe084ba2f554e946bd7e1
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MZmine2
  - OpenMS
  - Optimus
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-data-preprocessing-for-bioactivity-workflow

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Step 1 of the bioactive molecular networking workflow (Nothias et al. 2018): preprocess fractionated-extract LC-MS/MS data with MZmine2 2.31 or Optimus (OpenMS/KNIME) into the GNPS FBMN feature quantification table plus an .MGF MS/MS spectral summary that feed the downstream bioactivity-score correlation.

## When to use

Use when raw LC-MS/MS files of bioassay-guided fractionation fractions must be converted into the GNPS FBMN feature quantification table (features_quantification_matrix.csv) and its .MGF MS/MS summary before bioactivity scoring.

## When NOT to use

- A feature quantification table already exists — proceed directly to Step 2 (Jupyter notebook Pearson correlation) instead of re-preprocessing raw files
- The dataset lacks MS/MS scans — the required 'Keep only features with MS/MS scan (GNPS)' filter would remove every feature from the table
- The goal is running the GNPS FBMN Data Analysis job itself (Step 3) — that step consumes this skill's CSV/MGF outputs rather than producing them

## Inputs

- Raw LC-MS/MS acquisition files for each fraction of the bioactive extract (e.g., the fractionated Euphorbia dendroides extract)
- Experimental design file for Optimus runs (optional, per the Optimus README)

## Outputs

- features_quantification_matrix.csv containing the aligned list of features and their intensity across the analyzed fractions, with columns 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'
- .MGF file containing the MS/MS spectral summary for downstream GNPS FBMN processing

## How to apply

Run MZmine2 (version 2.31) or Optimus (a KNIME workflow executing OpenMS feature detection/quantification algorithms) over the raw LC-MS/MS files of all fractions, following the GNPS feature-based molecular networking (FBMN) steps of feature detection, deconvolution, and alignment. In MZmine2 apply the 'Peak list row filter' with both required options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number'; in Optimus activate the 'MS/MS filter' instead. Export results with MZmine2's 'Export to CSV' module including 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', and generate the .MGF file containing the MS/MS spectral summary. Retaining only MS/MS-acquired features is essential because each surviving row must later map to a GNPS FBMN molecular networking node. Save the output as features_quantification_matrix.csv, verify it matches the template in the TEMPLATE_FILES folder of the Bioactive_Molecular_Networks GitHub repository, and only then add per-fraction bioassay values for Step 2.

## Related tools

- **MZmine2** (GUI feature detection, deconvolution, and alignment of fraction LC-MS/MS files (version 2.31); applies the 'Peak list row filter' (Keep only features with MS/MS scan (GNPS), Reset the peak number) and exports the feature table via 'Export to CSV' plus the .MGF MS/MS summary) — http://mzmine.github.io/
- **Optimus** (Alternative KNIME workflow for LC-MS feature detection, quantification, and alignment of fractions; the 'MS/MS filter' option must be activated to keep only MS/MS-acquired features for FBMN) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Supplies the underlying state-of-the-art LC-MS feature detection and quantification algorithms executed inside the Optimus pipeline) — http://www.openms.de

## Evaluation signals

- The exported CSV contains the required columns 'row ID', 'row m/z', 'row retention time', and 'Peak area' and matches the template file format in the TEMPLATE_FILES folder of the Bioactive_Molecular_Networks GitHub repository
- The companion .MGF file is present and contains the MS/MS spectral summary for the retained features
- Every row in the table corresponds to a feature with an MS/MS scan, confirming the 'Keep only features with MS/MS scan (GNPS)' (or Optimus 'MS/MS filter') option was applied
- After 'Reset the peak number', row IDs are re-numbered contiguously over the aligned feature set
- Intensity columns cover every fraction analyzed by LC-MS/MS so each feature can later be correlated against per-fraction bioassay values

## Limitations

- Version-pinned: the paper specifies MZmine2 version 2.31; module names and export options may differ in other MZmine releases
- Features without acquired MS/MS spectra are deliberately dropped by the required filter, so bioactive compounds lacking fragmentation are excluded from the molecular network
- Optimus requires KNIME Analytics Platform plus Python 2.7 on a 64-bit OS (per its README), and its temporary files are not deleted automatically unless the cleanup option is used

## Evidence

- [readme] Optimus relies on OpenMS algorithms: [evidence span withheld — anchor status: external_doc]
