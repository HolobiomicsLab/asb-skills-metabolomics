---
name: molecular-feature-abundance-profiling-across-fractions
description: Use when a bioassay-annotated feature quantification table (row ID, row
  m/z, row retention time, peak area, plus bioassay values per sample) exported from
  MZmine2/Optimus is available and each feature's abundance across fractions must
  be correlated with per-fraction bioassay results.
when_to_use_negative:
- The input is not yet a bioassay-annotated feature quantification table — Step 1
  (MZmine2/Optimus processing with the required 'Export row ID', 'Export row m/z',
  'Export row retention time', 'Peak area' CSV export and addition of bioassay values)
  must be completed first
- The analysis goal is the GNPS feature-based molecular networking (FBMN) job itself
  (Step 3) rather than computing per-feature bioactivity correlations
- There are too few bioassayed fractions to support meaningful Pearson correlation
  significance against the bioassay result vector
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Jupyter Notebook
  role: Hosts the Step 2 bioactivity-correlation code, released as a notebook for
    easiness and reproducibility
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks
- name: R / IRkernel
  role: Provides the R kernel in which the Pearson correlation scores and p-values
    for every detected feature are calculated inside the Jupyter notebook
  repo: https://github.com/IRkernel/IRkernel
- name: MZmine2 (version 2.31)
  role: Produces the upstream feature quantification matrix (with peak area export)
    whose features are correlated against bioassay results
  repo: http://mzmine.github.io/
- name: Anaconda
  role: Convenient environment manager for installing Jupyter Notebook, R and IRkernel
    dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:5ecf366d8daa9a45dc9481a2fd48a967f39d92bc2f486cca71815b878346100f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks
  tools:
  - Anaconda
  - Jupyter Notebook
  - MZmine2
  - MZmine2 (version 2.31)
  - R
  - R / IRkernel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# molecular-feature-abundance-profiling-across-fractions

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Correlates the relative abundance profile of every LC-MS/MS-detected feature across bioassay-guided fractions with each fraction's bioassay result, using Pearson correlation and p-value significance computed in R inside the Bioactive Molecular Networks Jupyter notebook. This converts feature abundance patterns into bioactivity scores that can be mapped onto molecular networks.

## When to use

Use when a bioassay-annotated feature quantification table (row ID, row m/z, row retention time, peak area, plus bioassay values per sample) exported from MZmine2/Optimus is available and each feature's abundance across fractions must be correlated with per-fraction bioassay results.

## When NOT to use

- The input is not yet a bioassay-annotated feature quantification table — Step 1 (MZmine2/Optimus processing with the required 'Export row ID', 'Export row m/z', 'Export row retention time', 'Peak area' CSV export and addition of bioassay values) must be completed first
- The analysis goal is the GNPS feature-based molecular networking (FBMN) job itself (Step 3) rather than computing per-feature bioactivity correlations
- There are too few bioassayed fractions to support meaningful Pearson correlation significance against the bioassay result vector

## Inputs

- features_quantification_matrix_edited_bioactivity.csv — MZmine2/Optimus feature quantification table with row ID, row m/z, row retention time, peak area, and added bioassay values per LC-MS/MS-analyzed sample/fraction
- Bioassay result vector giving the bioactivity level of each fraction analyzed by LC-MS/MS

## Outputs

- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv — transposed feature table (features as rows) with Pearson correlation scores and corrected p-values, formatted for import into Cytoscape as a node attribute table

## How to apply

Load the bioassay-annotated table (features_quantification_matrix_edited_bioactivity.csv) and verify it contains row ID, row m/z, row retention time and peak area columns plus the bioassay values for each LC-MS/MS-analyzed sample. Transpose the matrix so detected features become rows and fractions become columns, because the Pearson correlation is computed per feature across fractions rather than per sample. In the Jupyter notebook running an R kernel via IRkernel, compute the Pearson correlation coefficient between each feature's intensity profile and the bioassay result vector of those fractions, then compute each correlation's significance (p-value) and apply multiple-testing correction. Write the result to features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv, formatted so it can be imported into Cytoscape as a node attribute table in the later GNPS FBMN step. The rationale is that a molecule driving bioactivity should track the bioactivity level across fractions, so correlation of relative abundance with fraction bioactivity predicts which molecular network nodes to prioritize.

## Related tools

- **Jupyter Notebook** (Hosts the Step 2 bioactivity-correlation code, released as a notebook for easiness and reproducibility) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks
- **R / IRkernel** (Provides the R kernel in which the Pearson correlation scores and p-values for every detected feature are calculated inside the Jupyter notebook) — https://github.com/IRkernel/IRkernel
- **MZmine2 (version 2.31)** (Produces the upstream feature quantification matrix (with peak area export) whose features are correlated against bioassay results) — http://mzmine.github.io/
- **Anaconda** (Convenient environment manager for installing Jupyter Notebook, R and IRkernel dependencies)

## Examples

```
install.packages('IRkernel'); IRkernel::installspec()  # register the R kernel ('ir') in Jupyter, then run the Step 2 notebook on features_quantification_matrix_edited_bioactivity.csv
```

## Evaluation signals

- The output matrix is correctly transposed: detected features are rows and LC-MS/MS-analyzed fractions are columns
- Every detected feature has an associated Pearson correlation score (bounded in [-1, 1]) and a p-value (bounded in [0, 1]) against the bioassay result vector
- p-values have been multiple-testing corrected before export
- The exported CSV retains row ID, row m/z, row retention time and peak area columns so it can be imported into Cytoscape as a node attribute table
- Features from bioactive fractions show positive correlation with the bioassay values, consistent with the bioactivity score prediction concept

## Limitations

- Correlation significance depends on the number of fractions analyzed; features without sufficient fraction-level abundance variation yield unstable scores
- The correlation-based bioactivity score is a prediction over molecular networks and must be combined with MS/MS dereplication prior to compound isolation — bioactive candidates of Euphorbia dendroides were missed by classical bioassay-guided fractionation and were only surfaced here
- Requires R installed with IRkernel registered; Jupyter setup issues are handled via the IRkernel installation instructions

## Evidence
