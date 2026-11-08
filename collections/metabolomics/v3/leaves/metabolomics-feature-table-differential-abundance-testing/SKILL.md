---
name: metabolomics-feature-table-differential-abundance-testing
description: Use when a negative-ion-mode LC-MS feature table with grouped samples
  (e.g. 94 samples across 14 lamprey tissues) must be tested for features selectively
  enriched in one group versus all others at a controlled FDR.
when_to_use_negative:
- The input is already a filtered differential abundance table rather than a raw/normalized
  feature table with group labels
- The analysis requires feature identification or annotation rather than statistical
  differential abundance testing
- The design is not a one-group-versus-rest comparison (e.g. paired or multi-factor
  models that MetaboReport's between-group test helpers do not cover)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: Interactive R Shiny app whose statistical analysis functions (R/fct_*.R analysis
    helpers) perform the between-group abundance test and feed the HTML report
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment in which the feature table is loaded, the FDR-adjusted
    test is run, fold changes are computed, and the filtered CSV is exported
- name: golem
  role: Production-grade modular schema underlying MetaboReport, separating analysis
    helpers (fct_*.R) from Shiny modules (mod_*.R)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:da2c5ecf98b892ec27468a93a6d66f82fae501da268ba4a4c0461f00471dd0ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - golem
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolomics-feature-table-differential-abundance-testing

## Summary

Run between-group differential abundance testing on an LC-MS metabolomics feature table using MetaboReport's statistical analysis helpers in R, applying Benjamini-Hochberg FDR correction and abundance fold-change filtering to isolate tissue-enriched mass features.

## When to use

Use when a negative-ion-mode LC-MS feature table with grouped samples (e.g. 94 samples across 14 lamprey tissues) must be tested for features selectively enriched in one group versus all others at a controlled FDR.

## When NOT to use

- The input is already a filtered differential abundance table rather than a raw/normalized feature table with group labels
- The analysis requires feature identification or annotation rather than statistical differential abundance testing
- The design is not a one-group-versus-rest comparison (e.g. paired or multi-factor models that MetaboReport's between-group test helpers do not cover)

## Inputs

- Dataset 1 feature table from MetaboReport example data (cancerCell dataset), negative-ion-mode LC-MS conditions
- Sample metadata with group/tissue labels (94 samples, 14 lamprey tissue groups)

## Outputs

- differential_abundance_buccal_gland.csv — filtered differential abundance table with 182 rows at FDR-adjusted P<0.001
- Between-group test P values with Benjamini-Hochberg FDR adjustment and per-feature fold changes

## How to apply

Load the feature table (e.g. the cancerCell example dataset shipped with MetaboReport under negative-ion-mode conditions) into R and start MetaboReport via run_app(). Use the package's internal statistical analysis helpers in R/fct_*.R to run the between-group abundance test comparing the group of interest (e.g. buccal gland) against the pooled remaining 13 tissue groups. Apply Benjamini-Hochberg FDR adjustment to the test P values, then compute per-feature abundance fold change between the group of interest and all others. Filter to features exceeding the enrichment threshold (here >1000-fold higher) and retain only those with FDR-adjusted P<0.001. Export the surviving rows as a differential abundance table (differential_abundance_buccal_gland.csv) and validate the row count against the expected metric (182 features).

## Related tools

- **MetaboReport** (Interactive R Shiny app whose statistical analysis functions (R/fct_*.R analysis helpers) perform the between-group abundance test and feed the HTML report) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment in which the feature table is loaded, the FDR-adjusted test is run, fold changes are computed, and the filtered CSV is exported)
- **golem** (Production-grade modular schema underlying MetaboReport, separating analysis helpers (fct_*.R) from Shiny modules (mod_*.R))

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- The exported differential_abundance_buccal_gland.csv contains exactly 182 rows
- Every retained row has FDR-adjusted P<0.001 under Benjamini-Hochberg correction
- Every retained row has abundance fold change >1000-times higher in buccal gland than in the other 13 tissues
- The test compares the buccal gland group against the pooled other 13 tissue groups from the 94-sample, 14-tissue negative-ion-mode dataset
- MetaboReport's regression tests pass (testthat::test_local()) after any modification of the analysis helpers

## Limitations

- MetaboReport's analysis helpers are internal by design — the supported public API is intentionally small (run_app()) — so direct programmatic reuse of fct_*.R functions may break without a compatibility contract
- The analysis helpers' implementation can evolve between releases (e.g. v0.3.6 'Faster in data analysis', 'Bug fixed in report generation'), which may change numerical behavior
- Validation is dataset-specific: the 182-row expectation applies only to the buccal-gland-tissue / negative-ion-mode conditions of this LC-MS dataset

## Evidence

- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [abstract] MetaboReport statistical analysis scope: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] fct_*.R analysis helpers and internal API: "R/fct_*.R` contains pure analysis, input/output, plotting, and report functions."
- [readme] App startup command: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
