---
name: ranked-gene-list-overlap-analysis
description: Use when two reproducibility-assessment methods have each declared top-k
  reproducible transcripts from replicate rank lists and an external gold-standard
  subset (e.g., 1129 Taqman PCR transcripts) is available to judge which method's
  method-specific genes are more biologically relevant.
when_to_use_negative:
- The experiment lacks an external gold-standard subset (e.g., Taqman PCR measurements)
  for only a fraction of transcripts — without it, method-specific genes cannot be
  ranked by PCR rank and the t-test comparison is undefined
- Only one method's reproducible set is available — this skill is designed for pairwise
  comparison at matched k, so a single top-k set cannot be validated relatively
- The goal is the reproducibility discovery itself (ranked-list reproducibility assessment
  with FDR control) rather than post-hoc external validation of already-declared top-k
  sets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SciPy
  role: Two-sample t-tests comparing PCR-measurement ranks of MaRR-specific versus
    copula-specific PCR genes for comparisons 1, 2, 3, and 6
- name: pandas
  role: Loading SEQC PCR and top-k set tables and assembling/writing the summary table
    pcr_validation_results.csv
- name: NumPy
  role: Ranking transcripts by the PCR replicate averages and handling numeric arrays
    during set intersection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:c6ba64d250850744d6abb1fedadd8e35eaf4750cd2f3c5c835d5a7611384cb29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - SciPy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ranked-gene-list-overlap-analysis

## Summary

External validation of reproducibility methods (MaRR vs a copula mixture model) by intersecting each method's top-k reproducible transcript sets with a Taqman PCR gold-standard subset and comparing PCR-measurement ranks of method-specific genes via two-sample t-tests.

## When to use

Use when two reproducibility-assessment methods have each declared top-k reproducible transcripts from replicate rank lists and an external gold-standard subset (e.g., 1129 Taqman PCR transcripts) is available to judge which method's method-specific genes are more biologically relevant.

## When NOT to use

- The experiment lacks an external gold-standard subset (e.g., Taqman PCR measurements) for only a fraction of transcripts — without it, method-specific genes cannot be ranked by PCR rank and the t-test comparison is undefined
- Only one method's reproducible set is available — this skill is designed for pairwise comparison at matched k, so a single top-k set cannot be validated relatively
- The goal is the reproducibility discovery itself (ranked-list reproducibility assessment with FDR control) rather than post-hoc external validation of already-declared top-k sets

## Inputs

- SEQC Taqman PCR measurements for 1129 transcripts across four technical replicates (tabular, e.g., CSV)
- Top-k reproducible transcript sets from the MaRR procedure at alpha = 0.01 (k = 38,975, 38,778, 27,020, 40,855, 40,760, 28,798 for comparisons 1-6)
- Top-k reproducible transcript sets from the copula mixture model at matched k for the six SEQC comparisons
- Transcript rank lists from replicate RNA-seq experiments used by the reproducibility methods

## Outputs

- Summary table written to pcr_validation_results.csv with common PCR gene counts (per comparison), method-specific gene identifiers with PCR ranks, and t-test p-values
- Common-count rows: 1101, 1099, 851, 1120, 1119, 877 for SEQC comparisons 1-6
- Two-sample t-test p-values for PCR rank comparisons: 0.0227, 0.0047, 0.2176, 0.1235 (comparisons 1, 2, 3, 6)

## How to apply

Load the gold-standard measurements (1129 SEQC Taqman PCR transcripts in four technical replicates), average PCR values over the four replicates, and rank transcripts by the average. Set k for each method to the number of transcripts declared reproducible at alpha = 0.01 (e.g., 38,975, 38,778, 27,020, 40,855, 40,760, 28,798 for the six SEQC comparisons), matching k across methods so overlap counts are comparable. For each method and comparison, intersect the top-k set with the 1129 PCR transcripts and record commonly identified PCR genes plus method-specific (MaRR-only and copula-only) PCR gene identifiers with their PCR ranks. Where a method contributes enough unique genes (comparisons 1, 2, 3, 6 here), run two-sample t-tests on PCR ranks of MaRR-specific versus copula-specific genes using SciPy. The rationale is that MaRR is nonparametric and makes no parametric assumptions on dependence structures, so its reproducible sets can be validated against an orthogonal quantitative standard without model-based assumptions.

## Related tools

- **SciPy** (Two-sample t-tests comparing PCR-measurement ranks of MaRR-specific versus copula-specific PCR genes for comparisons 1, 2, 3, and 6)
- **pandas** (Loading SEQC PCR and top-k set tables and assembling/writing the summary table pcr_validation_results.csv)
- **NumPy** (Ranking transcripts by the PCR replicate averages and handling numeric arrays during set intersection)

## Examples

```
import pandas as pd; from scipy import stats; t, p = stats.ttest_ind(marr_pcr_ranks, copula_pcr_ranks)
```

## Evaluation signals

- The six common-count rows reproduce 1101, 1099, 851, 1120, 1119, 877 for SEQC comparisons 1-6
- The four t-test p-values match 0.0227, 0.0047, 0.2176, and 0.1235 within rounding tolerance
- k values are matched between MaRR and the copula mixture model within each comparison and equal the MaRR counts declared at alpha = 0.01 (38,975, 38,778, 27,020, 40,855, 40,760, 28,798)
- PCR ranks are computed from transcript averages over the four technical replicates, not from a single replicate
- The output file pcr_validation_results.csv exists and contains common counts, method-specific gene identifiers with PCR ranks, and t-test p-values

## Limitations

- The t-test on PCR ranks is only reported where enough method-specific PCR genes exist — comparisons 4 and 5 yield no p-values in the source analysis
- Comparison 3 (p = 0.2176) and comparison 6 (p = 0.1235) do not show a significant PCR-rank advantage for MaRR-specific genes, so the validation is not uniformly significant across all six comparisons
- The gold standard covers only 1129 of the transcripts, so overlap counts depend on this subset rather than the full transcript universe

## Evidence

- [methods] PCR replicate averaging workflow step: [evidence span withheld — anchor status: not_found]
- [methods] alpha threshold for reproducible set sizes: [evidence span withheld — anchor status: not_found]
