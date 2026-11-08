---
name: pairwise-group-comparison-statistics
description: Use when a pre-processed (missing-value-handled, log-transformed, pareto-scaled)
  targeted or untargeted metabolomics feature table must be compared across sample
  groups to identify features with statistically significant and practically relevant
  abundance changes.
when_to_use_negative:
- The feature table is raw and still contains missing values and non-normal (non-log,
  non-pareto-scaled) intensities — the univariate tests assume transf_data has already
  been applied
- The analysis goal is longitudinal or censored-value modelling (mixed-effects or
  Tobit regression) rather than simple between-group pairwise comparison — GetFeatistics
  routes those to its regression functions instead
- Feature lists have not been filtered against pooled-QC criteria, so univariate testing
  would run on low-reproducibility features
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: GetFeatistics
  role: Provides the gentab_P.t.test, gentab_FC, gentab_P.1wayANOVA_posthocTukeyHSD,
    gentab_P.2wayANOVA_posthocTukeyHSD and Volcano_ttest_FC functions that execute
    the pairwise comparisons and their visualisation
  repo: https://github.com/FrigerioGianfranco/GetFeatistics
- name: R
  role: Runtime language in which GetFeatistics (and hence the t-test/FC/ANOVA routines)
    is implemented; requires R ≥ 4.3.1
- name: ggplot
  role: Underlying grammar for the customisable volcano-plot output produced from
    the t-test and fold-change results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1515/jib-2025-0047
    title: 'Streamlining feature elaboration and statistics analysis in metabolomics:
      the GetFeatistics R-package'
  source_content_hash: sha256:9de4974fcbca93428a64638cbd2e9d486f4b56c7869d1eec3304f2b5cd60061b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrigerioGianfranco/GetFeatistics
  tools:
  - GetFeatistics
  - R
  - ggplot
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Pairwise group comparison statistics

## Summary

Runs univariate pairwise group comparisons on metabolomics feature tables — Student's t-tests, fold-change calculations, and one/two-way ANOVA with post-hoc Tukey HSD — as implemented by the GetFeatistics R-package's gentab functions, with FDR multiple-testing correction and volcano-plot output.

## When to use

Use when a pre-processed (missing-value-handled, log-transformed, pareto-scaled) targeted or untargeted metabolomics feature table must be compared across sample groups to identify features with statistically significant and practically relevant abundance changes.

## When NOT to use

- The feature table is raw and still contains missing values and non-normal (non-log, non-pareto-scaled) intensities — the univariate tests assume transf_data has already been applied
- The analysis goal is longitudinal or censored-value modelling (mixed-effects or Tobit regression) rather than simple between-group pairwise comparison — GetFeatistics routes those to its regression functions instead
- Feature lists have not been filtered against pooled-QC criteria, so univariate testing would run on low-reproducibility features

## Inputs

- Transformed (transf_data-processed) targeted or untargeted feature-intensity table (features × samples)
- Sample metadata / phenotype table mapping samples to comparison groups (e.g. df_example_sample_data)
- Design specification: which group pairs to compare (t-test/FC) or one-way vs two-way factor layout (ANOVA)

## Outputs

- gentab_P.t.test table: pairwise Student's t-test p-values with FDR correction
- gentab_FC table: pairwise fold changes per feature
- gentab_P.1wayANOVA_posthocTukeyHSD table: one-way ANOVA p-values plus post-hoc Tukey HSD comparisons, FDR-corrected
- gentab_P.2wayANOVA_posthocTukeyHSD table: two-factor ANOVA with Tukey HSD post-hoc results
- Descriptive-statistics table from gentab_descr (group means/spread on transformed data)
- Volcano plot (Volcano_ttest_FC): ggplot graphic combining t-test significance and fold-change magnitude

## How to apply

First ensure the feature table has been transformed via GetFeatistics' transf_data function, which replaces missing values, log-transforms intensities, and applies pareto scaling — pairwise tests assume data in this state. Run gentab_P.t.test on the transformed table together with the sample-metadata table (df_example_sample_data-style phenotype/group assignments) to obtain pairwise Student's t-test p-values, which are automatically FDR-corrected to control the false discovery rate across the full feature list. Pair this with gentab_FC on the same inputs to compute fold changes for every pair of groups, giving both statistical and effect-size views of each feature. For designs with more than two groups or a two-factor layout, call gentab_P.1wayANOVA_posthocTukeyHSD or gentab_P.2wayANOVA_posthocTukeyHSD respectively; these produce ANOVA tables followed by post-hoc Tukey HSD comparisons, also with FDR correction. Finally, feed the paired t-test and fold-change results into the package's volcano-plot function (Volcano_ttest_FC), a ggplot-based visualisation, to display features that pass both significance and magnitude thresholds. Judge correctness by checking that every table row carries both a raw and FDR-corrected p-value, that fold-change directions match the group ordering, and that volcano-plot points reflect the underlying t-test/FC values.

## Related tools

- **GetFeatistics** (Provides the gentab_P.t.test, gentab_FC, gentab_P.1wayANOVA_posthocTukeyHSD, gentab_P.2wayANOVA_posthocTukeyHSD and Volcano_ttest_FC functions that execute the pairwise comparisons and their visualisation) — https://github.com/FrigerioGianfranco/GetFeatistics
- **R** (Runtime language in which GetFeatistics (and hence the t-test/FC/ANOVA routines) is implemented; requires R ≥ 4.3.1)
- **ggplot** (Underlying grammar for the customisable volcano-plot output produced from the t-test and fold-change results)

## Examples

```
library(GetFeatistics); transformed <- transf_data(feature_table, sample_table); t_test_tab <- gentab_P.t.test(transformed, sample_table, factor); fc_tab <- gentab_FC(transformed, sample_table, factor); Volcano_ttest_FC(t_test_tab, fc_tab)
```

## Evaluation signals

- Every feature row in the gentab_P.t.test output carries both a raw p-value and an FDR-corrected p-value; absence of either indicates the correction step was skipped
- gentab_FC fold-change signs and magnitudes are consistent with group-mean differences in the gentab_descr descriptive table (direction should match group ordering)
- ANOVA outputs include post-hoc Tukey HSD comparison columns alongside omnibus p-values; missing Tukey columns signal the wrong gentab variant was used
- Volcano-plot x/y coordinates reproduce the gentab_FC and gentab_P.t.test values for a spot-checked subset of features
- Re-running on the vignette's mock data (df_example_sample_data joined with the targeted and untargeted feature tables) reproduces the same table structure and significance calls

## Limitations

- Requires the transf_data transformation (missing-value replacement, log-transform, pareto scaling) beforehand; running the tests on raw intensities violates the assumed data state
- All pairwise tests sit atop a large feature list, so FDR correction is essential — the package handles this internally but the number of significant calls shrinks with feature count
- Post-hoc Tukey HSD is the only pairwise exploration offered for ANOVA designs; other post-hoc contrasts are not part of this skill

## Evidence

- [other] Vignette pairwise-comparison workflow on combined targeted/untargeted mock data: [evidence span withheld — anchor status: not_found]
- [abstract] Output visualisations are ggplot-based and customisable: "Output tables are concise and informative, facilitating interpretation and reporting, while output visualisations are fully customisable via the ggplot grammar."
- [intro] Data intensity pre-processing precedes statistics: "data intensities are pre-processed to deal with missing values and data normality"
- [intro] Statistical analyses follow pre-processing to highlight relevant associations: "and then statistical analyses are applied to highlight biologically relevant associations"
- [other] Volcano plot combines t-test and fold-change results: [evidence span withheld — anchor status: not_found]
- [discussion] Pooled-QC filtration should precede feature elaboration: "the filtration of features considering pooled QC criteria, including also the use separated pooled QC per sample group"
- [discussion] Mixed-effect and Tobit models are distinct from pairwise comparisons: "the efficient construction of multiple regression models with mixed-effect and Tobit models"
- [readme] GetFeatistics installation requirements: "R (version ≥ 4.3.1)
- Java, with the same architecture as R (64-bit or 32-bit).
- git"
