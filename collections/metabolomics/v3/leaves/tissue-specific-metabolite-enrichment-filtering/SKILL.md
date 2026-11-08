---
name: tissue-specific-metabolite-enrichment-filtering
description: Use when a negative-ion-mode LC-MS feature table spans multiple tissue
  groups and you need features with extreme, statistically controlled enrichment in
  one tissue versus every other group.
when_to_use_negative:
- The goal is generic between-tissue differential abundance at moderate effect sizes
  (e.g. 2-fold, P<0.05) rather than extreme single-tissue enrichment
- Input data is raw mass-spectrometry signal that still requires MetaboReport's preprocessing
  step — this skill starts from an already-processed feature table
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: R Shiny app providing the statistical analysis functions (R/fct_*.R helpers)
    used to run the between-group abundance test and produce the differential abundance
    results
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment in which the MetaboReport package is installed and the
    filtering/export script is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:e1566ef49ba3cda86cebb19b2d5556a6eaa401f8717b787830a98a064c18fbcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tissue-specific-metabolite-enrichment-filtering

## Summary

Identifies mass features drastically enriched in one tissue versus all others in an LC-MS spatial metabolomics dataset by combining between-group statistical testing with Benjamini-Hochberg FDR adjustment and an extreme fold-change cutoff (e.g. >1000-fold, FDR P<0.001), as demonstrated for 182 buccal-gland-enriched features in lamprey.

## When to use

Use when a negative-ion-mode LC-MS feature table spans multiple tissue groups and you need features with extreme, statistically controlled enrichment in one tissue versus every other group.

## When NOT to use

- The goal is generic between-tissue differential abundance at moderate effect sizes (e.g. 2-fold, P<0.05) rather than extreme single-tissue enrichment
- Input data is raw mass-spectrometry signal that still requires MetaboReport's preprocessing step — this skill starts from an already-processed feature table

## Inputs

- LC-MS negative-ion-mode feature table (samples x mass features) with tissue group annotations (e.g. 94 samples, 14 lamprey tissues)
- MetaboReport example dataset 1 (cancerCell) or an equivalent metabolomics feature-intensity table in R

## Outputs

- Filtered differential abundance table of tissue-enriched mass features passing the fold-change and FDR cutoffs (e.g. differential_abundance_buccal_gland.csv with exactly 182 rows at FDR P<0.001)
- Per-feature between-group test P values with Benjamini-Hochberg FDR adjustment

## How to apply

Load the feature table (e.g. MetaboReport's Dataset 1 shipped under negative-ion-mode conditions) into R and run a between-group abundance test comparing the target tissue group (e.g. buccal gland) against the pooled remaining 13 tissue groups using MetaboReport's R/fct_*.R statistical analysis helpers. Apply Benjamini-Hochberg FDR adjustment to the between-group P values, then compute the per-feature abundance fold change of the target tissue versus all other tissues. Filter to features more than 1000-times higher in the target tissue and retain only those with FDR-adjusted P<0.001. Export the filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) and validate the row count against the expected metric.

## Related tools

- **MetaboReport** (R Shiny app providing the statistical analysis functions (R/fct_*.R helpers) used to run the between-group abundance test and produce the differential abundance results) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment in which the MetaboReport package is installed and the filtering/export script is executed)

## Examples

```
Rscript -e 'remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()'
```

## Evaluation signals

- The exported differential_abundance table contains exactly 182 rows for the buccal-gland-tissue / negative-ion-mode conditions
- Every retained row satisfies FDR-adjusted P<0.001 after Benjamini-Hochberg correction
- Every retained feature has abundance more than 1000-times higher in the target tissue than in the other 13 tissues
- The between-group test is run against the pooled other-tissue groups, not pairwise per tissue, and the comparison uses the negative-ion-mode feature table

## Limitations

- The 182-feature / >1000-fold / FDR P<0.001 result is specific to the buccal-gland-tissue and negative-ion-mode conditions; other tissues, ion modes, or datasets will yield different counts
- Analysis helpers in R/fct_*.R are internal to MetaboReport (public API is intentionally limited to run_app()), so their implementation can evolve without an explicit compatibility contract

## Evidence

- [other] 182 mass features were over 1000-times higher in the buccal gland compared to the other 13 tissues, with a statistically significant FDR-adjusted P-value of <0.001.: [evidence span withheld — anchor status: not_found]
- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] The supported public package API is intentionally small: run_app(). Analysis helpers are internal so their implementation can evolve without creating an accidental compatibility contract.: "The supported public package API is intentionally small: `run_app()`. Analysis
helpers are internal so their implementation can evolve without creating an
accidental compatibility contract."
- [readme] `R/fct_*.R` contains pure analysis, input/output, plotting, and report functions.: "- `R/fct_*.R` contains pure analysis, input/output, plotting, and report
  functions."
- [methods] cancerCell (MetaboReport dataset) cited from the methods section as the example dataset.: "- `cancerCell (MetaboReport)`"
