---
name: chemodiversity-metric-computation
description: Use when you have a processed FT-ICR MS peak table with assigned molecular
  formulas and need per-sample diversity metrics (Shannon, Gini-Simpson, Chao1, Rao's
  Q) grouped by experimental variables.
when_to_use_negative:
- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed
  before MetaboDirect (e.g., with MFAssignR/Formularity) and peaks already filtered
  by m/z, 13C isotopic presence, 0.5 ppm error, and minimum sample count
- The analysis goal is multivariate statistics, Van Krevelen visualization, or transformation-network
  generation rather than per-sample diversity indices
- The peak table has not been normalized — run the test_normalization companion script
  first to select the best normalization method before computing diversity
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Command-line pipeline whose chemodiversity analysis step computes the Shannon,
    Gini-Simpson, Chao1, and Rao's Q indices and generates the box plots and .csv
    outputs
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Implementation language of MetaboDirect's chemodiversity step
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Assembles per-sample diversity values into summary tables and writes them
    as .csv files
  repo: https://github.com/Coayala/MetaboDirect
- name: NumPy
  role: Numerical computation underlying the diversity index calculations
  repo: https://github.com/Coayala/MetaboDirect
- name: Matplotlib
  role: Renders box plots of diversity indices grouped by the user's grouping variable
  repo: https://github.com/Coayala/MetaboDirect
- name: seaborn
  role: Statistical plotting library used for the diversity index box plots
  repo: https://github.com/Coayala/MetaboDirect
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:f648df05e0f0622d3805a4d91da558e5b63cff083a39ddc74a261297ccf85a67
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemodiversity-metric-computation

## Summary

Computes per-sample chemodiversity indices on sum-normalized FT-ICR MS peak intensities — abundance-based Shannon, Gini-Simpson, and Chao1 plus functional Rao's quadratic entropy derived from molecular traits — and exports them as .csv tables and grouped box plots, as implemented in MetaboDirect.

## When to use

Use when you have a processed FT-ICR MS peak table with assigned molecular formulas and need per-sample diversity metrics (Shannon, Gini-Simpson, Chao1, Rao's Q) grouped by experimental variables.

## When NOT to use

- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed before MetaboDirect (e.g., with MFAssignR/Formularity) and peaks already filtered by m/z, 13C isotopic presence, 0.5 ppm error, and minimum sample count
- The analysis goal is multivariate statistics, Van Krevelen visualization, or transformation-network generation rather than per-sample diversity indices
- The peak table has not been normalized — run the test_normalization companion script first to select the best normalization method before computing diversity

## Inputs

- Sum-normalized FT-ICR MS peak abundance table with assigned molecular formulas (peak abundances per sample)
- User-specified grouping variable(s) for samples (e.g., phage treatment, incubation condition)
- Normalization method selected via the test_normalization (SPANS) companion script

## Outputs

- Per-sample diversity summary table (.csv) containing Shannon, Gini-Simpson, Chao1, and Rao's Q values plus grouping columns
- Box plots of each diversity index grouped by the user-defined grouping variable (Matplotlib/seaborn figures saved in the chemodiversity output folder)

## How to apply

Load the processed FT-ICR MS peak abundance table (with assigned molecular formulas) that was sum-normalized using the method selected by the test_normalization/SPANS companion script. For each sample, compute the abundance-based indices Shannon diversity, Gini-Simpson diversity, and Chao1 richness on the normalized intensities. Separately derive molecular-level trait vectors from the assigned elemental composition — elemental ratios, unsaturation via double-bond equivalents / AI_mod aromaticity, and reactivity proxies — and compute Rao's quadratic entropy as the functional diversity index per sample. Assemble all per-sample diversity values into a summary table written as .csv files with pandas, then group samples by the user-specified grouping variable (e.g., phage treatment or incubation condition) and render box plots of each index per group with Matplotlib/seaborn. Save both the figures and the .csv tables into the chemodiversity output folder of the MetaboDirect pipeline.

## Related tools

- **MetaboDirect** (Command-line pipeline whose chemodiversity analysis step computes the Shannon, Gini-Simpson, Chao1, and Rao's Q indices and generates the box plots and .csv outputs) — https://github.com/Coayala/MetaboDirect
- **Python** (Implementation language of MetaboDirect's chemodiversity step) — https://github.com/Coayala/MetaboDirect
- **pandas** (Assembles per-sample diversity values into summary tables and writes them as .csv files) — https://github.com/Coayala/MetaboDirect
- **NumPy** (Numerical computation underlying the diversity index calculations) — https://github.com/Coayala/MetaboDirect
- **Matplotlib** (Renders box plots of diversity indices grouped by the user's grouping variable) — https://github.com/Coayala/MetaboDirect
- **seaborn** (Statistical plotting library used for the diversity index box plots) — https://github.com/Coayala/MetaboDirect

## Examples

```
pip install metabodirect && metabodirect -h
```

## Evaluation signals

- All four metrics (Shannon, Gini-Simpson, Chao1, Rao's Q) are computed for every sample in the normalized peak table
- The output .csv tables contain the expected per-sample diversity values plus the grouping columns
- Box plot figures for each diversity index render without error and are grouped by the user-defined grouping variable
- Indices are computed on sum-normalized intensities (normalization chosen via test_normalization/SPANS), not raw abundances
- Rao's quadratic entropy is derived from molecular trait vectors built from elemental ratios and DBE/AI_mod, not from intensity alone

## Limitations

- Chemodiversity results depend on upstream formula assignment and filtering (m/z, 13C isotopic peaks, 0.5 ppm error, minimum sample count); isotopic-carbon formulas can remove ~200 masses from a dataset
- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream analysis
- Requires Python (3.5+), R (4+, with vegan/SYNCSA among others), and Cytoscape (3.8+) installed for the full pipeline

## Evidence

- [intro] chemodiversity as pipeline step: "MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles"
- [abstract] implementation stack: [evidence span withheld — anchor status: not_found]
- [intro] normalization prerequisite: [evidence span withheld — anchor status: not_found]
- [readme] runtime dependencies: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
