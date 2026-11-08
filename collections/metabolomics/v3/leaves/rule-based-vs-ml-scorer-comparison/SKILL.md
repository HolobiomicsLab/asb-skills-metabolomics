---
name: rule-based-vs-ml-scorer-comparison
description: Use when evaluating or comparing metabolite annotation scoring methods
  on imaging mass spectrometry datasets where target/decoy ion candidates exist and
  ranking quality (AP/MAP) is the metric of interest.
when_to_use_negative:
- The task is to evaluate FDR calibration or annotation counts at FDR thresholds rather
  than ranking quality — use the paper's separate low-FDR and low-intensity-ion evaluations
  instead
- Datasets lack decoy ion candidates, making target-vs-decoy average precision undefined
- Evaluating a single scorer's absolute performance rather than a head-to-head ML
  vs rule-based comparison
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: METASPACE-ML
  role: Machine-learning scorer whose target/decoy ion rankings are compared against
    MSM
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Annotation engine that produces target and decoy candidates and runs both
    the MSM and ML scoring
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:18c27e234d9852a8f4e0cca06708f722580d1bc03821c3ff4e3004f28163b21d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rule-based-vs-ml-scorer-comparison

## Summary

Benchmark a machine-learning metabolite annotation scorer (METASPACE-ML) against the rule-based Metabolite Signal Matching (MSM) scorer by ranking target-versus-decoy ions per dataset and computing average precision (AP) and mean average precision (MAP). Reproduces the paper's headline comparison showing higher median MAP for the ML model on animal (0.36 vs 0.27) and plant (0.32 vs 0.17) testing datasets.

## When to use

Use when evaluating or comparing metabolite annotation scoring methods on imaging mass spectrometry datasets where target/decoy ion candidates exist and ranking quality (AP/MAP) is the metric of interest.

## When NOT to use

- The task is to evaluate FDR calibration or annotation counts at FDR thresholds rather than ranking quality — use the paper's separate low-FDR and low-intensity-ion evaluations instead
- Datasets lack decoy ion candidates, making target-vs-decoy average precision undefined
- Evaluating a single scorer's absolute performance rather than a head-to-head ML vs rule-based comparison

## Inputs

- List of testing dataset identifiers (Supplementary Data 5; 720 animal, 210 plant)
- Target and decoy ion annotation candidates per dataset with scores from METASPACE-ML
- Target and decoy ion annotation candidates per dataset with scores from rule-based MSM
- Sample-type labels (animal, plant) for grouping the aggregation

## Outputs

- Per-dataset AP values for each method and sample type (map_results.csv)
- Median MAP per method per sample type (expected: METASPACE-ML 0.36 animal / 0.32 plant; MSM 0.27 animal / 0.17 plant)
- Counts of datasets with statistically significant AP scores (expected: 674/720 animal; 207/210 plant)

## How to apply

Load the evaluation set — for this paper, the Supplementary Data 5 list of 720 animal and 210 plant testing datasets. For each dataset, obtain target and decoy annotation candidates scored by both METASPACE-ML and the rule-based MSM scorer using the METASPACE engine (github.com/metaspace2020/metaspace). Per dataset and per method, rank target and decoy ions by each scorer's score and compute average precision; aggregate per-dataset AP into mean average precision within each sample type (animal, plant). Compute median MAP per method per sample type and count datasets where the AP difference is statistically significant. Validation requires reproducing the reported medians (0.36 vs 0.27 animal; 0.32 vs 0.17 plant) and significance counts (674/720 animal; 207/210 plant), with per-dataset AP values written to map_results.csv.

## Related tools

- **METASPACE-ML** (Machine-learning scorer whose target/decoy ion rankings are compared against MSM) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Annotation engine that produces target and decoy candidates and runs both the MSM and ML scoring) — https://github.com/metaspace2020/metaspace

## Examples

```
python compare_map.py --datasets supplementary_data5.csv --methods metaspace_ml,msm --sample-types animal,plant --out map_results.csv
```

## Evaluation signals

- Median MAP for METASPACE-ML vs MSM reproduces 0.36 vs 0.27 on the animal cross-validated dataset group
- Median MAP for METASPACE-ML vs MSM reproduces 0.32 vs 0.17 on the plant cross-validated dataset group
- Number of datasets with significantly higher AP matches 674/720 (animal) and 207/210 (plant)
- map_results.csv contains one per-dataset AP row per method for all 930 testing datasets
- AP values lie in [0, 1] and MAP aggregates are medians over per-dataset AP within each sample type

## Limitations

- MSM captures some annotations missed by METASPACE-ML due to MSM's sensitivity to the rho_spatial feature score, so ranking-quality differences do not imply strict supersets of annotations
- Training and evaluation data are biased towards public datasets represented in METASPACE (from 47 labs); QTOF-based datasets and less common data types (e.g. single-cell metabolomics) are underrepresented
- Decoy ion generation affects the target/decoy separation on which AP is computed; the most reliable decoy generation method requires further investigation

## Evidence

- [abstract] METASPACE-ML outperforms its rule-based predecessor: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity"
- [intro] limitation of the rule-based MSM scoring: "a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability"
- [intro] evaluate at low FDR thresholds and low ion intensity: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds"
- [discussion] MSM captures annotations missed by the ML model: "some annotations were captured by MSM but not by METASPACE-ML due to MSM’s sensitivity to its feature scores, particularly the rho_spatial score"
- [readme] METASPACE hosts the annotation engine: "The METASPACE platform hosts an engine for
 metabolite annotation of imaging mass spectrometry data as well as a
 spatial metabolite knowledgebase"
