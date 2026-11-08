---
name: approach-exclusive-ion-sensitivity-analysis
description: Use when comparing two FDR-controlled annotation approaches on imaging
  MS datasets against an independent LC-MS/MS validation set, and you need TPRs split
  by ions shared vs exclusively captured by each approach.
when_to_use_negative:
- When no independent MS/MS-validated LC-MS/MS identification list exists to serve
  as ground truth — TPR cannot be computed
- When annotations are not comparable as formula+adduct identifier pairs (e.g. different
  FDR thresholds or incomparable metabolite databases across the two approaches)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE-ML
  role: Machine-learning annotation approach whose formula+adduct annotations are
    validated against LC-MS/MS identifications and compared to MSM
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform hosting the rule-based MSM annotations and the annotations database
    queried for the comparison
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:60143d795ae2954e9007bc2d960f8c814f92fd1feb74073ad00fca9e7588cdc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Approach-exclusive ion sensitivity analysis

## Summary

Benchmarks two metabolite annotation approaches (METASPACE-ML vs rule-based MSM) by intersecting their formula+adduct annotations against MS/MS-validated bulk LC-MS/MS identifications, reporting overall and exclusive-ion true positive rates to reveal which approach uniquely captures validated ions.

## When to use

Use when comparing two FDR-controlled annotation approaches on imaging MS datasets against an independent LC-MS/MS validation set, and you need TPRs split by ions shared vs exclusively captured by each approach.

## When NOT to use

- When no independent MS/MS-validated LC-MS/MS identification list exists to serve as ground truth — TPR cannot be computed
- When annotations are not comparable as formula+adduct identifier pairs (e.g. different FDR thresholds or incomparable metabolite databases across the two approaches)

## Inputs

- Supplementary Data 8: formula+adduct annotations at FDR < 10% from METASPACE-ML and MSM for 10 mouse brain MALDI-FTICR imaging datasets
- Supplementary Data 9: MS/MS-validated bulk LC-MS/MS metabolite identifications (reference ground truth)

## Outputs

- Per-approach metric table (validation_metrics.csv) with median MAP, overall TPR, and exclusive-ion TPR for METASPACE-ML and MSM
- Labels of each imaging annotation as MS/MS-validated true positive or not

## How to apply

Load the per-approach annotation table (e.g. Supplementary Data 8: formula+adduct pairs annotated at FDR < 10% by METASPACE-ML and MSM across the 10 mouse brain MALDI-FTICR imaging datasets) and the ground-truth table of MS/MS-validated bulk LC-MS/MS identifications (Supplementary Data 9). Match imaging annotations to the LC-MS/MS identifications as set intersections of (formula, adduct) identifiers in pandas to label MS/MS-validated true positives. Compute per-approach median METASPACE annotation power (MAP) across the 10 datasets. Then compute the overall TPR per approach and, critically, the TPR restricted to ions annotated exclusively by one approach (set difference A−B for each side). Reproduced reference values: median MAP 0.57 (METASPACE-ML) vs 0.43 (MSM), overall TPR 91% (METASPACE-ML), and exclusive-ion TPR 72% vs 11% (METASPACE-ML vs rule-based MSM).

## Related tools

- **METASPACE-ML** (Machine-learning annotation approach whose formula+adduct annotations are validated against LC-MS/MS identifications and compared to MSM) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform hosting the rule-based MSM annotations and the annotations database queried for the comparison) — https://github.com/metaspace2020/metaspace

## Examples

```
ml=set(df8[df8.approach=='METASPACE-ML'].formula_adduct); msm=set(df8[df8.approach=='MSM'].formula_adduct); tpr_excl=(ml-msm).isin(set(df9.id)).mean()
```

## Evaluation signals

- Overall METASPACE-ML TPR against LC-MS/MS bulk identifications should be approximately 91%
- Exclusive-ion TPR should be ~72% for METASPACE-ML vs ~11% for the rule-based approach
- Per-approach median MAP across the 10 datasets should be ~0.57 (METASPACE-ML) vs ~0.43 (MSM)
- All annotations must be matched at FDR < 10% and compared as exact formula+adduct identifier pairs via set intersection
- Output table must contain one row per approach with MAP, overall TPR, and exclusive-ion TPR columns

## Limitations

- Some annotations were captured by MSM but not by METASPACE-ML due to MSM's sensitivity to its feature scores, particularly the rho_spatial score — exclusive-ion analysis must count both directions
- MSM's sensitivity to the rho_spatial score means exclusive-ion TPR asymmetries may reflect feature-score behavior rather than annotation quality alone

## Evidence

- [results] METASPACE-ML achieves a 91% overall true positive rate, and for ions exclusively captured by either approach, a 72% vs 11% TPR for METASPACE-ML vs the rule-based approach: [evidence span withheld — anchor status: not_found]
- [discussion] MSM sensitivity to rho_spatial: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [abstract] METASPACE-ML outperforms rule-based predecessor: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity"
