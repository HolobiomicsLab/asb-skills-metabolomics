---
name: lc-msms-feature-detection-and-alignment
description: Use when raw LC-MS/MS files from bioassay-guided fractionation must be
  converted into a feature-level table aligned across fractions before Pearson correlation
  scoring and GNPS FBMN.
when_to_use_negative:
- A feature quantification table (features_quantification_matrix.csv) already exists
  — proceed directly to Step 2 (Jupyter notebook Pearson correlation) instead.
- Data are direct-infusion MS without LC separation or without acquired MS/MS scans
  — the mandatory 'Keep only features with MS/MS scan (GNPS)' filter would discard
  all features.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine2
  role: Primary LC-MS/MS processing tool (version 2.31) for feature detection, deconvolution,
    alignment, Peak list row filtering, CSV export, and .MGF generation
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative KNIME/OpenMS-based LC-MS feature detection and alignment workflow
    with an 'MS/MS filter' option
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Underlying feature detection and quantification algorithms used by Optimus
  repo: http://www.openms.de
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:aac04d77df38facc4acd93032963aa0c278a3ec15bd79d5a1d06aea23495e099
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - MZmine2
  - OpenMS
  - Optimus
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-msms-feature-detection-and-alignment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Step 1 of the bioactive molecular networking workflow: process fractionated LC-MS/MS data with MZmine2 (v2.31) or Optimus (OpenMS-based) to produce an aligned feature quantification table and an MS/MS spectral summary (.MGF) that feed downstream bioactivity correlation scoring.

## When to use

Use when raw LC-MS/MS files from bioassay-guided fractionation must be converted into a feature-level table aligned across fractions before Pearson correlation scoring and GNPS FBMN.

## When NOT to use

- A feature quantification table (features_quantification_matrix.csv) already exists — proceed directly to Step 2 (Jupyter notebook Pearson correlation) instead.
- Data are direct-infusion MS without LC separation or without acquired MS/MS scans — the mandatory 'Keep only features with MS/MS scan (GNPS)' filter would discard all features.

## Inputs

- Raw LC-MS/MS data files from fractionated bioactive extracts (e.g. Euphorbia dendroides fractions)

## Outputs

- features_quantification_matrix.csv — aligned list of features with intensity (Peak area) across all analyzed fractions, including Export row ID, row m/z, row retention time columns
- .MGF file containing the MS/MS spectral summary for downstream GNPS FBMN

## How to apply

Run MZmine2 2.31 (or Optimus on OpenMS/KNIME) over the raw fraction LC-MS/MS files following the GNPS feature-based molecular networking processing steps: feature detection, deconvolution, and alignment. In MZmine2 apply the 'Peak list row filter' with the two mandatory options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number'; in Optimus activate the 'MS/MS filter' option so only features with acquired MS/MS spectra are retained. Export with MZmine2's 'Export to CSV' module selecting 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', and simultaneously generate the .MGF file containing the MS/MS spectral summary. Rationale: keeping only MS/MS-backed features ensures every row in the feature table can later be annotated and correlated in the molecular network, and resetting the peak number keeps row IDs stable for the subsequent Jupyter notebook correlation step.

## Related tools

- **MZmine2** (Primary LC-MS/MS processing tool (version 2.31) for feature detection, deconvolution, alignment, Peak list row filtering, CSV export, and .MGF generation) — http://mzmine.github.io/
- **Optimus** (Alternative KNIME/OpenMS-based LC-MS feature detection and alignment workflow with an 'MS/MS filter' option) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Underlying feature detection and quantification algorithms used by Optimus) — http://www.openms.de

## Examples

```
MZmine2 (GUI, version 2.31): run batch mode with 'Peak list row filter' → 'Keep only features with MS/MS scan (GNPS)' + 'Reset the peak number', then 'Export to CSV' → features_quantification_matrix.csv and .MGF export.
```

## Evaluation signals

- Exported CSV must contain the exact required columns: 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'.
- Every retained feature row must have an associated MS/MS scan (verified by the applied 'Keep only features with MS/MS scan (GNPS)' filter or Optimus 'MS/MS filter').
- The table must match the template file format provided in the TEMPLATE_FILES folder of the Bioactive_Molecular_Networks GitHub repository.
- The companion .MGF file must contain the MS/MS spectral summary for all retained features.
- Feature alignment must be completed across all analyzed fractions, producing a single row per unique m/z-RT feature.

## Limitations

- MZmine2 version 2.31 is the version specified for this workflow; other versions may not expose the required 'Peak list row filter' options identically.
- Feature detection and quantification settings depend on instrument, mass resolution, and LC run-time; large datasets (>100 LC-MS runs) may require more than 2 GB RAM when using Optimus.
- Putative annotation via mz-RT matching (Optimus) is at Metabolomics Standards Initiative level 2 and requires MS/MS validation not provided in Optimus.

## Evidence

- [readme] Optimus feature detection and quantification built on OpenMS: [evidence span withheld — anchor status: not_found]
