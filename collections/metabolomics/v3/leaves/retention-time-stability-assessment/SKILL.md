---
name: retention-time-stability-assessment
description: Use when a metabolomics/lipidomics feature table contains repeated pooled-QC
  injections and you need to verify retention time precision (RSD < 2%) before trusting
  chromatographic alignment across the sequence.
when_to_use_negative:
- Data lack repeated pooled-QC injections (QCpool), so no RT precision can be estimated
  across the sequence
- The goal is drift correction or signal-drift adjustment rather than assessing whether
  RT stability meets acceptance criteria
- Input is raw vendor data not yet processed into a per-descriptor feature table (e.g.
  un-exported Sciex MultiQuant project)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QComics
  role: Shiny app implementing the sequential QC protocol, including the RT stability
    criterion and time-series tolerance plots
  repo: https://github.com/ricoderks/QComics
- name: R
  role: Runtime environment for the QComics Shiny app and descriptor-wise RSD computation
- name: RStudio
  role: Environment from which QComics::run_app() is launched
- name: dplyr
  role: Per-descriptor RSD computation over repeated QCpool injections
- name: ggplot2
  role: Time-series QC plots of RT with tolerance-window bands via make_qc_graph
- name: DT
  role: Rendering of the per-descriptor pass/fail table via make_nice_table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c03660
    title: 'QC <i>omics</i> : Recommendations and Guidelines for Robust, Easily Implementable
      and Reportable Quality Control of Metabolomics Data'
  source_content_hash: sha256:89b1abfcaee0f0d84ebbf36127c4496b5ad952369b5da6f3c4b7c3ca9a484fae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ricoderks/QComics
  tools:
  - DT
  - QComics
  - R
  - RStudio
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Retention time stability assessment

## Summary

Evaluates chromatographic reproducibility in repeated pooled-QC (QCpool) injections by computing per-descriptor retention time RSD and applying the QComics acceptance criterion (RSD < 2%), visualized as time-series plots with predefined tolerance windows.

## When to use

Use when a metabolomics/lipidomics feature table contains repeated pooled-QC injections and you need to verify retention time precision (RSD < 2%) before trusting chromatographic alignment across the sequence.

## When NOT to use

- Data lack repeated pooled-QC injections (QCpool), so no RT precision can be estimated across the sequence
- The goal is drift correction or signal-drift adjustment rather than assessing whether RT stability meets acceptance criteria
- Input is raw vendor data not yet processed into a per-descriptor feature table (e.g. un-exported Sciex MultiQuant project)

## Inputs

- Feature table of QCpool samples (data_df) exported from Sciex MultiQuant (> v3.0.3) as txt, including retention times per descriptor
- Injection sequence order for the repeated QCpool measurements

## Outputs

- Per-descriptor retention time RSD (%) estimates
- Pass/fail flag per descriptor against the RSD < 2% acceptance criterion (e.g. qc_descriptor_assessment.csv)
- Time-series QC plots of retention time per descriptor with tolerance-window bands (ggplot2)

## How to apply

Load the QCpool feature table (data_df) with its injection sequence into the QComics Shiny app (QComics::run_app() in R/RStudio, or run_in_browser()). Compute per-descriptor retention time RSD (%) across the repeated QCpool injections, e.g. with dplyr, alongside the companion descriptor checks (peak intensity RSD < 30%, peak width RSD < 15%, m/z error < 10 ppm for HRMS). Flag each descriptor pass/fail against the RSD < 2% retention time threshold, which QComics adapted from bioanalytical method validation guidelines. Export the per-descriptor pass/fail table (e.g. qc_descriptor_assessment.csv via make_nice_table/DT) and generate time-series RT plots per descriptor with tolerance-window bands using make_qc_graph/ggplot2 to spot drifts or out-of-control injections. Descriptors failing the 2% RT criterion indicate chromatographic instability requiring investigation before downstream analysis.

## Related tools

- **QComics** (Shiny app implementing the sequential QC protocol, including the RT stability criterion and time-series tolerance plots) — https://github.com/ricoderks/QComics
- **R** (Runtime environment for the QComics Shiny app and descriptor-wise RSD computation)
- **RStudio** (Environment from which QComics::run_app() is launched)
- **dplyr** (Per-descriptor RSD computation over repeated QCpool injections)
- **ggplot2** (Time-series QC plots of RT with tolerance-window bands via make_qc_graph)
- **DT** (Rendering of the per-descriptor pass/fail table via make_nice_table)

## Examples

```
remotes::install_github("ricoderks/QComics"); QComics::run_app()  # load QCpool data_df with injection sequence, then inspect RT RSD (< 2%) in the QC assessment tab
```

## Evaluation signals

- Retention time RSD (%) computed per descriptor across all repeated QCpool injections matches values recomputed independently (e.g. with dplyr/Excel)
- Every descriptor carries an explicit pass/fail flag against the RSD < 2% threshold
- Time-series plots show RT values with tolerance-window bands and correctly ordered injection sequence
- QCpool feature table is the txt export from Sciex MultiQuant (> v3.0.3), preserving per-descriptor RT values
- Pass/fail table (qc_descriptor_assessment.csv) contains all descriptors and no missing RT entries are silently dropped

## Limitations

- The RSD < 2% threshold is designed for targeted MS-based metabolomics/lipidomics QCpool data; QComics is experimental and oriented toward Sciex MultiQuant exports
- The m/z error criterion (< 10 ppm) applies only to HRMS descriptors, and other thresholds (30% intensity, 15% width) must be assessed alongside RT for the overall QC verdict
- Preanalytical factors and missing vs. truly absent data must be handled separately; RT stability alone does not guarantee overall data quality

## Evidence

- [full_text] RSD < 30% for peak intensity, RSD < 2% for RT, RSD < 15% for peak width, m/z error <10 ppm (HRMS only): [evidence span withheld — anchor status: not_found]
- [intro] The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods: "The quality criteria employed in QComics have been adapted from guidelines for validating conventional bioanalytical methods21 and from existing literature on current QC practices in metabolomics"
- [methods] From Rstudio run the app as: "From Rstudio run the app as"
- [abstract] assess overall data quality in terms of precision and accuracy: "(vi) assess overall data quality in terms of precision and accuracy"
- [readme] These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to txt format: "These QCpool samples need to be analysed with Sciex Multiquant (> v3.0.3) software and exported to `txt` format."
