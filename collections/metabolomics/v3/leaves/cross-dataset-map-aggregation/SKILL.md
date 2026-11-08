---
name: cross-dataset-map-aggregation
description: Use when comparing two annotation scoring methods across a corpus of
  imaging mass spectrometry datasets and you need a single ranking-quality metric
  (MAP) per method per sample type, plus counts of datasets with statistically significant
  AP differences.
when_to_use_negative:
- Evaluating a single dataset — AP/MAP aggregation is designed for cross-dataset comparison,
  not one-off annotation quality checks
- Comparing annotation databases (e.g. CoreMetabolome vs general databases) using
  precision-recall at fixed FDR rather than target-vs-decoy ranking metrics
- Input data lacks decoy ion annotations, making target-vs-decoy AP computation impossible
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: GBDT machine-learning scorer whose target/decoy ion rankings are aggregated
    into MAP for benchmarking
- name: METASPACE
  role: Annotation engine used to obtain target and decoy candidates and scores for
    both methods on each testing dataset
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:ae443f9b2bb2b5958b967dd0ccd4974a06f6f3fd485f072b5a2596224c60d530
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-dataset-map-aggregation

## Summary

Aggregates per-dataset average precision (AP) of target-vs-decoy ion rankings into mean average precision (MAP) across many testing datasets, enabling benchmarking of METASPACE-ML against the rule-based MSM scorer on animal and plant imaging MS datasets.

## When to use

Use when comparing two annotation scoring methods across a corpus of imaging mass spectrometry datasets and you need a single ranking-quality metric (MAP) per method per sample type, plus counts of datasets with statistically significant AP differences.

## When NOT to use

- Evaluating a single dataset — AP/MAP aggregation is designed for cross-dataset comparison, not one-off annotation quality checks
- Comparing annotation databases (e.g. CoreMetabolome vs general databases) using precision-recall at fixed FDR rather than target-vs-decoy ranking metrics
- Input data lacks decoy ion annotations, making target-vs-decoy AP computation impossible

## Inputs

- Supplementary Data 5 list of testing datasets (720 animal, 210 plant)
- Per-dataset target and decoy ion annotation candidates with scores from METASPACE-ML
- Per-dataset target and decoy ion annotation candidates with scores from the rule-based MSM scorer
- METASPACE knowledge base datasets accessible via the METASPACE engine

## Outputs

- Per-dataset AP values for each method (map_results.csv)
- Median MAP per method per sample type (animal: 0.36 vs 0.27; plant: 0.32 vs 0.17)
- Counts of datasets with statistically significant AP differences (674/720 animal; 207/210 plant)

## How to apply

Load the evaluation set (Supplementary Data 5: 720 animal and 210 plant testing datasets). For each dataset, obtain target and decoy annotation candidates scored by both METASPACE-ML (GBDT ranking model) and the rule-based MSM scorer via the METASPACE engine. Rank target and decoy ions per dataset by each method's score, compute average precision (AP) per dataset per method, then aggregate AP values into MAP within each sample type. Compute the median MAP for each method (expected: 0.36 vs 0.27 animal; 0.32 vs 0.17 plant) and count datasets with statistically significant AP differences (expected: 674/720 animal, 207/210 plant). Rationale: MAP over target-vs-decoy rankings captures how well a scorer separates true from decoy annotations across heterogeneous datasets, complementing FDR-threshold comparisons; the GBDT model's robustness to noisy outliers supports its use for target/decoy separation.

## Related tools

- **METASPACE-ML** (GBDT machine-learning scorer whose target/decoy ion rankings are aggregated into MAP for benchmarking)
- **METASPACE** (Annotation engine used to obtain target and decoy candidates and scores for both methods on each testing dataset) — https://github.com/metaspace2020/metaspace

## Examples

```
python compute_map.py --dataset-list SupplementaryData5.csv --scorers metaspace_ml,msm --output map_results.csv
```

## Evaluation signals

- Computed median MAP reproduces reported values: 0.36 (METASPACE-ML) vs 0.27 (MSM) on animal and 0.32 vs 0.17 on plant cross-validated datasets
- Significant-AP dataset counts match 674/720 (animal) and 207/210 (plant) testing datasets
- map_results.csv contains one AP row per dataset per method covering all 930 testing datasets (720 animal + 210 plant)
- Per-dataset AP values fall within [0, 1] and are higher for METASPACE-ML in the majority of datasets

## Limitations

- MSM remains sensitive to its feature scores, particularly the rho_spatial score, so some annotations are captured by MSM but missed by METASPACE-ML
- Target-decoy separation is driven mainly by rho_spectral, with rho_chaos6 having only minor impact, which demands re-defining the quantification of spatial informativeness
- Decoy ions show heterogeneity in similarity to target ions; reliable decoy generation needs further investigation and affects FDR estimation
- QTOF-based datasets are underrepresented due to lack of publicly available data needed for statistically reliable model evaluation

## Evidence

- [intro] We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity.: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [discussion] GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [discussion] CoreMetabolome, which resulted in higher MAP scores compared to more general databases: "CoreMetabolome, which resulted in higher MAP scores compared to more general databases"
- [readme] Contains a daemon that runs the metabolite annotation engine, and a REST API for sending jobs to the daemon: [evidence span withheld — anchor status: external_doc]
