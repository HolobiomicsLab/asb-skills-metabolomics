---
name: molecular-trait-functional-diversity-calculation
description: Use when a sum-normalized FT-ICR MS peak table with assigned molecular
  formulas must be summarized as per-sample functional diversity (Rao's Q) grouped
  by experimental conditions such as phage treatment or incubation condition.
when_to_use_negative:
- The input is raw FT-ICR MS spectra that has not yet undergone molecular formula
  assignment — MetaboDirect accepts peak abundance and formula data only after initial
  processing
- The peak table has not been normalized — run the test_normalization/SPANS companion
  script first to select the best normalization method
- The goal is taxonomic or sequence-based diversity rather than molecular-trait-based
  chemodiversity of metabolomic peaks
tools:
- name: MetaboDirect
  role: Pipeline whose chemodiversity analysis step computes Rao's quadratic entropy
    and abundance-based diversity indices and renders grouped box plots
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Implementation language for the chemodiversity calculations and plotting
- name: pandas
  role: Assembles per-sample diversity values into summary tables and exports them
    as .csv files
- name: NumPy
  role: Numerical computation of diversity indices over normalized peak intensities
- name: Matplotlib
  role: Renders box plots of diversity indices grouped by the user's grouping variable
- name: seaborn
  role: Statistical plotting of diversity index box plots in the chemodiversity step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:3a0c3957828e0d4076329fd8b9123370233e4b83c81d8ca6e3ceaba27d62bc31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  - seaborn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# molecular-trait-functional-diversity-calculation

## Summary

Computes functional (trait-based) chemodiversity from FT-ICR MS metabolomic data by deriving molecular-level trait vectors from assigned elemental formulas (elemental ratios, double-bond equivalents/AI_mod aromaticity, reactivity proxies) and calculating Rao's quadratic entropy per sample, alongside abundance-based indices (Shannon, Gini-Simpson, Chao1), in the MetaboDirect pipeline.

## When to use

Use when a sum-normalized FT-ICR MS peak table with assigned molecular formulas must be summarized as per-sample functional diversity (Rao's Q) grouped by experimental conditions such as phage treatment or incubation condition.

## When NOT to use

- The input is raw FT-ICR MS spectra that has not yet undergone molecular formula assignment — MetaboDirect accepts peak abundance and formula data only after initial processing
- The peak table has not been normalized — run the test_normalization/SPANS companion script first to select the best normalization method
- The goal is taxonomic or sequence-based diversity rather than molecular-trait-based chemodiversity of metabolomic peaks

## Inputs

- Processed FT-ICR MS peak abundance table with assigned molecular formulas (post formula assignment, e.g., via MFAssignR), sum-normalized per the method chosen by the test_normalization/SPANS script
- Sample metadata / grouping variable definitions (e.g., phage treatment, incubation condition)

## Outputs

- Per-sample diversity summary table (.csv) containing Shannon, Gini-Simpson, Chao1, and Rao's Q values with grouping columns
- Box plot figures of each diversity index grouped by the user-defined grouping variable, saved to the chemodiversity output folder

## How to apply

Load the processed peak table of peak abundances with assigned molecular formulas that was sum-normalized using the method selected by the test_normalization/SPANS companion script. For each sample, derive molecular trait vectors from the elemental composition — elemental ratios, unsaturation via double-bond equivalents and AI_mod aromaticity, and reactivity proxies — and compute Rao's quadratic entropy as the functional diversity index, using the normalized intensities as abundance weights. Compute the abundance-based indices (Shannon, Gini-Simpson, Chao1) on the same normalized intensities so abundance-based and functional diversity are reported side by side. Assemble per-sample values into a summary table with pandas, export as .csv, group samples by the user-specified grouping variable, and render box plots of each diversity index per group with Matplotlib/seaborn into the chemodiversity output folder.

## Related tools

- **MetaboDirect** (Pipeline whose chemodiversity analysis step computes Rao's quadratic entropy and abundance-based diversity indices and renders grouped box plots) — https://github.com/Coayala/MetaboDirect
- **Python** (Implementation language for the chemodiversity calculations and plotting)
- **pandas** (Assembles per-sample diversity values into summary tables and exports them as .csv files)
- **NumPy** (Numerical computation of diversity indices over normalized peak intensities)
- **Matplotlib** (Renders box plots of diversity indices grouped by the user's grouping variable)
- **seaborn** (Statistical plotting of diversity index box plots in the chemodiversity step)

## Examples

```
pip install metabodirect
```

## Evaluation signals

- Every diversity metric (Shannon, Gini-Simpson, Chao1, Rao's Q) is computed for every sample in the input table
- The exported .csv contains the expected per-sample diversity values plus the grouping variable columns
- Box plot figures render without error and are saved in the chemodiversity output folder, grouped by the user's grouping variable
- Rao's Q values are consistent with the sum-normalized intensities produced by the test_normalization/SPANS-selected method
- Functional diversity (Rao's Q) is reported separately and distinguishable from the abundance-based indices in the summary table

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analyses including chemodiversity
- Rao's Q depends on the quality of molecular formula assignments; results are downstream of formula assignment and of the normalization method selected by test_normalization
- Diversity outputs are a quick overview of the data, upon which more in-depth analysis must be built

## Evidence

- [abstract] MetaboDirect is written in Python and uses a wide variety of packages, including pandas, NumPy, Matplotlib, seaborn, scikit-learn, and NetworkX: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline designed for chemodiversity analysis of FT-ICR MS metabolomic profiles: "MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles"
- [intro] Pipeline accepts post-processing peak abundance and formula data, not raw spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [results] Normalization helper script selects the best normalization method before the main pipeline: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [readme] README scope statement for the FT-ICR MS analysis pipeline: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
