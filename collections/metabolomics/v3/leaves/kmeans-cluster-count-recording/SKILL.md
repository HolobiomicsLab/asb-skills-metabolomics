---
name: kmeans-cluster-count-recording
description: Use when a tumor cohort has been immunoblotted for two markers (e.g.
  phosphoAKT1 and MYC) and needs to be stratified into high/low subgroups via k-means
  clustering of densitometric values, with subgroup counts and percentages recorded
  for downstream comparison or reporting.
when_to_use_negative:
- The tumor samples have already been assigned to subgroups by a different marker
  set or an independent classifier — k-means re-clustering would overstate agreement.
- The cohort size is too small for meaningful k-means clustering (e.g. fewer tumors
  than the requested k), or densitometric values are not available and only IHC-positive/negative
  calls exist — a simpler 2x2 contingency analysis is more appropriate.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: GraphPad Prism
  role: Used to compute the Fisher test statistic (p<0.05 threshold) when validating
    the significance of cluster assignments across subgroups, and to produce the stratification
    table.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:e2018d79f98d3e3e79713d69d0a897f7d469d50c131eda2b2bc1048b5ecf65c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# kmeans-cluster-count-recording

## Summary

A skill for segmenting a small tumor cohort by two immunoblot markers (phosphoAKT1 and MYC) using k-means clustering on densitometric values, then recording the exact tumor count and percentage assigned to each resulting subgroup — e.g. the 60-tumor prostate cohort split into phosphoAKT1-high/MYC-high (6/60, 10%), phosphoAKT1-high/MYC-low (21/60, 35%), phosphoAKT1-low/MYC-high (9/60, 15%), and phosphoAKT1-low/MYC-low (24/60, 40%).

## When to use

Use when a tumor cohort has been immunoblotted for two markers (e.g. phosphoAKT1 and MYC) and needs to be stratified into high/low subgroups via k-means clustering of densitometric values, with subgroup counts and percentages recorded for downstream comparison or reporting.

## When NOT to use

- The tumor samples have already been assigned to subgroups by a different marker set or an independent classifier — k-means re-clustering would overstate agreement.
- The cohort size is too small for meaningful k-means clustering (e.g. fewer tumors than the requested k), or densitometric values are not available and only IHC-positive/negative calls exist — a simpler 2x2 contingency analysis is more appropriate.

## Inputs

- Densitometric values of phosphoAKT1 and MYC for each tumor sample (e.g. from immunoblot quantification)
- Total cohort size (e.g. 60 tumors) and subgroup count/percentage pairings

## Outputs

- K-means cluster assignments per tumor for the two-marker (phosphoAKT1/MYC) subgroups
- A count-and-percentage record per subgroup (e.g. 6/60=10%, 21/60=35%, 9/60=15%, 24/60=40%)
- Optional cross-tabulation of cluster assignments vs. covariates (e.g. Gleason grade) with Fisher-test significance (p<0.05)

## How to apply

First perform immunoblotting of the tumor samples for the markers of interest (phosphoAKT1 and MYC) and extract densitometric values for each tumor. Then run k-means clustering on the two-marker densitometric matrix to segregate the cohort into subgroups (e.g. 4 subgroups for a 60-tumor cohort). Record for each resulting subgroup the number of tumors (e.g. 6, 21, 9, 24) and the percentage of the total (e.g. 10%, 35%, 15%, 40%), using GraphPad Prism or equivalent to verify significance (e.g. Fisher test, p<0.05) when comparing cluster assignments across covariates. The count/percentage record should include the marker cutoff rationale (e.g. 50% positive-cell threshold for IHC scoring) and the k-means parameters (k=4) so the stratification is reproducible.

## Related tools

- **GraphPad Prism** (Used to compute the Fisher test statistic (p<0.05 threshold) when validating the significance of cluster assignments across subgroups, and to produce the stratification table.)

## Examples

```
R: km <- kmeans(cbind(phosphoAKT1_dens, MYC_dens), centers=4); table(km$cluster); fisher.test(table(km$cluster, gleason))  # GraphPad Prism 5.0 equivalent
```

## Evaluation signals

- The sum of recorded subgroup counts equals the total cohort size (e.g. 6+21+9+24 = 60) and the sum of percentages equals 100% (10+35+15+40).
- Each recorded count/percentage pair matches a verbatim k-means cluster assignment from the article (e.g. phosphoAKT1-high/MYC-high = 6/60 = 10%).
- The k value used matches the number of distinct subgroups reported (k=4 for the 60-tumor phosphoAKT1/MYC cohort).
- Any significance test (e.g. Fisher test) is run on the correct contingency table (cluster assignment vs. covariate) and the reported p-value falls below the stated threshold (p<0.05, or p<0.01 where the article applies a stricter cutoff).

## Limitations

- The article notes that phosphoAKT1 and MYC levels were not associated with Gleason grade, so cluster assignments should not be interpreted as grade-linked without further validation.
- The 60-tumor cohort was pathologic stage T2, so counts/percentages are specific to that stage and should not be extrapolated to advanced-stage tumors without re-stratification.
- Silhouette analysis indicated that human tumors fell into 3 clusters rather than 4, so the 4-subgroup partition from k-means should be treated as an operational grouping rather than an intrinsic number of clusters.
- The Fisher test on cluster assignment vs. covariates reached p<0.01 only for the phosphoAKT1-low/MYC-high vs. phosphoAKT1-high/MYC-low segregation, so other pairwise comparisons may lack statistical support at that threshold.

## Evidence

- [methods] IHC scoring threshold: [evidence span withheld — anchor status: not_found]
