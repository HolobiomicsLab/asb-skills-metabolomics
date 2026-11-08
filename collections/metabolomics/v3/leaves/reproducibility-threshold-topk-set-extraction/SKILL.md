---
name: reproducibility-threshold-topk-set-extraction
description: Use when replicate rank lists have already been scored by MaRR and a
  competing method, and you need per-method top-k gene sets of equal size to compare
  against an external benchmark.
when_to_use_negative:
- When the goal is to rank a fresh list of transcripts by reproducibility from scratch
  rather than extracting already-declared top-k sets for gold-standard validation.
- When no external gold standard (e.g. Taqman PCR measurements) is available to intersect
  against the extracted sets.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3577
tools:
- name: SciPy
  role: Two-sample t-tests comparing PCR-measurement ranks of method-specific PCR
    genes
- name: pandas
  role: Assembling the common-count and method-specific PCR gene table and writing
    pcr_validation_results.csv
- name: NumPy
  role: Ranking transcripts by averaged PCR values and intersecting top-k sets with
    PCR gene identifiers
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:844f12d37697b80843e667413cbe20368859b8c546563dee7e4c742998302a56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reproducibility-threshold-topk-set-extraction

## Summary

Extracts the top-k reproducible transcript sets declared by MaRR and the copula mixture model at matched k values, so their overlap with an external gold standard (Taqman PCR) can be quantified per SEQC comparison.

## When to use

Use when replicate rank lists have already been scored by MaRR and a competing method, and you need per-method top-k gene sets of equal size to compare against an external benchmark.

## When NOT to use

- When the goal is to rank a fresh list of transcripts by reproducibility from scratch rather than extracting already-declared top-k sets for gold-standard validation.
- When no external gold standard (e.g. Taqman PCR measurements) is available to intersect against the extracted sets.

## Inputs

- SEQC RNA-seq replicate rank lists per comparison
- MaRR declared reproducible transcript sets at alpha = 0.01
- Taqman PCR measurements for 1129 transcripts across four technical replicates
- copula mixture model ranked transcript lists

## Outputs

- per-method top-k reproducible transcript sets (six SEQC comparisons)
- common counts of PCR genes per comparison (1101, 1099, 851, 1120, 1119, 877)
- method-specific PCR gene identifiers with PCR-measurement ranks
- two-sample t-test p-values (0.0227, 0.0047, 0.2176, 0.1235) for comparisons 1, 2, 3, and 6
- pcr_validation_results.csv summary table

## How to apply

Take the MaRR procedure's declared reproducible transcripts at alpha = 0.01 and set k to their count for each SEQC comparison (38975, 38778, 27020, 40855, 40760, 28798). Apply the same k to the copula mixture model's ranked list to obtain matched-size top-k sets. Intersect each method's top-k set with the 1129 Taqman PCR transcripts (averaged over four technical replicates) and record the counts of commonly identified PCR genes for comparisons 1-6, expecting 1101, 1099, 851, 1120, 1119, 877. Then, for the method-specific PCR genes in comparisons 1, 2, 3, and 6, compare their PCR-measurement ranks between MaRR and the copula model using two-sample t-tests, expecting p-values 0.0227, 0.0047, 0.2176, and 0.1235.

## Related tools

- **SciPy** (Two-sample t-tests comparing PCR-measurement ranks of method-specific PCR genes)
- **pandas** (Assembling the common-count and method-specific PCR gene table and writing pcr_validation_results.csv)
- **NumPy** (Ranking transcripts by averaged PCR values and intersecting top-k sets with PCR gene identifiers)

## Examples

```
from scipy import stats; import pandas as pd; counts, pvals = pcr_validation_results.csv, stats.ttest_ind(marr_ranks, copula_ranks)
```

## Evaluation signals

- The six common-count rows reproduce the reported values 1101, 1099, 851, 1120, 1119, 877 exactly.
- The four t-test p-values match 0.0227, 0.0047, 0.2176, and 0.1235 within rounding tolerance.
- k values used for MaRR equal the numbers of declared reproducible transcripts at alpha = 0.01 (38975, 38778, 27020, 40855, 40760, 28798).
- pcr_validation_results.csv is produced and passes the rounding-tolerance check on all ten reported statistics.

## Limitations

- Counts and p-values are specific to the six SEQC comparisons; the skill presumes the exact alpha = 0.01 MaRR threshold and matched-k design.
- Method-specific PCR genes for comparisons 4 and 5 lack t-test p-values in the reported summary, so statistical comparison is incomplete for those two comparisons.

## Evidence

- [methods] k set to the number of transcripts declared reproducible at alpha = 0.01: 38,975, 38,778, 27,020, 40,855, 40,760, 28,798: [evidence span withheld — anchor status: not_found]
- [methods] Intersect each method's top-k set with the 1129 PCR transcripts and record the counts: [evidence span withheld — anchor status: not_found]
- [methods] two-sample t-tests (expected p-values: 0.0227, 0.0047, 0.2176, 0.1235): [evidence span withheld — anchor status: not_found]
- [results] MaRR identifies more PCR genes than the copula mixture model in all six SEQC comparisons: [evidence span withheld — anchor status: not_found]
