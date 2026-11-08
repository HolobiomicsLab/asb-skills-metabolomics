---
name: two-sample-t-test-rank-comparison
description: Use when two reproducibility methods (e.g., MaRR top-k sets vs. copula
  mixture model sets) yield different top-k transcript sets and you need to judge
  which method's method-specific genes are more biologically relevant against a quantitative
  gold-standard rank list.
when_to_use_negative:
- When no external gold-standard ranking with per-transcript quantitative measurements
  exists — the t-test compares PCR ranks, so without such measurements there is nothing
  to test
- When both methods declare the same top-k transcript sets (no method-specific genes),
  leaving zero genes on one side of the t-test
- When the goal is to determine which transcripts are reproducible in the first place
  — use the MaRR procedure itself rather than this post-hoc validation comparison
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SciPy
  role: Runs the two-sample t-tests comparing PCR-measurement ranks of MaRR-specific
    versus copula-specific PCR genes
- name: pandas
  role: Loads the top-k sets and PCR measurements and assembles the summary table
    written to pcr_validation_results.csv
- name: NumPy
  role: Supports array handling of the PCR ranks and per-method gene counts during
    intersection and rank extraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:68799050e7c72e54bc5cdd46910bf33b799f73371fb601b1c91f73f789e40161
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# two-sample-t-test-rank-comparison

## Summary

A rank-comparison validation procedure that scores method-specific gene sets against an external gold-standard ranking (e.g., Taqman PCR ranks on SEQC transcripts) using two-sample t-tests on the PCR-measurement ranks of each method's uniquely identified genes.

## When to use

Use when two reproducibility methods (e.g., MaRR top-k sets vs. copula mixture model sets) yield different top-k transcript sets and you need to judge which method's method-specific genes are more biologically relevant against a quantitative gold-standard rank list.

## When NOT to use

- When no external gold-standard ranking with per-transcript quantitative measurements exists — the t-test compares PCR ranks, so without such measurements there is nothing to test
- When both methods declare the same top-k transcript sets (no method-specific genes), leaving zero genes on one side of the t-test
- When the goal is to determine which transcripts are reproducible in the first place — use the MaRR procedure itself rather than this post-hoc validation comparison

## Inputs

- SEQC Taqman PCR measurements: 1129 transcripts measured in four technical replicates
- Top-k reproducible transcript sets from the MaRR procedure (k = 38975, 38778, 27020, 40855, 40760, 28798 for SEQC comparisons 1-6 at alpha = 0.01)
- Top-k reproducible transcript sets from the copula mixture model at matched k for the six SEQC comparisons

## Outputs

- Summary table of commonly identified PCR gene counts per comparison (pcr_validation_results.csv)
- Method-specific gene identifiers with their PCR-measurement ranks per comparison
- Two-sample t-test p-values for rank comparisons (expected: 0.0227, 0.0047, 0.2176, 0.1235 for comparisons 1, 2, 3, 6)

## How to apply

Average the gold-standard measurements over replicates (SEQC Taqman PCR: 1129 transcripts, four technical replicates) and rank transcripts by the average. Load each method's top-k reproducible sets (for MaRR, k equals the number declared reproducible at alpha = 0.01: 38,975; 38,778; 27,020; 40,855; 40,760; 28,798 for the six SEQC comparisons; for the competitor, match k). Intersect each set with the 1129 PCR transcripts to compute commonly identified PCR gene counts (expected 1101, 1099, 851, 1120, 1119, 877 for comparisons 1-6) and extract the method-specific genes with their PCR ranks. Run two-sample t-tests comparing MaRR-specific versus copula-specific PCR ranks for comparisons 1, 2, 3, and 6 with SciPy, expecting p-values 0.0227, 0.0047, 0.2176, and 0.1235; smaller PCR ranks indicate better biological relevance, so a significant test favors the method whose unique genes rank higher.

## Related tools

- **SciPy** (Runs the two-sample t-tests comparing PCR-measurement ranks of MaRR-specific versus copula-specific PCR genes)
- **pandas** (Loads the top-k sets and PCR measurements and assembles the summary table written to pcr_validation_results.csv)
- **NumPy** (Supports array handling of the PCR ranks and per-method gene counts during intersection and rank extraction)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(marr_specific_pcr_ranks, copula_specific_pcr_ranks)
```

## Evaluation signals

- Common-count rows reproduce the reported values 1101, 1099, 851, 1120, 1119, 877 for SEQC comparisons 1-6
- T-test p-values match 0.0227, 0.0047, 0.2176, and 0.1235 within rounding tolerance
- Every PCR gene counted in the common and method-specific tallies belongs to the 1129-transcript Taqman PCR set
- The copula mixture model's k values exactly match MaRR's declared reproducibility counts at alpha = 0.01 for each comparison
- pcr_validation_results.csv contains six common-count rows and four p-value entries (comparisons 1, 2, 3, 6)

## Limitations

- Only comparisons 1, 2, 3, and 6 yield t-test p-values in the reference workflow (0.0227, 0.0047, 0.2176, 0.1235); comparisons 4 and 5 are summarized by common counts only
- The test is meaningful only relative to the Taqman PCR gold standard on the 1129 measured transcripts; it does not assess reproducibility itself
- One p-value (0.2176 for comparison 3) is non-significant, so a non-significant result does not by itself refute either method

## Evidence

- [methods] SEQC Taqman PCR measurements and replicate averaging: [evidence span withheld — anchor status: not_found]
- [methods] t-test comparison of method-specific PCR ranks: [evidence span withheld — anchor status: not_found]
