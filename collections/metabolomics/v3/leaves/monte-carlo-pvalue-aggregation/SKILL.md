---
name: monte-carlo-pvalue-aggregation
description: Use when benchmarking how robustly ORA, mPLAGE, and GSEA rank a known-significant
  pathway under synthetic noise — e.g. decoy features injected at 0%–1000% levels
  in a PALS-format peak-intensity and KEGG annotation matrix.
when_to_use_negative:
- When you need a single deterministic pathway ranking of real data rather than a
  noise-robustness benchmark with repeated stochastic runs
- When peak features lack compound annotations — unannotated peaks are excluded from
  PALS pathway analysis, so the benchmark cannot be run
edam_operation: http://edamontology.org/operation_2497
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: PALS
  role: Pipeline used to run PLAGE/mPLAGE, ORA, and GSEA pathway ranking over each
    repetition of the synthetic benchmark
  repo: github.com/glasgowcompbio/PALS
- name: pandas
  role: Passes intensity/annotation DataFrames programmatically and aggregates per-repetition
    p-values into median tables
  repo: github.com/glasgowcompbio/PALS
- name: SciPy
  role: Standardises the intensity matrix (zero mean, unit variance) via its preprocessing
    module before scoring
  repo: github.com/glasgowcompbio/PALS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:e8fb7bd7624b6f23f76aa31e9d831518ea9c08793d5facccbb3e310c2a9f4423
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2497
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# monte-carlo-pvalue-aggregation

## Summary

Benchmark pathway-ranking methods (PLAGE/mPLAGE, ORA, GSEA) by running the PALS pipeline over 500 independent repetitions per decoy-feature level, recording the target pathway's p-value per repetition, and aggregating to a median p-value that is compared against reference values within Monte Carlo tolerance.

## When to use

Use when benchmarking how robustly ORA, mPLAGE, and GSEA rank a known-significant pathway under synthetic noise — e.g. decoy features injected at 0%–1000% levels in a PALS-format peak-intensity and KEGG annotation matrix.

## When NOT to use

- When you need a single deterministic pathway ranking of real data rather than a noise-robustness benchmark with repeated stochastic runs
- When peak features lack compound annotations — unannotated peaks are excluded from PALS pathway analysis, so the benchmark cannot be run

## Inputs

- PALS-format peak-intensity CSV (row_id + sample columns, second line carrying group labels)
- PALS-format annotation CSV mapping peak IDs to KEGG compound IDs (e.g. C00148)
- Experimental design (comparisons of case vs control, e.g. Stage_1/Control)
- Synthetic decoy-feature levels (0% to 1000%) injected into background pathways

## Outputs

- Per-repetition target-pathway p-values for each method (PLAGE/mPLAGE, ORA, GSEA) and decoy level (500 repetitions each)
- benchmark_median_pvalues.csv — table of median p-values per method and decoy level

## How to apply

Generate the synthetic benchmark as a peak-intensity matrix plus KEGG compound annotation matrix in PALS input format (seven target pathways, 100 background pathways), injecting decoy features into background pathways at each decoy level from 0% to 1000%. For each repetition, run the chosen method (PLAGE/mPLAGE, ORA, or GSEA) through the PALS pipeline with --min_replace 5000 imputation, log-2 transformation, and SciPy standardisation (zero mean, unit variance). Record the target pathway's p-value for every repetition (500 per method per decoy level), then compute the median p-value per method and decoy level with pandas. Compare the reproduced medians against the reported references — ORA 2e-6 and mPLAGE 0.0049 at 0% decoy, 0.0083 and 0.0003 at 100% decoy, GSEA 0.0578 at 0% decoy — allowing Monte Carlo tolerance, and verify mPLAGE yields lower medians than ORA and GSEA at high decoy levels. Write the full results table to benchmark_median_pvalues.csv.

## Related tools

- **PALS** (Pipeline used to run PLAGE/mPLAGE, ORA, and GSEA pathway ranking over each repetition of the synthetic benchmark) — github.com/glasgowcompbio/PALS
- **pandas** (Passes intensity/annotation DataFrames programmatically and aggregates per-repetition p-values into median tables) — github.com/glasgowcompbio/PALS
- **SciPy** (Standardises the intensity matrix (zero mean, unit variance) via its preprocessing module before scoring) — github.com/glasgowcompbio/PALS

## Examples

```
python pals/run.py PLAGE notebooks/test_data/HAT/int_df.csv notebooks/test_data/HAT/annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control Stage_2/Control --min_replace 5000
```

## Evaluation signals

- Median p-value at 0% decoy reproduces ORA 2e-6, mPLAGE 0.0049, and GSEA 0.0578 within Monte Carlo tolerance
- Median p-values at 100% decoy match ORA 0.0083 and mPLAGE 0.0003
- Exactly 500 repetitions per method per decoy level are recorded before aggregation
- mPLAGE yields lower median p-values than both ORA and GSEA at high decoy levels (250–1000%)
- Output file benchmark_median_pvalues.csv contains one median per method × decoy-level combination

## Limitations

- Results are stochastic: medians must be compared to reference values within Monte Carlo tolerance, not exactly
- The mPLAGE robustness advantage holds for missing features and noisy data; under complete annotations methods converge to similar medians
- No software version numbers or changelog are stated for PALS, so reproduction may be affected by library drift

## Evidence

- [abstract] mPLAGE robustness over ORA/GSEA: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given"
- [methods] SciPy standardisation step: "standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] PALS imputation rule: "if all of the samples in a single experimental factor have intensities of zero these are replaced by the minimum intensity value"
- [readme] p-value output columns: "Pathways are identified by their id and can be sorted by the `p-value` columns."
