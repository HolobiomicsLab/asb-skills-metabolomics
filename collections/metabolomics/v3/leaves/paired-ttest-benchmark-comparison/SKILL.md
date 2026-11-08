---
name: paired-ttest-benchmark-comparison
description: Use when two spectral-processing variants were applied to the same compounds/samples
  and you need a statistically grounded per-compound comparison of paired scores (e.g.,
  matching scores with vs. without PerformDDADeconvolution).
when_to_use_negative:
- Scores cannot be paired compound-by-compound (different compounds identified in
  each arm) — use an unpaired test or rank-based comparison instead
- The comparison is between independent datasets or different instruments rather than
  two processing variants of the same data
- Input is a feature-abundance table rather than per-compound identification scores;
  use metabolite quantification QC (e.g., dilution series) instead of score pairing
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetaboAnalystR 4.0
  role: Generates both arms of paired MS2 matching scores via PerformDDADeconvolution,
    PerformDBSearchingBatch, and PerformResultsExport
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Runs the paired one-tailed Student's t-test (stats package t.test) on the
    paired score tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:7951b4065b2f8c7f1353fbde52a6d193da1c258b1a212fd90c06d2b0dca34a77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# paired-ttest-benchmark-comparison

## Summary

A benchmark-comparison method that pairs per-compound MS2 matching scores from two processing variants (e.g., with vs. without DDA deconvolution in MetaboAnalystR 4.0) and tests the improvement with a paired one-tailed Student's t-test, as used to validate DDA deconvolution on the ESI+ standards mixture (n = 99, p = 1.5e-7).

## When to use

Use when two spectral-processing variants were applied to the same compounds/samples and you need a statistically grounded per-compound comparison of paired scores (e.g., matching scores with vs. without PerformDDADeconvolution).

## When NOT to use

- Scores cannot be paired compound-by-compound (different compounds identified in each arm) — use an unpaired test or rank-based comparison instead
- The comparison is between independent datasets or different instruments rather than two processing variants of the same data
- Input is a feature-abundance table rather than per-compound identification scores; use metabolite quantification QC (e.g., dilution series) instead of score pairing

## Inputs

- Compound-level matching-score table from the deconvoluted arm (PerformResultsExport output; scores 0-100, one row per identified compound)
- Matching compound-level table from the non-deconvolved arm (same schema, same compounds)
- Direction of expected effect (one-tailed: deconvolution improves scores)

## Outputs

- Paired per-compound score differences (deconvoluted minus non-deconvoluted)
- Mean score improvement across the n = 99 paired compounds
- Paired one-tailed Student's t-test statistic and p-value (expected p = 1.5e-7 for the standards-mixture benchmark)

## How to apply

Run the identical downstream pipeline on both arms — here PerformDBSearchingBatch with the MS-DIAL scoring rule Matching Score = (MS2 similarity + MS1 similarity + RT similarity + 0.5 x isotope similarity)/3.5 x 100 — differing only in the upstream step (deconvolution via PerformDDADeconvolution vs. none). Export compound-level results from both arms with PerformResultsExport and pair rows per compound (n = 99 compounds identified in ESI+ mode of the 1st standards mixture). Verify both score tables share the 0-100 scale and the same compound keys before pairing. Then run a paired one-tailed Student's t-test in the R stats package (t.test scores_decon, scores_nodecon, paired=TRUE, alternative='greater') and report the mean score improvement and p-value; in the source study this yielded p = 1.5e-7, consistent with the reported Fig. 4a result.

## Related tools

- **MetaboAnalystR 4.0** (Generates both arms of paired MS2 matching scores via PerformDDADeconvolution, PerformDBSearchingBatch, and PerformResultsExport) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Runs the paired one-tailed Student's t-test (stats package t.test) on the paired score tables)

## Examples

```
t.test(scores_decon$MatchingScore, scores_nodecon$MatchingScore, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- Paired score table contains exactly 99 rows (one per compound identified in ESI+ mode of the 1st standards mixture) with matched keys across both arms
- All matching scores fall on the 0-100 scale in both arms
- Paired one-tailed t-test yields a statistically significant improvement for deconvolution (reproducing p = 1.5e-7, matching Fig. 4a)
- Mean score difference is positive (deconvolution arm > non-deconvolution arm)
- Both arms used the identical PerformDBSearchingBatch call and SQLite reference library, differing only in the deconvolution step

## Limitations

- The p = 1.5e-7 result is reported unadjusted for multiple testing (significant, no adjustment); corrected comparisons may lose significance
- Benchmark is specific to the standards mixture (ESI+) dataset from Metabolomics Workbench ST002796; effect size may differ on other matrices or ionization modes
- One-tailed test presupposes the direction of improvement; it cannot detect a significant degradation

## Evidence

- [results] DDA deconvolution significantly improved the MS2 matching scores of chemical candidates relative to non-deconvolved spectra: [evidence span withheld — anchor status: not_found]
- [methods] Search the deconvoluted spectra against the curated SQLite MS2 reference library with PerformDBSearchingBatch: [evidence span withheld — anchor status: not_found]
- [methods] The database searching is performed via SQLite query and can be achieved with the function, PerformDBSearchingBatch.: "The database searching is performed via SQLite query and can be achieved with the function, PerformDBSearchingBatch."
- [methods] The database search results can be exported as a data frame table using the PerformResultsExport function.: "The database search results can be exported as a data frame table using the PerformResultsExport function."
- [readme] Benchmarking DDA/DIA deconvolution claims significant improvement in true positive identification without extra false positives: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
