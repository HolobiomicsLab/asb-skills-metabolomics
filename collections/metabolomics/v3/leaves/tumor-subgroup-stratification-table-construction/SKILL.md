---
name: tumor-subgroup-stratification-table-construction
description: Use when a tumor cohort must be stratified into molecular subgroups (e.g.
  by biomarker-high/low status such as phosphoAKT1 and MYC) and a table of subgroup
  counts, percentages, and the marker scoring parameters that produced them is required.
when_to_use_negative:
- Input is already a stratification table or feature table with subgroup labels assigned
  — the k-means and counting step is already done.
- Markers of interest are not phosphoAKT1/MYC-like continuous expression values and
  cannot be scored with the article's IHC/immunoblot cutoffs and scoring procedure.
- The goal is differential expression or pathway enrichment analysis rather than discrete
  subgroup stratification and count/percentage reporting.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3360
tools:
- name: GraphPad Prism
  role: Used to compute the Fisher Test statistic for differential segregation of
    phosphoAKT1-low/MYC-high versus phosphoAKT1-high/MYC-low subgroups at p<0.05 significance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:90ef478936c4e435988cb50742bd9b1a1e9b4709a1ba3ff138bdc6161b2c5b17
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tumor-subgroup-stratification-table-construction

## Summary

Constructs a reproducible stratification table for a tumor cohort by clustering biomarker expression (e.g. phosphoAKT1/MYC densitometric values via k-means) into defined subgroups and reporting counts and percentages per subgroup, with the marker scoring parameters, cohort composition, and significance testing documented alongside.

## When to use

Use when a tumor cohort must be stratified into molecular subgroups (e.g. by biomarker-high/low status such as phosphoAKT1 and MYC) and a table of subgroup counts, percentages, and the marker scoring parameters that produced them is required.

## When NOT to use

- Input is already a stratification table or feature table with subgroup labels assigned — the k-means and counting step is already done.
- Markers of interest are not phosphoAKT1/MYC-like continuous expression values and cannot be scored with the article's IHC/immunoblot cutoffs and scoring procedure.
- The goal is differential expression or pathway enrichment analysis rather than discrete subgroup stratification and count/percentage reporting.

## Inputs

- Densitometric values of phosphorylated AKT1 and MYC per tumor (from immunoblotting of human tumors)
- Immunohistochemistry scoring parameters for markers scored (e.g. MYC, stathmin, GLUT-1, phosphoAKT1) with 50% positive-cell cutoff
- Cohort composition metadata: 60 pathological stage T2 tumors, 22 high Gleason (4+3/4+4) and 38 low Gleason (3+3/3+4)
- Formalin-fixed paraffin-embedded human prostate tumor tissue samples

## Outputs

- stratification_table.tsv with columns for marker/scoring parameter or subgroup, count, percentage, and source statement
- Four phosphoAKT1/MYC subgroup assignments per tumor (high/high, high/low, low/high, low/low) with counts 6/60 (10%), 21/60 (35%), 9/60 (15%), 24/60 (40%)
- Fisher Test statistic and significance call (GraphPad Prism 5.0, p<0.05) for differential subgroup segregation

## How to apply

Score each tumor for the markers of interest using the article's IHC/immunoblot procedure — phosphoAKT1 and MYC densitometric values obtained by immunoblotting, with IHC scoring performed over the entire section in 5 representative areas at 20x, averaging 500 cells per section. Apply k-means clustering to the phosphoAKT1/MYC densitometric values to segregate the 60 stage T2 prostate tumors (22 high Gleason 4+3/4+4, 38 low Gleason 3+3/3+4) into 4 subgroups: phosphoAKT1-high/MYC-high (6/60, 10%), phosphoAKT1-high/MYC-low (21/60, 35%), phosphoAKT1-low/MYC-high (9/60, 15%), and phosphoAKT1-low/MYC-low (24/60, 40%). Test whether the stratification reflects the molecular biology by evaluating the differential segregation of subgroups with a Fisher Test (GraphPad Prism 5.0, p<0.05), reporting that phosphoAKT1-low/MYC-high and phosphoAKT1-high/MYC-low tumors differentially segregated (p<0.01). Assemble the marker/scoring parameters (markers scored, the 50% positive-cell cutoff, significance threshold) and the four subgroup counts/percentages into a table with columns for marker/scoring parameter or subgroup, count, percentage, and the verbatim source statement, writing the result to stratification_table.tsv. Verify every extracted value matches the verbatim statements in the article text before finalizing.

## Related tools

- **GraphPad Prism** (Used to compute the Fisher Test statistic for differential segregation of phosphoAKT1-low/MYC-high versus phosphoAKT1-high/MYC-low subgroups at p<0.05 significance)

## Examples

```
K-means clustering of phosphoAKT1/MYC densitometric values was conducted to segregate 4 prostate tumor subgroups (Fig. 1C); Fisher Test via GraphPad Prism 5.0 with p<0.05 significance; output stratification_table.tsv
```

## Evaluation signals

- Subgroup counts sum to the cohort size (6+21+9+24 = 60) and each percentage equals count/60 (10%, 35%, 15%, 40%).
- Every row in stratification_table.tsv carries a verbatim source statement that matches the article text exactly.
- Fisher Test result reported as p<0.01 for differential segregation of phosphoAKT1-low/MYC-high and phosphoAKT1-high/MYC-low subgroups.
- Table includes the marker/scoring parameters (markers scored, 50% positive-cell cutoff, GraphPad Prism 5.0, p<0.05) alongside the subgroup counts.
- Cohort composition columns (22 high Gleason 4+3/4+4, 38 low Gleason 3+3/3+4) are present and consistent with the verbatim statements.

## Limitations

- The k-means stratification is tied to phosphoAKT1/MYC densitometric values and the specific 60-tumor cohort; it does not transfer directly to other marker pairs or cohorts without re-clustering.
- PhosphoAKT1 and MYC levels were not associated with Gleason grade, so the stratification is independent of Gleason and pathologic staging and should not be conflated with grade-based stratification.
- Silhouette analysis indicated human tumors fell in 3 clusters, meaning the 4-subgroup k-means partition and the 3-cluster Silhouette partition differ; both should be reported to avoid over-interpreting one clustering result.

## Evidence

- [methods] IHC scoring parameters: markers scored with 50% positive-cell cutoff and Fisher Test via GraphPad Prism 5.0: [evidence span withheld — anchor status: not_found]
- [results] Cohort composition: 60 pathological stage T2 tumors (22 high Gleason, 38 low Gleason): [evidence span withheld — anchor status: not_found]
