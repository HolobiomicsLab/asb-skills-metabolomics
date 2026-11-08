---
name: replicate-averaging-and-ranking-of-expression-measurements
description: Use when replicate rank lists (e.g., the six SEQC RNA-seq replicate comparisons)
  must be reduced to per-gene average scores and ranks to test whether a reproducibility
  method's top-k sets are enriched for gold-standard genes such as the 1129 SEQC Taqman
  PCR transcripts.
when_to_use_negative:
- Input is raw sequencing data or un-ranked expression intensities rather than per-transcript
  replicate measurements that can be averaged and ranked
- There is only a single experiment (no replicate comparisons), since the skill evaluates
  overlap between replicate-derived top-k sets against PCR ranks
- The analysis goal is parametric modeling of between-replicate dependence; the MaRR
  framework is explicitly nonparametric and makes no parametric assumptions on dependence
  structures
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Load the SEQC Taqman PCR replicate measurements and top-k transcript sets,
    average PCR values over the four replicates, and assemble the summary table written
    to pcr_validation_results.csv
- name: SciPy
  role: Run two-sample t-tests on PCR ranks of MaRR-specific versus copula-specific
    PCR genes for comparisons 1, 2, 3, and 6
- name: NumPy
  role: Numeric ranking of transcripts by averaged PCR values and array manipulation
    during top-k set intersection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:a7772fcef8162912afcf3666c431affcadf4ca416bb3bc3ad16cab221ac4c8f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# replicate-averaging-and-ranking-of-expression-measurements

## Summary

A validation workflow for the MaRR reproducibility procedure: technical replicates of expression measurements (e.g., four SEQC Taqman PCR replicates over 1129 transcripts) are averaged, converted to transcript ranks, and used to score the biological relevance of the top-k reproducible sets returned by MaRR versus the copula mixture model.

## When to use

Use when replicate rank lists (e.g., the six SEQC RNA-seq replicate comparisons) must be reduced to per-gene average scores and ranks to test whether a reproducibility method's top-k sets are enriched for gold-standard genes such as the 1129 SEQC Taqman PCR transcripts.

## When NOT to use

- Input is raw sequencing data or un-ranked expression intensities rather than per-transcript replicate measurements that can be averaged and ranked
- There is only a single experiment (no replicate comparisons), since the skill evaluates overlap between replicate-derived top-k sets against PCR ranks
- The analysis goal is parametric modeling of between-replicate dependence; the MaRR framework is explicitly nonparametric and makes no parametric assumptions on dependence structures

## Inputs

- SEQC Taqman PCR measurements: 1129 transcripts measured in four technical replicates
- MaRR top-k reproducible transcript lists for the six SEQC comparisons (k fixed at the number declared reproducible at alpha = 0.01)
- Copula mixture model top-k transcript lists at k matched to the MaRR sets

## Outputs

- Per-transcript PCR ranks derived from four-replicate averages
- Counts of commonly identified PCR genes per comparison (1101, 1099, 851, 1120, 1119, 877)
- Method-specific PCR gene identifiers annotated with PCR ranks
- Two-sample t-test p-values comparing PCR ranks for comparisons 1, 2, 3, and 6
- Summary table pcr_validation_results.csv

## How to apply

Load the SEQC Taqman PCR measurements (1129 transcripts in four technical replicates), average the PCR values over the four replicates with pandas/NumPy, and rank transcripts by the average. Set k per comparison to the number of transcripts MaRR declares reproducible at alpha = 0.01 (38,975, 38,778, 27,020, 40,855, 40,760, 28,798) and take the copula mixture model's top-k sets at those matched k values for the six SEQC comparisons. Intersect each method's top-k set with the 1129 PCR transcripts to record commonly identified PCR gene counts (expected 1101, 1099, 851, 1120, 1119, 877) and the method-specific PCR genes. For comparisons 1, 2, 3, and 6, use SciPy two-sample t-tests to compare PCR-measurement ranks of MaRR-specific versus copula-specific PCR genes (expected p-values 0.0227, 0.0047, 0.2176, 0.1235). Assemble common counts, method-specific gene identifiers with PCR ranks, and t-test p-values into a summary table written to pcr_validation_results.csv.

## Related tools

- **pandas** (Load the SEQC Taqman PCR replicate measurements and top-k transcript sets, average PCR values over the four replicates, and assemble the summary table written to pcr_validation_results.csv)
- **SciPy** (Run two-sample t-tests on PCR ranks of MaRR-specific versus copula-specific PCR genes for comparisons 1, 2, 3, and 6)
- **NumPy** (Numeric ranking of transcripts by averaged PCR values and array manipulation during top-k set intersection)

## Examples

```
pcr_rank = pd.read_csv('seqc_taqman_pcr.csv').groupby('transcript')['pcr'].mean().rank(); ttest_ind(marr_ranks, copula_ranks)
```

## Evaluation signals

- The six common-count rows in pcr_validation_results.csv equal 1101, 1099, 851, 1120, 1119, 877 for comparisons 1-6
- The four t-test p-values match 0.0227, 0.0047, 0.2176, and 0.1235 within rounding tolerance
- Each method's top-k size for comparison j equals the MaRR alpha = 0.01 reproducibility count (38,975, 38,778, 27,020, 40,855, 40,760, 28,798) and the copula model's k is matched to it
- Every transcript in the intersection analysis is one of the 1129 PCR-measured transcripts, and ranks are computed from the four-replicate PCR average
- MaRR identifies more PCR genes than the copula mixture model in all six SEQC comparisons

## Limitations

- The PCR-rank advantage of MaRR-specific genes is not statistically significant in all comparisons: t-test p-values of 0.2176 and 0.1235 for comparisons 3 and 6 fail to reject equal ranks
- Only 1129 transcripts have Taqman PCR measurements, so the intersection analysis is restricted to a small subset of the full SEQC transcript universe
- k is dictated by MaRR's alpha = 0.01 threshold per comparison, so the copula model is evaluated at ranks it did not choose itself

## Evidence
