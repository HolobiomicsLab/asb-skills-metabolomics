---
name: tukey-hsd-significance-letter-annotation
description: Use when comparing distributions of derived molecular descriptors (H/C,
  O/C, NOSC, GFE, AImod, DBE) across user-defined sample groups and the resulting
  violin/density plots need statistically justified significance letters on group
  medians.
when_to_use_negative:
- The goal is pairwise overlap of detected formulas between groups (Venn/upset plots)
  rather than between-group descriptor differences
- The input table has no metadata grouping variables, so no between-group statistical
  comparison can be defined
- Peaks have not yet passed MetaboDirect filtering (m/z, 13C isotopic peaks, 0.5 ppm
  formula-assignment error, minimum sample threshold), so descriptor comparisons would
  be computed on unfiltered data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Pipeline whose data exploration step generates the NOSC/GFE/AImod/DBE violin
    and density plots and computes the descriptors they display
  repo: https://github.com/Coayala/MetaboDirect
- name: statsmodels
  role: Python dependency used to run the Tukey HSD test that yields the significance
    letters
- name: seaborn
  role: Renders the violin and density plots onto which the Tukey letters are overlaid
- name: Matplotlib
  role: Plotting backend for the annotated exploration figures written as PNGs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:07a7ca0623720c95fade1379a43747c990b9f02878390ba18bdea0aa49030a97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - pandas
  - scikit-learn
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Tukey HSD significance letter annotation on group descriptor plots

## Summary

Annotates violin and density plots of FT-ICR MS molecular descriptors (NOSC, GFE, AImod, DBE) with Tukey HSD significance letters so that user-defined sample groups whose medians differ statistically are visually distinguishable, as implemented in the MetaboDirect data exploration step.

## When to use

Use when comparing distributions of derived molecular descriptors (H/C, O/C, NOSC, GFE, AImod, DBE) across user-defined sample groups and the resulting violin/density plots need statistically justified significance letters on group medians.

## When NOT to use

- The goal is pairwise overlap of detected formulas between groups (Venn/upset plots) rather than between-group descriptor differences
- The input table has no metadata grouping variables, so no between-group statistical comparison can be defined
- Peaks have not yet passed MetaboDirect filtering (m/z, 13C isotopic peaks, 0.5 ppm formula-assignment error, minimum sample threshold), so descriptor comparisons would be computed on unfiltered data

## Inputs

- Filtered FT-ICR MS peak/formula table from MetaboDirect pre-processing (per-sample abundances plus assigned molecular formulas)
- Sample metadata table defining the user-defined grouping variables

## Outputs

- Violin plots of NOSC, GFE, AImod, and DBE per group annotated with Tukey HSD significance letters (PNG)
- Density plots of NOSC, GFE, AImod, and DBE per group annotated with Tukey HSD significance letters (PNG)

## How to apply

Start from the filtered peak/formula table produced by MetaboDirect pre-processing, which carries per-sample abundances, assigned molecular formulas, and metadata grouping variables. Compute the derived descriptors for each formula (NOSC, GFE, AImod, DBE) with the pandas/NumPy formulas implemented in MetaboDirect, then run the statsmodels/scipy Tukey HSD test on each descriptor across the grouping variable to identify statistically different group medians. Convert the Tukey results into shared-letter assignments (groups sharing a letter are not significantly different) and overlay the letters onto Matplotlib/seaborn violin and density plots of NOSC, GFE, AImod, and DBE per group. Write each annotated figure as a PNG into the MetaboDirect exploration output directory, mirroring the outputs enumerated in the pipeline's user's guide.

## Related tools

- **MetaboDirect** (Pipeline whose data exploration step generates the NOSC/GFE/AImod/DBE violin and density plots and computes the descriptors they display) — https://github.com/Coayala/MetaboDirect
- **statsmodels** (Python dependency used to run the Tukey HSD test that yields the significance letters)
- **seaborn** (Renders the violin and density plots onto which the Tukey letters are overlaid)
- **Matplotlib** (Plotting backend for the annotated exploration figures written as PNGs)

## Examples

```
metabodirect -h
```

## Evaluation signals

- The exploration output directory contains violin and density plots for NOSC, GFE, AImod, and DBE, each annotated with Tukey letters, matching the outputs enumerated in the MetaboDirect user's guide
- Groups plotted with the same significance letter have group medians that are not significantly different under the Tukey HSD test, and groups with distinct letters do differ
- Letters are consistent with the group assignments in the supplied metadata table (no letters assigned across undefined groups)
- Every annotated figure is written as a PNG file in the exploration output directory

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis including these group-level comparisons
- The letters summarize differences in descriptor distributions between groups only; they do not identify which individual metabolites drive the differences
- MetaboDirect must be installed with its full dependency stack (Python 3.5+, R 4+, Cytoscape 3.8+), and missing R or Python packages can break the exploration step

## Evidence

- [intro] The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [intro] MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles: "MetaboDirect was designed to facilitate data exploration, data visualization, chemodiversity, and statistical analysis of metabolomic profiles"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above): "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
- [other] Generate violin and density plots of NOSC, GFE, AImod and DBE per group and overlay Tukey HSD significance letters computed with the statsmodels/scipy Tukey test: [evidence span withheld — anchor status: not_found]
