---
name: decoy-feature-injection-benchmarking
description: Use when evaluating whether a pathway ranking method degrades gracefully
  as annotation completeness falls, i.e. when missing/noisy peak features must be
  simulated via decoy-feature injection into background pathways.
when_to_use_negative:
- Input is real experimental data with no known ground-truth pathway activity — decoy
  injection benchmarks require synthetic target and background pathways
- The goal is to rank pathways in a single analysis rather than to characterise method
  robustness across annotation-degradation levels
- Peak features are already fully annotated so annotation-completeness degradation
  is not the quantity under test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PALS
  role: Pipeline that loads the intensity and annotation matrices, performs imputation,
    log-2 transform and SciPy standardisation, and runs PLAGE, ORA and GSEA pathway
    ranking
  repo: https://github.com/glasgowcompbio/PALS
- name: pandas
  role: Holds int_df and annotation_df dataframes passed programmatically to the PALS
    DataSource; used to aggregate repetition p-values and write the benchmark CSV
- name: SciPy
  role: Standardises the log-2 transformed intensity matrix to zero mean and unit
    variance before pathway scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11020103
    title: Ranking Metabolite Sets by Their Activity Levels
  source_content_hash: sha256:7e5d2a6d7b162bd7d8c23ba6e83d78db7c22140d6e7170c5631ff9863d0f3fb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/PALS
  tools:
  - PALS
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# decoy-feature-injection-benchmarking

## Summary

Benchmarks pathway ranking methods (mPLAGE/PLAGE, ORA, GSEA) for robustness by progressively injecting decoy features into background pathways of a synthetic PALS-format dataset and comparing median pathway p-values across 500 repetitions per decoy level (0%–1000%).

## When to use

Use when evaluating whether a pathway ranking method degrades gracefully as annotation completeness falls, i.e. when missing/noisy peak features must be simulated via decoy-feature injection into background pathways.

## When NOT to use

- Input is real experimental data with no known ground-truth pathway activity — decoy injection benchmarks require synthetic target and background pathways
- The goal is to rank pathways in a single analysis rather than to characterise method robustness across annotation-degradation levels
- Peak features are already fully annotated so annotation-completeness degradation is not the quantity under test

## Inputs

- Synthetic peak-intensity matrix (PALS CSV format: row_id plus sample columns, with a group line mapping samples to experimental factors)
- KEGG compound annotation matrix (two-column CSV mapping peak row_id to KEGG entity_id)
- Experimental design dictionary of groups and case/control comparisons
- Decoy-feature injection levels (0% to 1000%) applied to 100 background pathways

## Outputs

- benchmark_median_pvalues.csv: full results table of median target-pathway p-values per method and decoy level
- Per-repetition p-values of the seven target pathways for PLAGE/mPLAGE, ORA, and GSEA

## How to apply

Generate a synthetic benchmark consisting of seven synthetic target pathways plus 100 background pathways, formatted as a peak-intensity matrix and a KEGG compound annotation matrix in PALS input format. At each decoy level from 0% to 1000%, inject decoy features into background pathways to progressively degrade annotation completeness. Run each ranking method (PLAGE/mPLAGE, ORA, GSEA) through the PALS pipeline with --min_replace 5000 imputation, log-2 transformation, and SciPy standardisation, over 500 independent repetitions per method and decoy level. Record the target-pathway p-value in every repetition and compute the median per method and decoy level; compare against reference medians (ORA 2e-6 and mPLAGE 0.0049 at 0% decoy; 0.0083 and 0.0003 at 100% decoy; GSEA 0.0578 at 0% decoy) within Monte Carlo tolerance. The rationale is that mPLAGE decomposes pathway activity levels and is more robust to missing features than ORA and GSEA, so its median p-values should stay low even at high decoy levels.

## Related tools

- **PALS** (Pipeline that loads the intensity and annotation matrices, performs imputation, log-2 transform and SciPy standardisation, and runs PLAGE, ORA and GSEA pathway ranking) — https://github.com/glasgowcompbio/PALS
- **pandas** (Holds int_df and annotation_df dataframes passed programmatically to the PALS DataSource; used to aggregate repetition p-values and write the benchmark CSV)
- **SciPy** (Standardises the log-2 transformed intensity matrix to zero mean and unit variance before pathway scoring)

## Examples

```
python pals/run.py PLAGE int_df.csv annotation_df.csv test_output.csv --db PiMP_KEGG --comparisons Stage_1/Control --min_replace 5000
```

## Evaluation signals

- At 0% decoy features, median p-values match reference values within Monte Carlo tolerance: ORA 2e-6, mPLAGE 0.0049, GSEA 0.0578
- At 100% decoy features, medians match reference values: ORA 0.0083 and mPLAGE 0.0003
- mPLAGE yields lower median p-values than ORA and GSEA at high decoy levels, consistent with its reported robustness to missing features
- All 500 repetitions per method and decoy level completed and each produces a defined target-pathway p-value
- benchmark_median_pvalues.csv contains one median p-value per method per decoy level from 0% to 1000%

## Limitations

- Results are Monte Carlo estimates over 500 repetitions, so medians must be compared within tolerance rather than exactly
- GSEA performs poorly on the synthetic benchmark even at 0% decoy (median p-value 0.0578), limiting its usefulness as a comparator at low decoy levels
- Peaks without compound annotations are excluded from pathway analysis, so annotation quality directly bounds what any method can detect

## Evidence

- [other] Reference medians at 0% decoy level: [evidence span withheld — anchor status: not_found]
- [abstract] mPLAGE robustness finding: "A comparison of mPLAGE with two other commonly used methods (overrepresentation analysis (ORA) and gene set enrichment analysis (GSEA)) is also given and reveals that mPLAGE is more robust to missing"
- [readme] Preprocessing steps applied by PALS: "The data is subsequently transformed to log-2 base and standardised using the preprocessing module in Scipy such that the intensity matrix has a zero mean and unit variance"
- [readme] Default imputation parameter: "--min_replace: The minimum intensity value for data imputation, e.g. `--min_replace 5000`. Defaults to 5000."
- [readme] Unannotated peaks excluded from analysis: "users also provide a list of compound annotations assigned to peak features (peaks that do not have annotations will not be used for pathway analysis)"
