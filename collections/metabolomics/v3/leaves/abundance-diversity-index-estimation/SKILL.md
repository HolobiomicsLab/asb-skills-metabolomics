---
name: abundance-diversity-index-estimation
description: Use when a processed FT-ICR MS peak table (abundances + assigned molecular
  formulas) needs per-sample chemodiversity quantification grouped by experimental
  variables, distinguishing abundance-based indices from functional (trait-based)
  diversity.
when_to_use_negative:
- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed
  before MetaboDirect, so raw spectra need prior processing (e.g., MFAssignR)
- Goal is taxonomic/sequence-based community diversity rather than chemodiversity
  of mass spectrometry peaks
- The peak table has not been normalized — compute diversity only after test_normalization-guided
  normalization to avoid intensity-scale artifacts
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: MetaboDirect
  role: Command-line pipeline whose chemodiversity analysis step computes the Shannon,
    Gini-Simpson, Chao1, and Rao's Q indices and generates the box plots and .csv
    outputs
  repo: https://github.com/Coayala/MetaboDirect
- name: vegan
  role: R package dependency supporting the diversity index calculations within MetaboDirect
- name: SYNCSA
  role: R package dependency supporting trait-based (functional) diversity computation
    such as Rao's quadratic entropy
- name: pandas
  role: Writes the per-sample diversity summary tables as .csv files
- name: seaborn
  role: Renders the per-index box plots grouped by the user's grouping variable
- name: pmartR
  role: Used by the test_normalization companion script to select the best normalization
    method before diversity computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:9d0a38c752db1dba6c42690cce95eae50ed0e236c45ea2998dafd94dec1873e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - SYNCSA
  - pandas
  - pmartR
  - seaborn
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# abundance-diversity-index-estimation

## Summary

Computes per-sample abundance-based diversity (Shannon, Gini-Simpson, Chao1) and functional diversity (Rao's quadratic entropy) on sum-normalized FT-ICR MS peak intensities with assigned molecular formulas, as implemented in MetaboDirect's chemodiversity analysis step.

## When to use

Use when a processed FT-ICR MS peak table (abundances + assigned molecular formulas) needs per-sample chemodiversity quantification grouped by experimental variables, distinguishing abundance-based indices from functional (trait-based) diversity.

## When NOT to use

- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed before MetaboDirect, so raw spectra need prior processing (e.g., MFAssignR)
- Goal is taxonomic/sequence-based community diversity rather than chemodiversity of mass spectrometry peaks
- The peak table has not been normalized — compute diversity only after test_normalization-guided normalization to avoid intensity-scale artifacts

## Inputs

- Processed FT-ICR MS peak abundance table with assigned molecular formulas (output of prior formula assignment, e.g., MFAssignR/Formularity)
- Sum-normalized peak intensity matrix selected via the test_normalization companion script
- Sample metadata with user-defined grouping variable (e.g., phage treatment or incubation condition)

## Outputs

- .csv files of per-sample diversity values (Shannon, Gini-Simpson, Chao1, Rao's Q) with grouping columns
- Box plot figures of each diversity index grouped by the user's grouping variables, saved in the chemodiversity output folder

## How to apply

First run the 'test_normalization' companion script (which uses pmartR/SPANS) to select the best normalization method, then sum-normalize the peak abundance table. Load the normalized peak table with assigned molecular formulas into MetaboDirect's chemodiversity step (step iv of its 6-step pipeline). Compute Shannon diversity, Gini-Simpson diversity, and Chao1 richness on the normalized intensities for each sample as abundance-based metrics; derive molecular trait vectors from elemental composition (elemental ratios, double-bond equivalents/AI_mod aromaticity, reactivity proxies) and compute Rao's quadratic entropy as the functional diversity index. Write per-sample diversity values to .csv tables via pandas and render box plots of each index grouped by the user-defined grouping variable (e.g., phage treatment) with Matplotlib/seaborn. Note the R environment must provide vegan and SYNCSA, which support the diversity calculations.

## Related tools

- **MetaboDirect** (Command-line pipeline whose chemodiversity analysis step computes the Shannon, Gini-Simpson, Chao1, and Rao's Q indices and generates the box plots and .csv outputs) — https://github.com/Coayala/MetaboDirect
- **vegan** (R package dependency supporting the diversity index calculations within MetaboDirect)
- **SYNCSA** (R package dependency supporting trait-based (functional) diversity computation such as Rao's quadratic entropy)
- **pandas** (Writes the per-sample diversity summary tables as .csv files)
- **seaborn** (Renders the per-index box plots grouped by the user's grouping variable)
- **pmartR** (Used by the test_normalization companion script to select the best normalization method before diversity computation)

## Examples

```
pip install metabodirect; metabodirect -h
```

## Evaluation signals

- All four diversity metrics (Shannon, Gini-Simpson, Chao1, Rao's Q) are computed for every sample in the peak table
- Output .csv tables contain the expected per-sample values plus the user-specified grouping columns
- Box plot figures for each diversity index render without error and are saved in the chemodiversity output folder
- Indices are computed on sum-normalized intensities selected via the test_normalization companion script, not on raw abundances
-  Rao's Q values are derived from molecular trait vectors built from elemental composition (elemental ratios, DBE/AI_mod), not from abundances alone

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis, including diversity indices
- Rao's Q functional diversity depends on the quality of molecular formula assignments and derived trait proxies (elemental ratios, AI_mod), so misassignments propagate into functional diversity values
- Normalization choice materially affects abundance-based indices; MetaboDirect provides test_normalization to guide selection, but the choice remains user-dependent
- MetaboDirect requires Python (3.5+), R (4+) with packages including vegan and SYNCSA, and Cytoscape (3.8+), so the environment must be fully provisioned before running

## Evidence

- [intro] Pipeline steps include chemodiversity as a distinct stage: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis"
- [intro] Normalization must be selected before the main pipeline: [evidence span withheld — anchor status: not_found]
- [results] Applied workflow used test_normalization first in the case studies: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [abstract] Python stack used by the pipeline: [evidence span withheld — anchor status: not_found]
- [readme] README lists R dependencies relevant to diversity computation: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
