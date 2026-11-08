---
name: multiple-testing-pvalue-correction
description: Use when a feature-by-sample quantification table (e.g., from MZmine2
  with row ID, row m/z, row retention time, peak area) is annotated with bioassay
  values per LC-MS/MS fraction and per-feature correlation significance is needed
  before mapping onto molecular networks.
when_to_use_negative:
- The input table is a raw, unannotated MZmine2 export (features_quantification_matrix.csv)
  that does not yet contain the bioassay values per sample — the bioassay annotation
  must be added first.
- The goal is pure molecular networking/dereplication without bioactivity mapping
  (Step 3 GNPS FBMN alone), so no feature–bioactivity correlation significance is
  required.
- Data are spatial/2D-3D metabolomics cartography runs processed by Optimus without
  paired bioassay measurements per fraction.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Jupyter Notebook
  role: Hosts the Step 2 notebook in which Pearson correlation scores and their significance
    with multiple-testing correction are computed for every detected feature.
- name: R
  role: Language implementing the Pearson correlation and p-value calculations inside
    the notebook; requires a local R installation.
  repo: https://www.r-project.org/
- name: IRkernel
  role: Native R kernel that lets the Jupyter notebook execute the R correlation/p-value
    code (install IRkernel and register it with IRkernel::installspec()).
  repo: https://github.com/IRkernel/IRkernel
- name: MZmine2
  role: Upstream producer of the feature quantification table (Export to CSV with
    row ID, row m/z, row retention time, peak area) that is the input to this correction
    step.
  repo: http://mzmine.github.io/
- name: Anaconda
  role: Recommended environment manager for setting up the Jupyter/R environment in
    which the Step 2 notebook runs.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:fe95000e12cd96d1440ac6fa67add8be4c5a805442a9f4388ee44a082a659ec7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Anaconda
  - IRkernel
  - Jupyter Notebook
  - MZmine2
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# multiple-testing-pvalue-correction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Statistical validation step within the bioactive molecular networking Jupyter notebook (Step 2): computes the significance (p-value) of every feature–bioactivity Pearson correlation and applies multiple-testing p-value correction so that only statistically supported bioactive candidates become Cytoscape node attributes.

## When to use

Use when a feature-by-sample quantification table (e.g., from MZmine2 with row ID, row m/z, row retention time, peak area) is annotated with bioassay values per LC-MS/MS fraction and per-feature correlation significance is needed before mapping onto molecular networks.

## When NOT to use

- The input table is a raw, unannotated MZmine2 export (features_quantification_matrix.csv) that does not yet contain the bioassay values per sample — the bioassay annotation must be added first.
- The goal is pure molecular networking/dereplication without bioactivity mapping (Step 3 GNPS FBMN alone), so no feature–bioactivity correlation significance is required.
- Data are spatial/2D-3D metabolomics cartography runs processed by Optimus without paired bioassay measurements per fraction.

## Inputs

- features_quantification_matrix_edited_bioactivity.csv — MZmine2-exported feature quantification table with row ID, row m/z, row retention time, peak area, plus bioassay values for each LC-MS/MS-analyzed sample
- Bioassay result vector of the analyzed fractions (embedded in the annotated CSV)

## Outputs

- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv — per-feature Pearson correlation scores and multiple-testing-corrected p-values, formatted for Cytoscape import as a node attribute table

## How to apply

Load the bioassay-annotated feature quantification table (features_quantification_matrix_edited_bioactivity.csv) and verify it contains row ID, row m/z, row retention time, peak area columns plus the bioassay values per sample. Transpose the matrix so detected features are rows and the LC-MS/MS-analyzed fractions are columns, then, for every detected feature, compute the Pearson correlation coefficient between its relative-abundance profile across fractions and the bioassay result vector of those fractions, implemented in R inside the Jupyter notebook (R kernel via IRkernel). For each feature–bioactivity correlation, compute its significance (p-value) and apply a multiple-testing correction across all detected features, since thousands of simultaneous correlations inflate false positives. Write the resulting table to features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv, formatted as a node attribute table for later import into Cytoscape.

## Related tools

- **Jupyter Notebook** (Hosts the Step 2 notebook in which Pearson correlation scores and their significance with multiple-testing correction are computed for every detected feature.)
- **R** (Language implementing the Pearson correlation and p-value calculations inside the notebook; requires a local R installation.) — https://www.r-project.org/
- **IRkernel** (Native R kernel that lets the Jupyter notebook execute the R correlation/p-value code (install IRkernel and register it with IRkernel::installspec()).) — https://github.com/IRkernel/IRkernel
- **MZmine2** (Upstream producer of the feature quantification table (Export to CSV with row ID, row m/z, row retention time, peak area) that is the input to this correction step.) — http://mzmine.github.io/
- **Anaconda** (Recommended environment manager for setting up the Jupyter/R environment in which the Step 2 notebook runs.)

## Examples

```
install.packages('IRkernel'); IRkernel::installspec()
```

## Evaluation signals

- The output CSV is the transposed matrix: detected features as rows, LC-MS/MS-analyzed fractions as columns, with one correlation score and one corrected p-value per feature.
- Every corrected p-value is ≥ its corresponding raw p-value, and all detected features (not a subset) carry both statistics.
- The input table retains the required MZmine2 export columns (row ID, row m/z, row retention time, peak area) before transposition.
- The output file is directly importable into Cytoscape as a node attribute table for the subsequent GNPS FBMN network (Step 3).
- The corrected significance statistics successfully expose potentially bioactive molecules consistent with fraction bioactivity levels (validated conceptually by the Euphorbia dendroides antiviral case).

## Limitations

- The workflow requires a separate R installation (and IRkernel registration) inside Jupyter; the notebook will not run on a Python-only kernel.
- Significance relies on the bioassay values being added manually per sample before Step 2; missing or misaligned bioassay entries per fraction will invalidate correlations and p-values.
- Statistical significance of a feature–bioactivity correlation does not confirm compound identity — MS/MS dereplication and validation via GNPS molecular networking (Step 3) are still needed prior to isolation.

## Evidence
