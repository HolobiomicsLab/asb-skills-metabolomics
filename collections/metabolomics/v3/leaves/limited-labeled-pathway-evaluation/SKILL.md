---
name: limited-labeled-pathway-evaluation
description: Use when comparing a pathway/hyperlink prediction model against a baseline
  (e.g. CHESHIRE) under artificially limited labeled-pathway conditions on metabolite
  correlation networks, to mimic organisms with sparse pathway knowledge.
when_to_use_negative:
- When labeled pathways are abundant and the goal is standard model selection — the
  paper's main evaluation uses 10-fold cross-validation instead of the limited-label
  holdout
- When the aim is threshold tuning or score computation for NetAurHPD itself (that
  is handled by Youden's J statistic in the NetAurHPD pipeline, not by the CHESHIRE
  baseline protocol)
- When the input network is not a metabolite correlation network (e.g. the Enron or
  NDC hyperlink datasets used in the NetAurHPD repository examples), since the pathway
  labels and central-pathway metrics are tomato-specific
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: CHESHIRE
  role: State-of-the-art hyperlink prediction baseline trained on the reduced pathway
    training sets for comparison
- name: NetAurHPD
  role: Reference model whose Figure 5 results (AUC and central-pathway counts) the
    reproduced CHESHIRE results are validated against
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation framework used to train and evaluate the comparison models
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:3fa5b75da25fcfc934dd2c871de7cc20f6fe740ea2ce64157996b91d8453381c
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - CHESHIRE
  - NetAurHPD
  - PyTorch
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# limited-labeled-pathway-evaluation

## Summary

A benchmarking protocol that stresses hyperlink/pathway prediction models by holding out pathways around minimally-annotated metabolites, then compares a baseline (CHESHIRE) against NetAurHPD per year and per training fraction on tomato pericarp metabolite correlation networks.

## When to use

Use when comparing a pathway/hyperlink prediction model against a state-of-the-art baseline (e.g. CHESHIRE) under artificially limited labeled-pathway conditions on metabolite correlation networks, to mimic organisms with sparse pathway knowledge.

## When NOT to use

- When labeled pathways are abundant and the goal is standard model selection — the paper's main evaluation uses 10-fold cross-validation instead of the limited-label holdout
- When the aim is threshold tuning or score computation for NetAurHPD itself (that is handled by Youden's J statistic in the NetAurHPD pipeline, not by the CHESHIRE baseline protocol)
- When the input network is not a metabolite correlation network (e.g. the Enron or NDC hyperlink datasets used in the NetAurHPD repository examples), since the pathway labels and central-pathway metrics are tomato-specific

## Inputs

- Tomato pericarp metabolite correlation networks for 2001, 2003, and 2004 (75/75/78 metabolite nodes; 473/869/338 edges)
- Labeled pathway sets: TomatoCyc positive pathways, MetaCyc non-plant negative pathways, and sampled random negative pathways
- List of 13 central TomatoCyc pathways identified as critical for organism survival
- CHESHIRE source code and its recommended hyperparameter settings

## Outputs

- Per-year CHESHIRE AUC curves as a function of training fraction (10% to 100%, 10 sessions)
- Count of correctly classified central (survival-critical) pathways per year
- Comparison table of CHESHIRE vs. NetAurHPD and NetAurHPD-with-one-Augmentation for validation against Figure 5

## How to apply

Load the 2001/2003/2004 tomato pericarp metabolite correlation networks (75/75/78 metabolite nodes; 473/869/338 edges) plus the labeled pathway pool (TomatoCyc positive, MetaCyc negative, random negative). To construct the tough realistic condition per year, select 32 metabolites with minimal pathway involvement, hold out every pathway containing them, and form a test set of 40 TomatoCyc, 27 MetaCyc, and 13 random pathways, leaving 129 TomatoCyc, 65 MetaCyc, and 64 random pathways for training. Train CHESHIRE on the reduced training sets across 10 sessions growing the training fraction from 10% to 100%, with 75 epochs and classification threshold 0.5 (other parameters per CHESHIRE source-code recommendations). Record AUC and the count of correctly classified central pathways (the 13 TomatoCyc pathways critical for organism survival) per year and fraction, then tabulate against the NetAurHPD and NetAurHPD-with-one-Augmentation results (Figure 5) to expose where CHESHIRE struggles (2001, 2003) versus competes (2004).

## Related tools

- **CHESHIRE** (State-of-the-art hyperlink prediction baseline trained on the reduced pathway training sets for comparison)
- **NetAurHPD** (Reference model whose Figure 5 results (AUC and central-pathway counts) the reproduced CHESHIRE results are validated against) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation framework used to train and evaluate the comparison models)

## Examples

```
from NetAurHPD.result_optimization_and_analysis import optimize_results_and_calc_metrics; optimize_results_and_calc_metrics(y_test, y_pred)
```

## Evaluation signals

- Reproduced CHESHIRE AUC curves match the left panels of Figure 5 for 2001, 2003, and 2004
- Central-pathway prediction counts match the right panels of Figure 5, with CHESHIRE yielding poor central-pathway predictions in 2001 and 2003 and competitive results in 2004
- CHESHIRE AUC stays below NetAurHPD's general AUC across all three years (NetAurHPD micro-average AUC references: 0.87 in 2001, 0.857 in 2003, 0.884 in 2004)
- Test-set composition is exactly 40 TomatoCyc, 27 MetaCyc, and 13 random pathways per year, with training pools of 129/65/64
- Each training fraction is evaluated over 10 sessions, and AUC behavior is checked against the known non-monotonic pattern where models may decrease with more data

## Limitations

- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so single-fraction results can be misleading
- CHESHIRE is only competitive in the 2004 dataset; conclusions about the baseline should not be generalized to all harvesting seasons
- NetAurHPD has a low level of explainability and cannot give biologists a full understanding of its results, which limits the interpretability of the comparison

## Evidence
