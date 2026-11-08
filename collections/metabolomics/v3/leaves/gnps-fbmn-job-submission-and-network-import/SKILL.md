---
name: gnps-fbmn-job-submission-and-network-import
description: Use when an LC-MS/MS dataset has already been processed to a feature
  quantification table (row ID, row m/z, row retention time, peak area) plus a .MGF
  MS/MS spectral summary, and the goal is a GNPS FBMN job and a Cytoscape network
  in which bioactive nodes can be flagged.
when_to_use_negative:
- Raw LC-MS/MS data that has not yet been processed in MZmine2/Optimus into a feature
  quantification table and .MGF spectral summary — run Step 1 of the workflow first
- Feature lists lacking MS/MS spectra, since the required 'Keep only features with
  MS/MS scan (GNPS)' row filter removes features without MS/MS scans
- Datasets with no bioassay values per fraction, as the bioactivity-correlation layer
  (Pearson r2 > 0.8, p < 0.02 selection) cannot be mapped onto the network
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS web-platform
  role: Hosts the feature-based molecular networking (FBMN) Data Analysis job to which
    the .MGF and feature table are submitted
  repo: http://gnps.ucsd.edu
- name: MZmine2
  role: Exports the .MGF spectral summary and the feature quantification table (row
    ID, row m/z, row retention time, peak area) that are submitted to the FBMN job
  repo: http://mzmine.github.io/
- name: Optimus
  role: Alternative (OpenMS-based, KNIME) tool to MZmine2 for producing the feature
    quantification table; the 'MS/MS filter' option must be activated
  repo: https://github.com/MolecularCartography/Optimus
- name: Cytoscape
  role: Imports the GNPS FBMN network and the correlation output table; used for Select
    (r2 > 0.8, p < 0.02), bypass node style and pie charts of ion distribution (minimum
    version 3.4)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:4c48177accdfbce36c4cde75f6d073d886b853e6afb862e7c6deaa45d27f4f34
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS web-platform
  - MZmine2
  - OpenMS
  - Optimus
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# gnps-fbmn-job-submission-and-network-import

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Submit MZmine2/Optimus-derived .MGF spectral summaries and aligned feature tables to the GNPS web-platform as a feature-based molecular networking (FBMN) Data Analysis job, then import the resulting network into Cytoscape to expose bioactivity-correlated nodes. This is the networking/visualization half of the bioactive molecular networking workflow.

## When to use

Use when an LC-MS/MS dataset has already been processed to a feature quantification table (row ID, row m/z, row retention time, peak area) plus a .MGF MS/MS spectral summary, and the goal is a GNPS FBMN job and a Cytoscape network in which bioactive nodes can be flagged.

## When NOT to use

- Raw LC-MS/MS data that has not yet been processed in MZmine2/Optimus into a feature quantification table and .MGF spectral summary — run Step 1 of the workflow first
- Feature lists lacking MS/MS spectra, since the required 'Keep only features with MS/MS scan (GNPS)' row filter removes features without MS/MS scans
- Datasets with no bioassay values per fraction, as the bioactivity-correlation layer (Pearson r2 > 0.8, p < 0.02 selection) cannot be mapped onto the network

## Inputs

- .MGF MS/MS spectral summary file exported from MZmine2 (v2.31) or Optimus
- features_quantification_matrix.csv — aligned feature table with row ID, row m/z, row retention time and peak area
- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv — Jupyter notebook (R kernel) Pearson correlation output

## Outputs

- GNPS FBMN Data Analysis job results (molecular network of features linked by MS/MS spectral similarity)
- Molecular network imported into Cytoscape (>= 3.4) as a visual graph
- Cytoscape visualization with bioactive nodes flagged by r2 > 0.8 and p-value < 0.02, bypass node styling and pie charts of ion distribution across samples

## How to apply

First ensure the MZmine2 (v2.31) export used the required 'Peak list row filter' options ('Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number') and the 'Export to CSV' module with row ID, row m/z, row retention time and peak area. Submit the .MGF spectral summary and the feature quantification table to http://gnps.ucsd.edu as a feature-based molecular networking (FBMN) Data Analysis job — note that MS-Cluster is not used in this workflow, unlike classical molecular networking. When the job completes, import the resulting molecular network into Cytoscape (minimum version 3.4). Then import the Jupyter notebook correlation output (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv) and use the Select function with the significance threshold r2 > 0.8 and p-value < 0.02 to flag bioactive nodes. Finally, apply bypass node style (size/shape) to the selected nodes and display pie charts of ion distribution across the analyzed fractions; validate that significant-correlation nodes correspond to the reported bioactive leads (e.g. the antiviral candidates in the Euphorbia dendroides extract that were missed by classical bioassay-guided fractionation).

## Related tools

- **GNPS web-platform** (Hosts the feature-based molecular networking (FBMN) Data Analysis job to which the .MGF and feature table are submitted) — http://gnps.ucsd.edu
- **MZmine2** (Exports the .MGF spectral summary and the feature quantification table (row ID, row m/z, row retention time, peak area) that are submitted to the FBMN job) — http://mzmine.github.io/
- **Optimus** (Alternative (OpenMS-based, KNIME) tool to MZmine2 for producing the feature quantification table; the 'MS/MS filter' option must be activated) — https://github.com/MolecularCartography/Optimus
- **Cytoscape** (Imports the GNPS FBMN network and the correlation output table; used for Select (r2 > 0.8, p < 0.02), bypass node style and pie charts of ion distribution (minimum version 3.4))

## Evaluation signals

- The MZmine2 export complied with the two mandatory 'Peak list row filter' options ('Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number') before job submission
- The GNPS FBMN job completes on http://gnps.ucsd.edu without errors and returns a network whose nodes carry the row IDs, m/z and retention times of the submitted feature table
- The correlation output table (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv) imports into Cytoscape and the Select function with r2 > 0.8 and p-value < 0.02 flags a non-empty set of bioactive nodes
- Pie charts of ion distribution across samples are rendered for network nodes via bypass node style in Cytoscape (>= 3.4)
- The flagged significant-correlation nodes correspond to the reported bioactive leads (antiviral candidates of the Euphorbia dendroides extract) missed by classical bioassay-guided fractionation

## Limitations

- Only features with acquired MS/MS scans are retained for networking; the required MZmine2 'Peak list row filter' option 'Keep only features with MS/MS scan (GNPS)' discards the rest
- Unlike classical GNPS molecular networking, MS-Cluster is not used in the FBMN Data Analysis workflow
- Cytoscape minimum version 3.4 is required for network import and styling
- Bioactivity mapping depends on the prior Pearson correlation step (Jupyter notebook with R/IRkernel); without it the network cannot flag bioactive nodes

## Evidence
