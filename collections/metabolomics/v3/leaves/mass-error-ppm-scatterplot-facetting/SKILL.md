---
name: mass-error-ppm-scatterplot-facetting
description: Use when QC of direct-injection FT-ICR MS formula assignments is needed
  after MetaboDirect pre-processing/filtering — e.g. to inspect per-sample error distributions
  against the 0.5 ppm cutoff.
when_to_use_negative:
- The input is raw FT-ICR MS spectra — molecular formula assignment must be performed
  prior to using MetaboDirect.
- The dataset was already filtered upstream (e.g. the S. fallax leachate data was
  previously filtered by the original study's authors), so re-filtering/diagnostic
  error ranges may not reflect MetaboDirect's 0.5 ppm step.
- The goal is downstream multivariate statistics, chemodiversity, or transformation-network
  analysis rather than data QC overview.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: MetaboDirect
  role: Pipeline whose data diagnostics step generates the error-distribution scatterplots
    and per-sample count bar plots/tables
  repo: https://github.com/Coayala/MetaboDirect
- name: seaborn
  role: Faceted scatterplot rendering of per-sample error distributions
- name: Matplotlib
  role: Figure backend used with seaborn for the diagnostic plots
- name: pandas
  role: Per-sample peak and formula counts and error extraction from the filtered
    table; .csv export
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:8e5af7b5e245f7ccea98d6d2525c486becd18c47f510fb5ed739096160442906
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# mass-error-ppm-scatterplot-facetting

## Summary

Part of the MetaboDirect data diagnostics step: after the filtered FT-ICR MS peak table is produced, generate faceted scatterplots of molecular-formula assignment error (ppm) per sample, alongside per-sample bar plots and .csv tables of total peak and formula counts, to give a quick QC overview before deeper analysis.

## When to use

Use when QC of direct-injection FT-ICR MS formula assignments is needed after MetaboDirect pre-processing/filtering — e.g. to inspect per-sample error distributions against the 0.5 ppm cutoff.

## When NOT to use

- The input is raw FT-ICR MS spectra — molecular formula assignment must be performed prior to using MetaboDirect.
- The dataset was already filtered upstream (e.g. the S. fallax leachate data was previously filtered by the original study's authors), so re-filtering/diagnostic error ranges may not reflect MetaboDirect's 0.5 ppm step.
- The goal is downstream multivariate statistics, chemodiversity, or transformation-network analysis rather than data QC overview.

## Inputs

- Filtered FT-ICR MS peak abundance table with assigned molecular formulas from MetaboDirect data pre-processing (task_001 output)
- Per-mass formula-assignment error values (ppm) from the filtered table

## Outputs

- Faceted scatterplots of formula-assignment error (ppm) distribution per sample (PNG/PDF figures)
- Bar plots of total peaks per sample and formula assignments per sample
- .csv tables of per-sample peak counts and formula-assignment counts

## How to apply

Load the filtered peak table emitted by the MetaboDirect data pre-processing step (peaks already filtered by m/z, 13C isotopic presence, error in formula assignment of 0.5 ppm, and a user-defined minimum number of samples). Using pandas in Python, count total peaks and assigned molecular formulas per sample, and extract the formula-assignment error (ppm) column. Build scatterplots of error per sample with seaborn/Matplotlib, faceting (e.g. seaborn relplot col/col_wrap) so each sample occupies its own panel, and generate companion bar plots of per-sample peak and formula counts. Write all figures and the underlying counts as .csv tables to the MetaboDirect output directory; this step exists to give a quick overview of the data upon which more in-depth analysis can be built.

## Related tools

- **MetaboDirect** (Pipeline whose data diagnostics step generates the error-distribution scatterplots and per-sample count bar plots/tables) — https://github.com/Coayala/MetaboDirect
- **seaborn** (Faceted scatterplot rendering of per-sample error distributions)
- **Matplotlib** (Figure backend used with seaborn for the diagnostic plots)
- **pandas** (Per-sample peak and formula counts and error extraction from the filtered table; .csv export)

## Examples

```
import pandas as pd, seaborn as sns; df = pd.read_csv('filtered_peak_table.csv'); sns.relplot(data=df, x='error', col='sample', col_wrap=4)
```

## Evaluation signals

- Every sample present in the filtered peak table appears as a facet panel in the error scatterplot and a row in each .csv count table.
- Per-sample error values fall within the ±0.5 ppm band implied by the formula-assignment filter applied in pre-processing.
- Summed per-sample counts in the .csv tables equal row counts recomputed independently from the filtered peak table with pandas.
- All diagnostic figures and .csv tables are written to the designated MetaboDirect output directory without missing files.

## Limitations

- Direct-injection FT-ICR MS cannot separate chemical isomers, and signal suppression or enhancement can confound downstream data analysis, so these diagnostics reflect peaks, not fully resolved chemistry.
- The diagnostics are explicitly an overview step ('quick overview of the data, upon which more in-depth analysis can be built'), not a substitute for statistical analysis.
- Datasets pre-filtered by other authors (e.g. S. fallax) may show no filtered masses, so expected error/count patterns differ from freshly processed data.

## Evidence

- [intro] The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics"
- [intro] detected peaks are filtered by their m/z values (based on the user's input), isotopic presence (13C peaks), error in formula assignment (0.5 ppm): [evidence span withheld — anchor status: not_found]
- [abstract] MetaboDirect is written in Python and uses a wide variety of packages, including pandas, NumPy, Matplotlib, seaborn: [evidence span withheld — anchor status: not_found]
- [discussion] can be used to provide a quick overview of the data, upon which more in-depth analysis can be built.: "can be used to provide a quick overview of the data, upon which more in-depth analysis can be built."
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
