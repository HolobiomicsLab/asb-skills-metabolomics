---
name: nmr-metabolite-phenotype-correlation-computation
description: Use when a matrix of projected metabolic scores (e.g., mortality, MetaboAge,
  T2D, COVID-severity surrogates) must be correlated with metabolites or phenotypes,
  or when event prediction over time (Kaplan-Meier) is required.
when_to_use_negative:
- The metabolite dataset has not yet been uploaded/QC-prepped and projected into scores
  — run MiMIR score projection (QCprep, prep_met_for_scores) first
- Data are not Nightingale Health 1H-NMR metabolomics measurements with example-matching
  column names — the app checks required metabolites and will not proceed otherwise
- No time-to-event or endpoint data exist — Kaplan-Meier event prediction (kapmeier_scores,
  survival/survminer) is not applicable to single-timepoint data
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MiMIR
  role: Provides cor_assoc, MetaboWAS/do_metabowas, and kapmeier_scores helpers plus
    the Shiny GUI for association analysis and KM visualization
  repo: github.com/DanieleBizzarri/MiMIR
- name: survival
  role: Computes survival curves for event prediction over time in kapmeier_scores
- name: survminer
  role: Renders the Kaplan-Meier figure for mortality-score event prediction
- name: stats
  role: Statistical tests underlying score-metabolite/phenotype correlations
- name: ggplot2
  role: Plotting backend for association and survival figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:892de9ff29518058bd69bb8c865b5243cf1d0163dc4fabe4e34cb56da073fc86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiMIR
  - R
  - ggplot2
  - matrixStats
  - shiny
  - stats
  - survival
  - survminer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nmr-metabolite-phenotype-correlation-computation

## Summary

Exploratory association analysis in MiMIR: correlate projected metabolomics-based scores with each other, with individual Nightingale 1H-NMR metabolites, and with phenotypic variables (MetaboWAS-style testing adjusted for age, sex and surrogate 'obesity'), plus Kaplan-Meier event-over-time visualization for mortality scores.

## When to use

Use when a matrix of projected metabolic scores (e.g., mortality, MetaboAge, T2D, COVID-severity surrogates) must be correlated with metabolites or phenotypes, or when event prediction over time (Kaplan-Meier) is required.

## When NOT to use

- The metabolite dataset has not yet been uploaded/QC-prepped and projected into scores — run MiMIR score projection (QCprep, prep_met_for_scores) first
- Data are not Nightingale Health 1H-NMR metabolomics measurements with example-matching column names — the app checks required metabolites and will not proceed otherwise
- No time-to-event or endpoint data exist — Kaplan-Meier event prediction (kapmeier_scores, survival/survminer) is not applicable to single-timepoint data

## Inputs

- Matrix of projected metabolic scores (output of MiMIR score projection/calibration)
- Metabolites dataset (CSV or TSV) with column names matching the MiMIR example (Nightingale 1H-NMR features)
- Phenotypic dataset (e.g., synthetic_phenotypic_dataset) with outcomes and covariates (age, sex, obesity)
- Event/time variables for survival analysis (for kapmeier_scores)

## Outputs

- Score-score correlation matrix / heatmap
- Association table with columns: feature, score, statistic, p-value (optionally covariate-adjusted MetaboWAS results)
- Kaplan-Meier survival figure (survminer/ggplot2) for mortality-score event prediction over time
- Saved results file downloadable from the Shiny app

## How to apply

Load the projected metabolic scores together with the metabolic and phenotypic synthetic datasets shipped with MiMIR (CSV/TSV uploads must use the same column names as the example dataset). Compute score-score correlation matrices and score-feature associations with the cor_assoc helper, applying stats-based tests, then adjust associations for age, sex and obesity covariates via the MetaboWAS/do_metabowas wrappers. Assemble per-feature, per-score results into a table with columns feature, score, statistic, and p-value; in the LLS_PAROFFs illustration this surfaced e.g. 110 metabolic features significantly associated with diabetes after correcting for age, sex and the surrogate 'obesity'. For time-to-event analysis, build survival curves for the mortality score with the kapmeier_scores helper using the survival package, render the Kaplan-Meier figure with survminer/ggplot2, and save the association table to file.

## Related tools

- **MiMIR** (Provides cor_assoc, MetaboWAS/do_metabowas, and kapmeier_scores helpers plus the Shiny GUI for association analysis and KM visualization) — github.com/DanieleBizzarri/MiMIR
- **survival** (Computes survival curves for event prediction over time in kapmeier_scores)
- **survminer** (Renders the Kaplan-Meier figure for mortality-score event prediction)
- **stats** (Statistical tests underlying score-metabolite/phenotype correlations)
- **ggplot2** (Plotting backend for association and survival figures)

## Examples

```
library("MiMIR"); MiMIR::startApp()
```

## Evaluation signals

- Association table contains the expected schema: feature, score, statistic, p-value columns
- Score-score correlation matrix is symmetric with unit diagonal
- Covariate-adjusted MetaboWAS results reproduce the paper's pattern of significant feature-score associations (e.g., ~110 metabolic features associated with diabetes after correcting for age, sex and surrogate obesity)
- Kaplan-Meier figure renders with distinct survival curves stratified by mortality score via survminer
- App confirms all required metabolites were found and the dataset was correctly uploaded before scores/associations are computed

## Limitations

- Requires R version 3.6+ and manually installed Shiny/statistics/imaging dependencies if devtools GitHub installation fails
- Input metabolites must use the same column names as the MiMIR example dataset (CSV/TSV only); the app must find all necessary metabolites
- Association adjustment is limited to the covariates specified in the wrappers (age, sex, surrogate obesity); no discussion-section caveats or changelog were available in the source

## Evidence

- [readme] GUI framework for ad-hoc NMR statistics: "is a a unique graphical user interface that provides an intuitive framework for ad-hoc statistical analysis of 1H-NMR metabolomics by Nightingale Health."
- [abstract] 24 pre-trained models without programming: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
- [methods] Example datasets for testing the app: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
- [readme] Upload format requirement: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
