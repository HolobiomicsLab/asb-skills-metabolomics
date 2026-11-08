---
name: bioassay-guided-fractionation-comparison
description: Use when a bioactive molecular network (Pearson r2 > 0.8, p < 0.05 corrected)
  must be compared against the molecule list isolated by classical bioassay-guided
  fractionation to identify newly highlighted candidates.
when_to_use_negative:
- Input is already a merged comparison table or the candidate list has not yet been
  filtered to r2 > 0.8 and corrected p < 0.05.
- The extract has no bioassay-guided fractionation result to compare against (no classical
  isolation report available).
- Only a bioactivity score visualization in the molecular network is needed; the comparison
  step is specific to auditing what classical fractionation missed.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3172
tools:
- name: Jupyter Notebook
  role: Hosts the Pearson correlation calculation that produces the significant-correlation
    candidate table used for comparison; requires the IRkernel so R code runs inside
    the notebook.
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: R
  role: Runs the Pearson correlation and p-value correction inside the Jupyter notebook
    via IRkernel; pandas/R are used to load and filter the correlation output and
    perform the set comparison.
  repo: https://github.com/IRkernel/IRkernel
- name: GNPS web-platform
  role: Provides the feature-based molecular networking (FBMN) job whose molecular
    network context supplies the candidate molecules surfaced by bioactivity score
    prediction.
  repo: http://gnps.ucsd.edu
- name: MZmine2
  role: Produces the feature quantification table (features_quantification_matrix.csv)
    that, after adding bioassay values, becomes the input for the correlation and
    comparison workflow.
  repo: http://mzmine.github.io/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:bd11eafd48642e759f4814e7235a1e36a9ce4d092e39394657e4b184e6fca22d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks
  tools:
  - GNPS web-platform
  - Jupyter Notebook
  - MZmine2
  - R
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bioassay-guided-fractionation-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Set-comparison of bioactivity-score-predicted candidate molecules from a molecular network against the molecules actually recovered by classical bioassay-guided fractionation, to expose missed candidates.

## When to use

Use when a bioactive molecular network (Pearson r2 > 0.8, p < 0.05 corrected) must be compared against the molecule list isolated by classical bioassay-guided fractionation to identify newly highlighted candidates.

## When NOT to use

- Input is already a merged comparison table or the candidate list has not yet been filtered to r2 > 0.8 and corrected p < 0.05.
- The extract has no bioassay-guided fractionation result to compare against (no classical isolation report available).
- Only a bioactivity score visualization in the molecular network is needed; the comparison step is specific to auditing what classical fractionation missed.

## Inputs

- features_quantification_matrix_edited_bioactivity.csv (aligned feature quantification table annotated with bioassay values for each fraction analyzed by LC-MS/MS)
- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (Jupyter notebook Pearson correlation output with corrected p-values)
- list of molecules recovered by the classical bioassay-guided fractionation of the extract (from the original study report)

## Outputs

- comparison_table.csv with each molecule's row ID, m/z, Pearson r2, corrected p-value, and classification flag ('classically recovered' vs 'newly highlighted by the network')

## How to apply

Load the aligned feature quantification table annotated with bioassay values (features_quantification_matrix_edited_bioactivity.csv) and the Jupyter-notebook correlation output (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv) with pandas or R. The correlation output is produced by the notebook's Pearson correlation step, which computes the Pearson correlation score and its significance for every detected feature against the bioassay result, and requires the IRkernel so that R runs inside Jupyter. Filter the correlation table to candidate molecules meeting the significance threshold r2 > 0.8 and corrected p-value < 0.05. Load the list of molecules recovered by the classical bioassay-guided fractionation from the original study report. Perform a set comparison between the two molecule lists, classifying each candidate as 'classically recovered' or 'newly highlighted by the network', and write a comparison table (comparison_table.csv) listing each molecule's row ID, m/z, Pearson r2, corrected p-value, and classification flag.

## Related tools

- **Jupyter Notebook** (Hosts the Pearson correlation calculation that produces the significant-correlation candidate table used for comparison; requires the IRkernel so R code runs inside the notebook.) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **R** (Runs the Pearson correlation and p-value correction inside the Jupyter notebook via IRkernel; pandas/R are used to load and filter the correlation output and perform the set comparison.) — https://github.com/IRkernel/IRkernel
- **GNPS web-platform** (Provides the feature-based molecular networking (FBMN) job whose molecular network context supplies the candidate molecules surfaced by bioactivity score prediction.) — http://gnps.ucsd.edu
- **MZmine2** (Produces the feature quantification table (features_quantification_matrix.csv) that, after adding bioassay values, becomes the input for the correlation and comparison workflow.) — http://mzmine.github.io/

## Examples

```
import pandas as pd; corr = pd.read_csv('features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv'); candidates = corr[(corr.r2 > 0.8) & (corr.pvalue < 0.05)]
```

## Evaluation signals

- Every candidate in comparison_table.csv has a Pearson r2 > 0.8 and a corrected p-value < 0.05 as computed by the notebook.
- Each molecule is assigned exactly one classification flag: 'classically recovered' or 'newly highlighted by the network'.
- The comparison table columns include row ID, m/z, Pearson r2, corrected p-value, and classification flag.
- The set of 'classically recovered' molecules is a subset of the classical bioassay-guided fractionation molecule list from the original study.
- The input feature quantification table contains bioassay values for each fraction analyzed by LC-MS/MS (i.e. was edited to features_quantification_matrix_edited_bioactivity.csv).

## Limitations

- The comparison only exposes candidates at the significance threshold r2 > 0.8 and p < 0.05; weaker correlations are not surfaced.
- MS/MS validation of putative annotations is needed (currently not provided in Optimus), so candidates identified by mz-RT matching are not confirmed structures.
- The 'newly highlighted' classification depends on the completeness of the classical bioassay-guided fractionation report supplied as input.

## Evidence

- [other] Filter correlation table to candidates meeting the significance threshold r2 > 0.8 and p-value < 0.05 (the notebook applies Pearson correlation and p-value correction).: [evidence span withheld — anchor status: not_found]
- [readme] Note that MS/MS validation of putative annotations is needed (currently not provided in Optimus).: [evidence span withheld — anchor status: external_doc]
