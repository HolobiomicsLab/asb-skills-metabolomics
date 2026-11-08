---
name: lc-msms-feature-quantification-table-transposition
description: Use when an aligned LC-MS/MS feature table (row ID, row m/z, row retention
  time, peak area) has been exported and bioassay values per fraction have been added,
  and feature-wise correlation scoring requires features-as-rows orientation.
when_to_use_negative:
- The input is already in feature-as-row orientation from a prior transposition
- The MZmine2 export is missing required columns (Export row ID, Export row m/z, Export
  row retention time, Peak area) — regenerate the export first
- Feature table was not produced by MZmine2 or Optimus and lacks aligned peak areas
  across fractions
tools:
- name: MZmine2
  role: Produces the feature quantification table (via Export to CSV module) that
    is transposed; version 2.31 used
  repo: http://mzmine.github.io/
- name: Jupyter Notebook
  role: Hosts the Step 2 code that performs the transposition before correlation calculation
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: R
  role: Language in which the transposition and subsequent Pearson correlation are
    implemented in the notebook
  repo: https://www.r-project.org/
- name: IRkernel
  role: Enables the R kernel inside Jupyter so the notebook code executes
  repo: https://github.com/IRkernel/IRkernel
- name: Optimus
  role: Alternative to MZmine2 for generating the LC-MS/MS feature quantification
    table
  repo: https://github.com/MolecularCartography/Optimus
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:a2a8ee9968d5b23470ac9008713ac334cf91020d542ebd55d2a28e4b947ff60a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Anaconda
  - IRkernel
  - Jupyter Notebook
  - MZmine2
  - Optimus
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lc-msms-feature-quantification-table-transposition

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Transposes the MZmine2/Optimus-exported feature quantification table (features as columns, fractions as rows) so that detected features become rows and the LC-MS/MS-analyzed fractions become columns, enabling row-wise Pearson correlation of each feature's abundance profile against the bioassay result vector in the bioactive molecular networking workflow.

## When to use

Use when an aligned LC-MS/MS feature table (row ID, row m/z, row retention time, peak area) has been exported and bioassay values per fraction have been added, and feature-wise correlation scoring requires features-as-rows orientation.

## When NOT to use

- The input is already in feature-as-row orientation from a prior transposition
- The MZmine2 export is missing required columns (Export row ID, Export row m/z, Export row retention time, Peak area) — regenerate the export first
- Feature table was not produced by MZmine2 or Optimus and lacks aligned peak areas across fractions

## Inputs

- features_quantification_matrix_edited_bioactivity.csv (aligned feature quantification table from MZmine2 'Export to CSV' with row ID, row m/z, row retention time, peak area, plus bioassay values per sample)
- Bioassay experiment values for each sample analyzed by LC-MS/MS

## Outputs

- Transposed matrix with detected features as rows and LC-MS/MS-analyzed fractions as columns (intermediate to features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv)

## How to apply

After MZmine2 (or Optimus with the MS/MS filter activated) exports the CSV table containing 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', add the bioassay experimental values for each LC-MS/MS-analyzed sample as extra columns (features_quantification_matrix.csv → features_quantification_matrix_edited_bioactivity.csv). In Step 2 of the Jupyter notebook workflow (R kernel via IRkernel), transpose this edited matrix so detected features become rows and the analyzed fractions become columns. The transposed layout is the required input shape for the per-feature Pearson correlation computation against the bioassay result vector, since the correlation is calculated for every detected feature, against the result of the bioassay.

## Related tools

- **MZmine2** (Produces the feature quantification table (via Export to CSV module) that is transposed; version 2.31 used) — http://mzmine.github.io/
- **Jupyter Notebook** (Hosts the Step 2 code that performs the transposition before correlation calculation) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **R** (Language in which the transposition and subsequent Pearson correlation are implemented in the notebook) — https://www.r-project.org/
- **IRkernel** (Enables the R kernel inside Jupyter so the notebook code executes) — https://github.com/IRkernel/IRkernel
- **Optimus** (Alternative to MZmine2 for generating the LC-MS/MS feature quantification table) — https://github.com/MolecularCartography/Optimus

## Examples

```
df.set_index('row ID').T  # transpose features_quantification_matrix_edited_bioactivity.csv: features→rows, fractions→columns (R equivalent: t(df))
```

## Evaluation signals

- The transposed table has one row per detected feature and one column per LC-MS/MS-analyzed fraction
- Row count equals the number of features in features_quantification_matrix_edited_bioactivity.csv; column count equals the number of analyzed fractions plus metadata columns
- All 'Peak area' values are preserved across the transpose (values are only relocated, not recomputed)
- The transposed shape supports row-wise Pearson correlation of each feature against the bioassay result vector without further reshaping
- Output feeds the correlation step that writes features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv

## Limitations

- MZmine2 export must include exactly the options 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area', otherwise the table cannot be used downstream
- Optimus users must activate the 'MS/MS filter' option and set the GNPS 'Peak list row filter' to 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number' before export
- Bioassay values must already be added per sample before transposition; the raw features_quantification_matrix.csv without bioactivity columns is insufficient

## Evidence

- [other] Source cards transposition statement: [evidence span withheld — anchor status: not_found]
