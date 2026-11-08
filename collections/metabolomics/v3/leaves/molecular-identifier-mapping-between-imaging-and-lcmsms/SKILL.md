---
name: molecular-identifier-mapping-between-imaging-and-lcmsms
description: Use when imaging-MS metabolite annotations (formula+adduct identifiers
  at a stated FDR, e.g. <10%) need orthogonal validation against MS/MS-confirmed LC-MS/MS
  ground truth to estimate true positive rates.
when_to_use_negative:
- The ground-truth identifications are not MS/MS-validated (e.g., only accurate-mass
  matches) — the TPR estimate would not represent MS/MS validation
- Identifiers cannot be expressed as comparable formula+adduct pairs (e.g., imaging
  annotations lacking adduct information or LC-MS/MS IDs reported only as retention-time
  features)
- The goal is to validate individual pixel-level spatial localizations rather than
  molecule-level annotation identity
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: METASPACE-ML
  role: Machine-learning annotation engine whose formula+adduct imaging annotations
    are validated against LC-MS/MS identifications; also supplies METASPACE annotation
    power (MAP) scores
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform hosting the rule-based MSM annotations and the annotations database
    that the comparison sets are drawn from
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:9328a9126626c14c201c459bf1f8706395182a9650b5486cb932890f3b72c598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Molecular identifier mapping between imaging and LC-MS/MS annotations

## Summary

Validates MALDI imaging mass spectrometry metabolite annotations (formula+adduct pairs) by set-based intersection against MS/MS-validated bulk LC-MS/MS identifications, computing true positive rates for competing annotation approaches (e.g., METASPACE-ML vs rule-based MSM).

## When to use

Use when imaging-MS metabolite annotations (formula+adduct identifiers at a stated FDR, e.g. <10%) need orthogonal validation against MS/MS-confirmed LC-MS/MS ground truth to estimate true positive rates.

## When NOT to use

- The ground-truth identifications are not MS/MS-validated (e.g., only accurate-mass matches) — the TPR estimate would not represent MS/MS validation
- Identifiers cannot be expressed as comparable formula+adduct pairs (e.g., imaging annotations lacking adduct information or LC-MS/MS IDs reported only as retention-time features)
- The goal is to validate individual pixel-level spatial localizations rather than molecule-level annotation identity

## Inputs

- Imaging-MS annotation table of formula+adduct pairs annotated at FDR < 10% (Supplementary Data 8), one set per approach (METASPACE-ML, MSM) and per dataset
- MS/MS-validated bulk LC-MS/MS metabolite identifications serving as ground truth (Supplementary Data 9)
- Optional: per-annotation METASPACE annotation power (MAP) scores for median computation across datasets

## Outputs

- Boolean MS/MS-validated true-positive label per formula+adduct annotation per approach
- Overall and exclusive-ion true positive rates (TPR) per annotation approach
- Median METASPACE annotation power (MAP) per approach across the 10 datasets
- validation_metrics.csv summarizing the validation metrics

## How to apply

Load the imaging annotations as formula+adduct pairs (e.g., Supplementary Data 8, annotated at FDR < 10% by METASPACE-ML and MSM across the 10 mouse brain MALDI-FTICR imaging datasets) and the reference LC-MS/MS identifications (Supplementary Data 9). Represent each molecular identification as a normalized identifier string combining neutral formula and adduct so that imaging and LC-MS/MS entries are directly comparable, then use pandas set-based intersection to label each imaging annotation as an MS/MS-validated true positive or not. Partition results per annotation approach, computing the overall TPR and the TPR restricted to ions annotated exclusively by one approach — decision points include whether to compare overall vs exclusive-ion subsets. In the reference study this reproduced an overall TPR of 91% for METASPACE-ML, and exclusive-ion TPRs of 72% (METASPACE-ML) vs 11% (MSM), alongside median METASPACE annotation power (MAP) of 0.57 vs 0.43.

## Related tools

- **METASPACE-ML** (Machine-learning annotation engine whose formula+adduct imaging annotations are validated against LC-MS/MS identifications; also supplies METASPACE annotation power (MAP) scores) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform hosting the rule-based MSM annotations and the annotations database that the comparison sets are drawn from) — https://github.com/metaspace2020/metaspace

## Examples

```
import pandas as pd; tp_ml = set(ann_ml['formula_adduct']) & set(lcmsms['formula_adduct']); tpr_ml = ann_ml['formula_adduct'].isin(tp_ml).mean()
```

## Evaluation signals

- Identifier normalization check: every imaging annotation and LC-MS/MS ID resolves to a formula+adduct string with no unmatched formatting variants after parsing
- Set-inclusion invariant: the exclusive-ion TPR subsets partition annotations cleanly — union of ML-only, MSM-only, and shared sets equals the total annotation set
- Reproduce expected metric ranges from the reference study: overall TPR ≈ 91% (METASPACE-ML), exclusive-ion TPR 72% vs 11% (ML vs MSM), median MAP 0.57 vs 0.43
- Output schema check: validation_metrics.csv contains per-approach overall TPR, exclusive-ion TPR, and median MAP columns for all 10 datasets
- Sanity check that no LC-MS/MS ground-truth identifier appears as a validated TP in an approach's annotation set where that identifier is absent

## Limitations

- Some annotations are captured by MSM but not METASPACE-ML because MSM is sensitive to its feature scores, particularly the rho_spatial score, so neither approach is a strict superset and exclusive-ion TPRs must be reported separately
- MS/MS-validated ground truth covers only ions with reference LC-MS/MS identifications; annotations without LC-MS/MS counterparts cannot be classified and are excluded from TPR computation
- The validation is specific to the mouse brain MALDI-FTICR context (10 datasets) and FDR < 10% threshold; transferability to other instruments or FDR cutoffs is not established

## Evidence

- [results] In the LC-MS/MS bulk validation, METASPACE-ML achieves a 91% overall true positive rate, and for ions exclusively captured by either approach, a 72% vs 11% TPR for METASPACE-ML vs the rule-based approach.: [evidence span withheld — anchor status: not_found]
- [abstract] METASPACE-ML outperforms its rule-based predecessor with higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
- [discussion] MSM's sensitivity to rho_spatial causes exclusive MSM annotations: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [readme] The python-client provides programmatic access to the annotations database: [evidence span withheld — anchor status: external_doc]
