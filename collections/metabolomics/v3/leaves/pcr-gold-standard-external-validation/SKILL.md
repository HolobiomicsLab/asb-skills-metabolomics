---
name: pcr-gold-standard-external-validation
description: Use when two reproducibility methods disagree on top-k gene sets for
  SEQC RNA-seq replicate comparisons and an independent qPCR/Taqman gold standard
  is available to judge which method's sets are more biologically relevant.
when_to_use_negative:
- No independent gold-standard measurements (e.g., Taqman PCR) exist for the ranked
  transcripts, so external validation cannot arbitrate between methods and only cross-method
  agreement could be assessed
- The transcript sets being compared are already known to be identical (no method
  disagreement to adjudicate), making the method-specific rank comparison moot
- The data are not SEQC-style replicate rank lists with a matched-k top-k set per
  method, so the k values and alpha = 0.01 thresholds from this workflow do not apply
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: SciPy
  role: Runs the two-sample t-tests comparing PCR-measurement ranks of MaRR-specific
    versus copula-specific PCR genes
- name: pandas
  role: Loads the SEQC PCR measurements and top-k gene lists, computes intersections,
    and writes pcr_validation_results.csv
- name: NumPy
  role: Holds and averages the PCR replicate values and rank arrays used in the intersection
    and t-test steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:e86074afba74c94e32a1ff800e190170e31d21358f706a379aee4a9c86cf30fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pcr-gold-standard-external-validation

## Summary

Externally validates a reproducibility method (e.g., MaRR vs. the copula mixture model) by intersecting each method's declared top-k reproducible transcript sets with an independent Taqman qPCR gold standard from the SEQC project, then comparing PCR-measurement ranks of method-specific genes.

## When to use

Use when two reproducibility methods disagree on top-k gene sets for SEQC RNA-seq replicate comparisons and an independent qPCR/Taqman gold standard is available to judge which method's sets are more biologically relevant.

## When NOT to use

- No independent gold-standard measurements (e.g., Taqman PCR) exist for the ranked transcripts, so external validation cannot arbitrate between methods and only cross-method agreement could be assessed
- The transcript sets being compared are already known to be identical (no method disagreement to adjudicate), making the method-specific rank comparison moot
- The data are not SEQC-style replicate rank lists with a matched-k top-k set per method, so the k values and alpha = 0.01 thresholds from this workflow do not apply

## Inputs

- SEQC Taqman PCR measurements for 1129 transcripts across four technical replicates (per-comparison gene IDs and qPCR values)
- Top-k reproducible transcript sets from the MaRR procedure at alpha = 0.01 for the six SEQC comparisons (k = 38,975, 38,778, 27,020, 40,855, 40,760, 28,798)
- Top-k reproducible transcript sets from the copula mixture model at matched k for the same six SEQC comparisons

## Outputs

- pcr_validation_results.csv containing, per comparison: counts of commonly identified PCR genes, method-specific gene identifiers with their PCR ranks, and two-sample t-test p-values
- Per-comparison PCR-rank summary distinguishing MaRR-specific from copula-specific PCR genes

## How to apply

Load the SEQC Taqman PCR measurements for the 1129 transcripts measured in four technical replicates, average the PCR values over the four replicates, and rank transcripts by that average. For each of the six SEQC comparisons, take the top-k set declared reproducible by MaRR at alpha = 0.01 (k = 38,975, 38,778, 27,020, 40,855, 40,760, 28,798) and the copula mixture model's set at the same matched k, intersecting each with the 1129 PCR transcripts to record common and method-specific PCR genes. For the comparisons where methods diverge (1, 2, 3, and 6), run two-sample t-tests (SciPy) comparing PCR-measurement ranks of MaRR-specific versus copula-specific PCR genes. Assemble common counts, method-specific gene identifiers with their PCR ranks, and t-test p-values into a summary table written to pcr_validation_results.csv. The rationale is that agreement with an orthogonal, higher-accuracy measurement platform (qPCR) arbitrates disagreements that internal reproducibility criteria cannot resolve.

## Related tools

- **SciPy** (Runs the two-sample t-tests comparing PCR-measurement ranks of MaRR-specific versus copula-specific PCR genes)
- **pandas** (Loads the SEQC PCR measurements and top-k gene lists, computes intersections, and writes pcr_validation_results.csv)
- **NumPy** (Holds and averages the PCR replicate values and rank arrays used in the intersection and t-test steps)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(marr_specific_pcr_ranks, copula_specific_pcr_ranks)
```

## Evaluation signals

- The six common-count rows reproduce the reported values 1101, 1099, 851, 1120, 1119, 877 for SEQC comparisons 1-6
- The four t-test p-values match 0.0227, 0.0047, 0.2176, and 0.1235 (comparisons 1, 2, 3, 6) within rounding tolerance
- MaRR top-k set sizes equal the number of transcripts declared reproducible at alpha = 0.01 (38,975, 38,778, 27,020, 40,855, 40,760, 28,798), with copula sets matched to the same k
- Every method-specific gene listed carries a PCR rank derived from the four-replicate averaged PCR measurements, and counts of common + specific PCR genes do not exceed 1129
- pcr_validation_results.csv is produced with the expected summary-table schema (common counts, gene identifiers, ranks, p-values)

## Limitations

- Two of the four t-test comparisons (3 and 6) yielded non-significant p-values (0.2176 and 0.1235), so PCR-rank superiority of MaRR-specific genes is not uniformly significant across comparisons
- Validation is limited to the 1129 transcripts with Taqman PCR measurements, a small subset of the tens of thousands of transcripts in the top-k sets

## Evidence
