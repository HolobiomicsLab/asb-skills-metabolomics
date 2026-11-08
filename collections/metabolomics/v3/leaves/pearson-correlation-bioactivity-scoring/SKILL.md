---
name: pearson-correlation-bioactivity-scoring
description: Use when you have a bioassay-annotated feature quantification table (e.g.
  features_quantification_matrix_edited_bioactivity.
when_to_use_negative:
- The input is already a bioactivity-scored table (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv);
  re-running the correlation would be redundant.
- The LC-MS/MS-analyzed fractions are not paired with a bioassay result, so there
  is no bioactivity vector to correlate against.
- The data are not LC-MS/MS feature intensities (e.g. direct-infusion spectra without
  fraction-level quantification), which the Pearson correlation scoring is not designed
  for.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jupyter Notebook
  role: Provides the notebook environment in which the Pearson correlation and p-value
    correction step is run; the code is released as a Jupyter notebook for reproducibility.
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: R
  role: Implements the Pearson correlation calculation and p-value correction within
    the notebook; the notebook requires R installed (with IRkernel to run it under
    Jupyter).
  repo: https://github.com/IRkernel/IRkernel
- name: Anaconda
  role: Convenient environment manager used to install the Python and R dependencies
    the notebook needs.
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:3bec0acf0d89e98bdbfc2617cdf9b7212de6264821ce236e032085e80356ab25
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks
  tools:
  - Anaconda
  - Jupyter Notebook
  - MZmine2
  - OpenMS
  - Optimus
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# pearson-correlation-bioactivity-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes, for every detected feature, a Pearson correlation coefficient and its p-value between the feature's relative abundance profile across LC-MS/MS-analyzed fractions and the corresponding bioassay result vector, producing a bioactivity-score attribute table that is later overlaid onto a GNPS molecular network in Cytoscape.

## When to use

Use when you have a bioassay-annotated feature quantification table (e.g. features_quantification_matrix_edited_bioactivity.csv) exported from MZmine2 or Optimus and you want to score each feature's association with the measured bioactivity of each fraction before mapping it onto a molecular network.

## When NOT to use

- The input is already a bioactivity-scored table (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv); re-running the correlation would be redundant.
- The LC-MS/MS-analyzed fractions are not paired with a bioassay result, so there is no bioactivity vector to correlate against.
- The data are not LC-MS/MS feature intensities (e.g. direct-infusion spectra without fraction-level quantification), which the Pearson correlation scoring is not designed for.

## Inputs

- features_quantification_matrix_edited_bioactivity.csv
- aligned feature quantification table exported from MZmine2 or Optimus
- bioassay results for each LC-MS/MS-analyzed fraction

## Outputs

- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv

## How to apply

First, load the feature quantification table annotated with bioassay values (features_quantification_matrix_edited_bioactivity.csv) and transpose it so that detected features become rows and the LC-MS/MS-analyzed fractions become columns. Then, for every feature, calculate the Pearson correlation coefficient between its intensity profile across the fractions and the bioassay result vector of those fractions, implemented in R within the Jupyter notebook (R kernel via IRkernel). Next, compute the significance (p-value) of each feature-bioactivity correlation and apply multiple-testing p-value correction. Finally, write the resulting table to features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv, formatted for import into Cytoscape as a node attribute table.

## Related tools

- **Jupyter Notebook** (Provides the notebook environment in which the Pearson correlation and p-value correction step is run; the code is released as a Jupyter notebook for reproducibility.) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **R** (Implements the Pearson correlation calculation and p-value correction within the notebook; the notebook requires R installed (with IRkernel to run it under Jupyter).) — https://github.com/IRkernel/IRkernel
- **Anaconda** (Convenient environment manager used to install the Python and R dependencies the notebook needs.)

## Examples

```
Rscript -e "df <- read.csv('features_quantification_matrix_edited_bioactivity.csv'); res <- apply(df[,-1], 1, function(x) cor.test(x, bioassay_vector)); write.csv(res, 'features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv')"
```

## Evaluation signals

- Verify the output file features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv exists and contains a correlation score and corrected p-value for every detected feature in the input table.
- Check that p-values are corrected (multiple-testing) and that the significance threshold applied downstream (e.g. r2 > 0.8 and p-value < 0.02) identifies the same set of candidate bioactive nodes expected by the workflow.
- Confirm the transposed table's row/column orientation: features as rows, fractions as columns, matching the node attribute format expected by Cytoscape.
- Cross-check a manually computed Pearson r and p-value for a few features against the notebook output to confirm the implementation is correct.

## Limitations

- Pearson correlation assumes a linear relationship between feature abundance and bioactivity; non-linear associations may be missed.
- Correlation only highlights features whose abundance pattern tracks the bioassay result; it does not confirm causality or identity of the bioactive compound.
- The notebook requires R and IRkernel to be installed; failure to install IRkernel will prevent the notebook from running.
- No correlation-based filtering is applied at this step alone — the r2 > 0.8 / p-value < 0.02 threshold is applied later in Cytoscape, so unfiltered output may contain many low-significance features.

## Evidence
