---
name: molecular-formula-assignment-count-visualization
description: Use when you have a formula-assigned, filtered FT-ICR MS peak table and
  need per-sample QC summaries of peak counts, formula assignments, and ppm assignment
  errors, as done in MetaboDirect's data diagnostics step.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or a peak table without prior molecular formula assignment
  — MetaboDirect requires formula-assigned data produced after initial processing
  of raw spectra
- Goal is downstream analysis such as chemodiversity, multivariate statistics, or
  mass-difference transformation networks rather than QC-style per-sample count summaries
- The dataset has not yet passed the pre-processing filters (m/z range, 13C isotopes,
  0.5 ppm assignment error, minimum sample threshold), so counts would reflect unfiltered
  data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Pipeline whose data diagnostics step defines the diagnostic plots and tables
    being reproduced; written in Python
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Counting total peaks and assigned molecular formulas per sample from the filtered
    peak table
- name: seaborn
  role: Rendering bar plots of per-sample counts and faceted scatterplots of assignment
    error
- name: Matplotlib
  role: Plotting backend for the diagnostic bar plots and faceted scatterplots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:22e8a266bd4c50f31fcb8882cbc90139d76027b3cc71fd776fcd75456ce425ad
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# molecular-formula-assignment-count-visualization

## Summary

Reproduces the MetaboDirect 'data diagnostics' step (step ii of the 6-step FT-ICR MS pipeline), which summarizes per-sample peak counts and molecular formula assignments as bar plots, .csv count tables, and faceted scatterplots of formula-assignment error (ppm). It serves as a quality-control overview of the filtered peak table before deeper analyses.

## When to use

Use when you have a formula-assigned, filtered FT-ICR MS peak table and need per-sample QC summaries of peak counts, formula assignments, and ppm assignment errors, as done in MetaboDirect's data diagnostics step.

## When NOT to use

- Input is raw FT-ICR MS spectra or a peak table without prior molecular formula assignment — MetaboDirect requires formula-assigned data produced after initial processing of raw spectra
- Goal is downstream analysis such as chemodiversity, multivariate statistics, or mass-difference transformation networks rather than QC-style per-sample count summaries
- The dataset has not yet passed the pre-processing filters (m/z range, 13C isotopes, 0.5 ppm assignment error, minimum sample threshold), so counts would reflect unfiltered data

## Inputs

- Filtered FT-ICR MS peak abundance table with assigned molecular formulas (output of MetaboDirect data pre-processing)
- Per-sample metadata/sample identifiers present in the peak table

## Outputs

- Bar plot of total peaks detected per sample
- Bar plot of molecular formula assignments per sample
- .csv tables of per-sample peak counts and per-sample formula-assignment counts
- Faceted scatterplots of formula-assignment error (ppm) distribution per sample

## How to apply

Load the filtered peak table produced by the MetaboDirect data pre-processing step (peaks already filtered by m/z values, 13C isotopic presence, formula-assignment error of 0.5 ppm, and a user-defined minimum number of samples). Use pandas to count total peaks detected per sample and assigned molecular formulas per sample, then render these counts as bar plots with seaborn/Matplotlib and export the counts as .csv tables. Compute the per-sample formula-assignment error values in ppm from the filtered table and plot them as faceted scatterplots per sample. Write all figures and .csv tables to the output directory; the diagnostics are meant to 'provide a quick overview of the data, upon which more in-depth analysis can be built.'

## Related tools

- **MetaboDirect** (Pipeline whose data diagnostics step defines the diagnostic plots and tables being reproduced; written in Python) — https://github.com/Coayala/MetaboDirect
- **pandas** (Counting total peaks and assigned molecular formulas per sample from the filtered peak table)
- **seaborn** (Rendering bar plots of per-sample counts and faceted scatterplots of assignment error)
- **Matplotlib** (Plotting backend for the diagnostic bar plots and faceted scatterplots)

## Evaluation signals

- Every sample present in the filtered peak table appears in each diagnostic .csv table and figure
- Per-sample peak counts and formula-assignment counts in the .csv tables match direct groupby tallies of the filtered peak table
- Error values plotted in the faceted scatterplots are in ppm and consistent with the 0.5 ppm assignment-error filter (no values beyond the filter)
- All expected diagnostic files (bar plots, .csv tables, faceted error scatterplots) are written to the output directory
- Counts reflect the post-filter table, e.g. masses with isotopic-carbon formulas (about 200 removed in the bacterium-phage data set) are excluded

## Limitations

- Diagnostics provide only a quick overview; more in-depth analyses must be built on top of them
- DI-MS direct injection cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream data analysis, which these counts cannot reveal
- Counts depend on upstream filtering choices (m/z range, 13C isotope removal, 0.5 ppm error threshold, user-defined minimum sample presence); differently filtered inputs give incomparable counts

## Evidence

- [intro] data diagnostics as step ii of the MetaboDirect pipeline: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis"
- [intro] upstream filters applied before diagnostics: [evidence span withheld — anchor status: not_found]
- [discussion] purpose of the diagnostics outputs: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
- [abstract] Python plotting stack used: [evidence span withheld — anchor status: not_found]
- [readme] scope of the pipeline on formula-assigned direct-injection data: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
