---
name: anova-posthoc-tukey-hsd-analysis
description: Use when a transf_data-normalised targeted or untargeted metabolomics
  feature table has one or two grouping factors (e.g. study group, time point) and
  per-feature ANOVA plus post-hoc Tukey HSD pairwise tables are needed.
when_to_use_negative:
- The comparison is strictly pairwise between two groups — use gentab_P.t.test with
  FDR plus gentab_FC and a volcano plot (Volcano_ttest_FC) instead of ANOVA.
- The response is longitudinal/censored — GetFeatistics routes these to mixed-effects
  or Tobit regression models, not the gentab_P ANOVA functions.
- Intensities are still raw (missing values not replaced, not log-transformed, not
  pareto-scaled); run transf_data first or the p-values will be computed on unnormalised
  data.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: GetFeatistics
  role: R package providing gentab_P.1wayANOVA_posthocTukeyHSD and gentab_P.2wayANOVA_posthocTukeyHSD
    for ANOVA with post-hoc Tukey HSD and FDR correction on metabolomics feature tables
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime environment (version ≥ 4.3.1) in which GetFeatistics ANOVA functions
    are executed
- name: ggplot
  role: Grammar used by GetFeatistics to customise output visualisations generated
    alongside the statistical tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:34a8121de4e3e0357d1a98039f56656671cdcb7c8c1eddcc02536d279c078190
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - R
  - ggplot
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# anova-posthoc-tukey-hsd-analysis

## Summary

GetFeatistics-style one- and two-way ANOVA with post-hoc Tukey HSD comparisons and FDR correction for metabolomics feature-intensity tables, implemented as gentab_P.1wayANOVA_posthocTukeyHSD and gentab_P.2wayANOVA_posthocTukeyHSD on transf_data-normalised data.

## When to use

Use when a transf_data-normalised targeted or untargeted metabolomics feature table has one or two grouping factors (e.g. study group, time point) and per-feature ANOVA plus post-hoc Tukey HSD pairwise tables are needed.

## When NOT to use

- The comparison is strictly pairwise between two groups — use gentab_P.t.test with FDR plus gentab_FC and a volcano plot (Volcano_ttest_FC) instead of ANOVA.
- The response is longitudinal/censored — GetFeatistics routes these to mixed-effects or Tobit regression models, not the gentab_P ANOVA functions.
- Intensities are still raw (missing values not replaced, not log-transformed, not pareto-scaled); run transf_data first or the p-values will be computed on unnormalised data.

## Inputs

- Targeted or untargeted metabolomics feature-intensity matrix (feature rows x sample columns) already processed with transf_data (missing-value replacement, log-transform, pareto scaling)
- Sample metadata table df_example_sample_data with the grouping factor(s) used for one-way or two-way ANOVA

## Outputs

- Concise per-feature one-way ANOVA table with p-values and post-hoc Tukey HSD pairwise comparisons (gentab_P.1wayANOVA_posthocTukeyHSD)
- Concise per-feature two-way ANOVA table with p-values and post-hoc Tukey HSD pairwise comparisons (gentab_P.2wayANOVA_posthocTukeyHSD)
- FDR-corrected p-value tables for reporting

## How to apply

First pass the intensity matrix through transf_data, which replaces missing values, log-transforms intensities and applies pareto scaling, since the ANOVA functions expect processed intensities. Call gentab_P.1wayANOVA_posthocTukeyHSD on the transformed data with the single grouping factor (e.g. sample group) to get a per-feature table of ANOVA p-values and post-hoc Tukey HSD pairwise comparisons; call gentab_P.2wayANOVA_posthocTukeyHSD with two factors to obtain the equivalent two-way ANOVA table. FDR correction is applied to the resulting p-value tables, mirroring the vignette's workflow where gentab_P.t.test output is also FDR-adjusted. The rationale is that GetFeatistics wraps these steps into concise, informative tables rather than raw R model output, so the outputs are directly usable in reporting, in contrast to ad-hoc aov()/TukeyHSD() scripts. In the GetFeatistics vignette workflow these ANOVA tables are produced after gentab_descr, gentab_P.t.test and gentab_FC, and can feed downstream visualisations such as Volcano_ttest_FC.

## Related tools

- **GetFeatistics** (R package providing gentab_P.1wayANOVA_posthocTukeyHSD and gentab_P.2wayANOVA_posthocTukeyHSD for ANOVA with post-hoc Tukey HSD and FDR correction on metabolomics feature tables) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime environment (version ≥ 4.3.1) in which GetFeatistics ANOVA functions are executed)
- **ggplot** (Grammar used by GetFeatistics to customise output visualisations generated alongside the statistical tables)

## Examples

```
library(GetFeatistics); res <- gentab_P.1wayANOVA_posthocTukeyHSD(df_processed, df_example_sample_data, group = "study_group")
```

## Evaluation signals

- The output is a GetFeatistics 'concise and informative' per-feature table — one row per feature with ANOVA p-value and Tukey HSD pairwise entries — not raw aov()/TukeyHSD() R objects.
- P-values in the gentab tables are FDR-adjusted, matching the vignette's treatment of gentab_P.t.test with FDR.
- ANOVA is run on transf_data output (missing values replaced, log-transformed, pareto-scaled intensities), so running it directly on raw intensities is a red flag.
- Two-way ANOVA results should show both main effects and their factors' pairwise Tukey HSD comparisons, mirroring the two grouping factors of the vignette's mock targeted/untargeted example.
- Reproducing the vignette's combined targeted/untargeted example data should yield the same ANOVA/Tukey tables as the published GF vignette workflow.

## Limitations

- The GetFeatistics ANOVA functions are described for feature tables where data intensities have already been pre-processed for missing values and data normality; behaviour on unnormalised input is not specified.
- In the GetFeatistics workflow, mixed-effects (longitudinal) and Tobit (censored, above LOD) regression models are the recommended alternatives when the design involves longitudinal data or censoring, rather than these ANOVA tables.
- GetFeatistics installation requires R ≥ 4.3.1 and a matching-architecture Java installation; missing Java breaks installation of the package providing these functions.

## Evidence

- [other] Vignette applies transf_data then runs one- and two-way ANOVA with post-hoc Tukey HSD and FDR correction: [evidence span withheld — anchor status: not_found]
- [abstract] Output tables are concise and informative: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
- [intro] Pre-processing deals with missing values and data normality before statistics: "data intensities are pre-processed to deal with missing values and data normality"
- [intro] GetFeatistics offers a pipeline for pooled QC filtration and mixed effect and Tobit regression models: "The tool is designed to streamline the data elaboration of targeted and untargeted metabolomics studies, by offering an R-based pipeline for data processing, particularly for pooled QCs filtration,"
- [readme] README: install GetFeatistics with devtools and requires R ≥ 4.3.1 and Java: "devtools::install_github("FrigerioGianfranco/GetFeatistics", dependencies = TRUE)"
- [readme] README: vignette guides through the workflow step by step: [evidence span withheld — anchor status: external_doc]
