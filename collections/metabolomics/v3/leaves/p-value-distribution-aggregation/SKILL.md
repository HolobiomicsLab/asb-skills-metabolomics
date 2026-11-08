---
name: p-value-distribution-aggregation
description: Use when comparing robustness of pathway ranking methods on metabolomics
  peak data by repeating an analysis over multiple random feature-removal replicates
  and summarising each method's p-value distribution as mean and variance.
when_to_use_negative:
- When the goal is a single-pathway analysis on complete, unperturbed data rather
  than a robustness benchmark across replicated perturbations.
- When comparing methods on metrics other than p-value distributions (e.g. runtime
  or memory), since this skill only aggregates p-value statistics.
- When p-values are not comparable across methods (e.g. mixing calibrated and uncalibrated
  scores), as mean/variance comparison becomes meaningless.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Provides the PLAGE/mPLAGE, ORA and GSEA pathway-ranking methods whose p-values
    are aggregated in the benchmark
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds intensity/annotation DataFrames passed to PALS and aggregates the collected
    p-values into mean/variance summaries
- name: SciPy
  role: Used for the preprocessing/standardisation applied to the intensity matrix
    before each replicate run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:1757240bb79cebaa41356c20a8414e68180475972429d5aed370aecfb134d396
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# p-value-distribution-aggregation

## Summary

Benchmarking skill that aggregates pathway p-values across many randomized replicates into per-method mean p-values and variances, to quantify robustness of metabolite-set ranking methods (mPLAGE vs ORA and GSEA) under controlled feature-missingness perturbations.

## When to use

Use when comparing robustness of pathway ranking methods on metabolomics peak data by repeating an analysis over multiple random feature-removal replicates and summarising each method's p-value distribution as mean and variance.

## When NOT to use

- When the goal is a single-pathway analysis on complete, unperturbed data rather than a robustness benchmark across replicated perturbations.
- When comparing methods on metrics other than p-value distributions (e.g. runtime or memory), since this skill only aggregates p-value statistics.
- When p-values are not comparable across methods (e.g. mixing calibrated and uncalibrated scores), as mean/variance comparison becomes meaningless.

## Inputs

- PALS pathway-ranking output DataFrames containing a p-value column, one per replicate and method (PLAGE/mPLAGE, ORA, GSEA)
- Perturbation specification: missingness levels (20%, 40%, 60%, 80%), number of replicates (500) and distinct random seeds
- Set of known true pathways and fixed decoy level (100%) for F1-score computation
- Pandas/SciPy environment for aggregation and variance calculation

## Outputs

- Aggregated per-method/per-missingness table of mean p-value and p-value variance (e.g. missing_feature_benchmark_results.csv)
- F1 scores per method at the fixed decoy level
- Robustness ranking conclusion (which method retains lower, tighter p-values as missingness grows)

## How to apply

Run each ranking method (pals.PLAGE, pals.ORA, pals.GSEA via the PALS DataSource API) on the same perturbed intensity matrix at every replicate — e.g. 500 repetitions with distinct random seeds for each missingness level in {20%, 40%, 60%, 80%} of annotated peak features, with the decoy metabolite-set level fixed at 100%. Collect the pathway p-value column from each run's output DataFrame. Aggregate per method and per missingness level the mean p-value and the variance of p-values across replicates, and optionally compute F1 scores against the known true pathways for the fixed decoy level. Interpret lower mean p-values with smaller variance as the more robust method; in the PALS paper this criterion showed mPLAGE degrading more gracefully than ORA and GSEA as missing features increased. Write the aggregated per-method/per-missingness table (e.g. missing_feature_benchmark_results.csv) as the reusable benchmark artifact.

## Related tools

- **PALS** (Provides the PLAGE/mPLAGE, ORA and GSEA pathway-ranking methods whose p-values are aggregated in the benchmark) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds intensity/annotation DataFrames passed to PALS and aggregates the collected p-values into mean/variance summaries)
- **SciPy** (Used for the preprocessing/standardisation applied to the intensity matrix before each replicate run)

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- The aggregated table shows the target method (mPLAGE) with consistently lower mean p-values than ORA and GSEA at every missingness level from 20% to 80%.
- The target method also shows smaller p-value variance than the alternatives as missingness increases.
- The target method achieves the highest F1 scores against the known true pathways at the fixed 100% decoy level.
- Results are reproducible across 500 distinct random seeds per missingness level.
- The output CSV contains one aggregated row per (method, missingness-level) combination.

## Limitations

- Conclusions depend on the synthetic missing-features design (fixed 100% decoy level, random feature removal); results may not transfer to real noise structures.
- PALS reports no software version numbers and no changelog, so exact-version reproducibility of the benchmark is hard to pin down.
- Comparison is limited to the three methods included in PALS (PLAGE/mPLAGE, ORA, GSEA); other pathway-ranking approaches are not covered.

## Evidence

- [abstract] A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [methods] standardised using the preprocessing module in Scipy: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance across the samples."
- [intro] The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA).: "The results are found to be more robust to noise and missing peaks compared to the alternatives (ORA, GSEA)."
- [readme] python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG: "python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control"
