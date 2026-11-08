---
name: f-beta-reliability-scoring-of-annotations
description: Use when per-dataset annotation results (target and decoy hits) are available
  at multiple default FDR thresholds and you need to rank thresholds by a precision-weighted
  reliability metric or identify the optimal FDR for a sample-type group.
when_to_use_negative:
- Input is raw imaging MS data (e.g. imzML) not yet annotated by METASPACE-ML — annotation
  must exist before reliability can be scored
- Only a single FDR threshold is available, making optimal-threshold selection meaningless
- The comparison target is the MSM rule-based scoring system rather than threshold
  reliability of METASPACE-ML results
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE-ML
  role: Provides the machine-learning annotation results whose target/decoy outputs
    are scored by the F-beta reliability metric
- name: METASPACE
  role: Source platform of the knowledge base datasets and target/decoy annotation
    results; accessed via its python-client
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:145e454e60dd04f94befa6210e1c0388553034a71a9f15d27e96a768a407c0ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# f-beta-reliability-scoring-of-annotations

## Summary

Quantifies the reliability of METASPACE-ML metabolite annotation results by computing an F-beta score (beta = 0.5, precision-weighted) of target-decoy-derived precision/recall at each default FDR threshold per dataset, then selecting the FDR threshold that maximizes this reliability score.

## When to use

Use when per-dataset annotation results (target and decoy hits) are available at multiple default FDR thresholds and you need to rank thresholds by a precision-weighted reliability metric or identify the optimal FDR for a sample-type group.

## When NOT to use

- Input is raw imaging MS data (e.g. imzML) not yet annotated by METASPACE-ML — annotation must exist before reliability can be scored
- Only a single FDR threshold is available, making optimal-threshold selection meaningless
- The comparison target is the MSM rule-based scoring system rather than threshold reliability of METASPACE-ML results

## Inputs

- Per-dataset target annotation results from the METASPACE knowledge base
- Per-dataset decoy annotation results for target-decoy FDR estimation
- List of the four default FDR thresholds (including 5%)
- Sample-type group labels (e.g. animal testing datasets) for aggregation

## Outputs

- reliability_scores.csv with per-dataset F-beta scores, optimal threshold, and sample-type group
- Fig. 5C/D-style PNG plots: reliability-score distribution and optimal-FDR breakdown
- Aggregates: fraction of datasets with optimal FDR at 5% and median reliability score

## How to apply

Load per-dataset target and decoy annotation results from the METASPACE knowledge base (metaspace2020/metaspace repository, documented in its GitHub Wiki). For each dataset, compute precision and recall of confident metabolite annotations at each of the four default FDR thresholds using the target-decoy estimates. Compute the F-beta score with beta = 0.5 — a precision-weighted harmonic mean that penalizes false positives more than misses — at every threshold to obtain the reliability_score per dataset per threshold. Select, per dataset, the FDR threshold that maximizes the F-beta score, then aggregate over sample-type groups (e.g. animal datasets): the fraction with optimal FDR at 5% and the median reliability score. Write reliability_scores.csv and render Fig. 5C/D-style distribution and optimal-FDR-breakdown plots, flagging the run as failed if aggregates deviate substantially from the reported ~70% at 5% FDR and median reliability score of 0.88.

## Related tools

- **METASPACE-ML** (Provides the machine-learning annotation results whose target/decoy outputs are scored by the F-beta reliability metric)
- **METASPACE** (Source platform of the knowledge base datasets and target/decoy annotation results; accessed via its python-client) — https://github.com/metaspace2020/metaspace

## Examples

```
python scripts/reliability_scores.py --input annotations.csv --fdr 0.05,0.1,0.2,0.5 --beta 0.5 --out reliability_scores.csv
```

## Evaluation signals

- ~70% of animal testing datasets should have an optimal FDR threshold at 5%
- Median reliability score across animal datasets should be approximately 0.88
- reliability_scores.csv must contain per-dataset F-beta scores, the selected optimal threshold, and the sample-type group for every dataset
- The optimal threshold per dataset must equal the argmax of that dataset's F-beta scores across the four default FDR thresholds
- Flag the run as failed if either aggregate deviates substantially from the reported 70% / 0.88 values (Fig. 5C, D)

## Limitations

- Target-decoy FDR estimation depends on decoy generation quality; decoy ions show heterogeneity in similarity to target ions and a fully reliable decoy generation method needs further investigation
- Reliability estimates are biased towards public datasets represented in METASPACE (from 47 labs); less common data types, e.g. single-cell metabolomics from cultured cells or industrial lab data, are underrepresented
- QTOF-based datasets are underrepresented in the model due to lack of publicly available data needed for statistically reliable evaluation

## Evidence

- [abstract] F-beta reliability scoring with target-decoy FDR: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation."
- [abstract] Knowledge base provenance of scored datasets: "derived from the METASPACE knowledge base"
- [readme] Repo and docs for data access: [evidence span withheld — anchor status: external_doc]
- [discussion] Decoy generation caveat affecting FDR reliability: "Further investigations are needed into how this affects FDR estimation and in finding the most reliable way of decoy generation"
- [discussion] Dataset-selection bias limiting generalization of reliability aggregates: "this selection is biased towards public datasets represented in METASPACE"
