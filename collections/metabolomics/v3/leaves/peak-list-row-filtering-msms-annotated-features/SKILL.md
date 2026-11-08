---
name: peak-list-row-filtering-msms-annotated-features
description: 'Use when preparing a feature table exported from MZmine2 or Optimus
  for bioactive molecular networking: Step 1 requires a features_quantification_matrix.csv
  limited to features with an acquired MS/MS scan.'
when_to_use_negative:
- If the feature table is already restricted to MS/MS-annotated features (e.g. prepared
  via Optimus with the MS/MS filter already applied), rerunning the MZmine2 filter
  step would be redundant.
- If the analysis targets MS1-only features without any MS/MS spectra (e.g. for purely
  quantitative profiling without molecular networking), this skill would discard the
  features of interest.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MZmine2
  role: Performs feature detection, deconvolution, alignment and the 'Peak list row
    filter' step with 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak
    number' options; exports the filtered table to CSV and generates the MGF file.
  repo: http://mzmine.github.io/
- name: Optimus
  role: OpenMS/KNIME-based workflow that provides the 'MS/MS filter' option as the
    equivalent of the MZmine2 row filter for retaining only MS/MS-annotated features.
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Underlying LC-MS feature detection and quantification algorithms used by Optimus.
  repo: http://www.openms.de
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:04bd036e9a032150d9e71f788d587ee6b91924840e92a673c995787b260bf6ac
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MZmine2
  - OpenMS
  - Optimus
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-list-row-filtering-msms-annotated-features

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Prune an LC-MS/MS feature quantification table to retain only MS/MS-annotated features (and optionally renumber rows) before downstream Pearson correlation and GNPS FBMN analysis.

## When to use

Use when preparing a feature table exported from MZmine2 or Optimus for bioactive molecular networking: Step 1 requires a features_quantification_matrix.csv limited to features with an acquired MS/MS scan.

## When NOT to use

- If the feature table is already restricted to MS/MS-annotated features (e.g. prepared via Optimus with the MS/MS filter already applied), rerunning the MZmine2 filter step would be redundant.
- If the analysis targets MS1-only features without any MS/MS spectra (e.g. for purely quantitative profiling without molecular networking), this skill would discard the features of interest.

## Inputs

- Aligned LC-MS/MS feature list from MZmine2 or Optimus (peak table with row ID, m/z, retention time, peak area)
- Raw or processed fraction LC-MS/MS files from bioassay-guided fractionated extracts (e.g. Euphorbia dendroides fractions)

## Outputs

- features_quantification_matrix.csv (MS/MS-annotated features only, with row ID, row m/z, row retention time, peak area columns)
- MGF file containing the MS/MS spectral summary for downstream GNPS FBMN

## How to apply

In MZmine2 (version 2.31), after feature detection, deconvolution and alignment, add a 'Peak list row filter' step with two mandatory options: 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number'. This drops rows whose MS1-only features lack MS/MS spectra, which is necessary because the downstream GNPS feature-based molecular networking job and the Jupyter notebook bioactivity correlation step operate on MS/MS-annotated features. In Optimus (built on OpenMS/KNIME), the equivalent operation is activating the 'MS/MS filter' option in the workflow. Then export the filtered table via MZmine2's 'Export to CSV' module, ensuring the output includes 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', and also generate the .MGF file containing the MS/MS spectral summary. Rationale: the feature quantification table is the direct input for Pearson correlation against bioassay values, so unannotated MS1 features would either fail in FBMN or dilute bioactivity scoring.

## Related tools

- **MZmine2** (Performs feature detection, deconvolution, alignment and the 'Peak list row filter' step with 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number' options; exports the filtered table to CSV and generates the MGF file.) — http://mzmine.github.io/
- **Optimus** (OpenMS/KNIME-based workflow that provides the 'MS/MS filter' option as the equivalent of the MZmine2 row filter for retaining only MS/MS-annotated features.) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Underlying LC-MS feature detection and quantification algorithms used by Optimus.) — http://www.openms.de

## Evaluation signals

- The exported CSV must contain the columns 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area' exactly as required by the Bioactive_Molecular_Networks template format.
- Every remaining row in the filtered feature table must correspond to a feature with an acquired MS/MS scan (i.e. no MS1-only rows remain).
- Row numbers in the filtered table must be contiguous/reset (the 'Reset the peak number' option), so downstream correlation and FBMN IDs align correctly.
- The accompanying .MGF file must contain MS/MS spectral summary entries for each retained feature, matching the row IDs in the CSV.
- Feature count should decrease relative to the pre-filter aligned peak list; if it does not, the 'Keep only features with MS/MS scan (GNPS)' option was likely not applied.

## Limitations

- The MZmine2 filter option name 'Keep only features with MS/MS scan (GNPS)' is version-dependent; this procedure was documented for MZmine2 version 2.31.
- Optimus does not provide MS/MS validation of putative annotations (level 2 Metabolomics Standards Initiative); mz-RT matching requires downstream MS/MS validation.
- The workflow does not delete Optimus temporary files automatically; iterative re-execution requires manual cleanup.

## Evidence

- [methods] Peak list row filter options in MZmine2: [evidence span withheld — anchor status: not_found]
- [readme] Optimus README on optional MS/MS exclusion step: [evidence span withheld — anchor status: external_doc]
