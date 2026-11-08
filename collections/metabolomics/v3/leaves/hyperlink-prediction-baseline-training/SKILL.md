---
name: hyperlink-prediction-baseline-training
description: Use when benchmarking a hyperlink prediction model against CHESHIRE on
  metabolite correlation networks, specifically when labeled pathways are sparse and
  a fair reduced-training-set comparison is needed.
when_to_use_negative:
- When the organism or network has abundant labeled pathways and the 'limited known
  pathways' tough condition is irrelevant — the reduced training-split simulation
  would artificially handicap the baseline
- When the goal is to run the NetAurHPD pipeline itself (auralization, M5 training,
  Youden threshold) rather than train and score the CHESHIRE baseline for comparison
- When hyperlinks are not pathway-presence labels over a metabolite correlation network
  (e.g. generic graph link prediction) unless CHESHIRE's input format has been adapted
  accordingly
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: CHESHIRE
  role: State-of-the-art hyperlink prediction baseline trained on the reduced pathway
    training sets (10 sessions, 10%-100% training fraction, 75 epochs, classification
    threshold 0.5)
- name: NetAurHPD
  role: Reference model whose Figure 5 AUC curves and central-pathway counts serve
    as the comparison target for the CHESHIRE baseline
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Deep learning framework in which the model implementations (including the
    auralization-based competitor) are executed
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:bf537cc83955fa6f31a5a9f68e703df1ec39cfae59f3c25a72156cac41b46072
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hyperlink-prediction-baseline-training

## Summary

Trains the state-of-the-art CHESHIRE hyperlink prediction baseline on tomato pericarp Metabolite CNs (2001/2003/2004) under 'tough realistic' limited-known-pathway conditions, then tabulates its AUC and central-pathway predictions against NetAurHPD results from Figure 5.

## When to use

Use when benchmarking a hyperlink prediction model against CHESHIRE on metabolite correlation networks, specifically when labeled pathways are sparse and a fair reduced-training-set comparison is needed.

## When NOT to use

- When the organism or network has abundant labeled pathways and the 'limited known pathways' tough condition is irrelevant — the reduced training-split simulation would artificially handicap the baseline
- When the goal is to run the NetAurHPD pipeline itself (auralization, M5 training, Youden threshold) rather than train and score the CHESHIRE baseline for comparison
- When hyperlinks are not pathway-presence labels over a metabolite correlation network (e.g. generic graph link prediction) unless CHESHIRE's input format has been adapted accordingly

## Inputs

- Tomato pericarp metabolite correlation networks for 2001, 2003, 2004 (75/75/78 nodes; 473/869/338 edges)
- Labeled pathway pool: TomatoCyc positive pathways, MetaCyc negative pathways, random negative pathways
- List of 32 metabolites with minimal pathway involvement (to define the holdout condition)
- Set of 13 critical central TomatoCyc pathways for organism survival

## Outputs

- Per-year, per-training-fraction table of CHESHIRE AUC values
- Counts of correctly classified central pathways per year and training fraction
- Comparison tabulation against NetAurHPD and NetAurHPD-with-one-Augmentation results reproducing Figure 5

## How to apply

Load the 2001, 2003, and 2004 tomato pericarp Metabolite CNs (75/75/78 metabolite nodes with 473/869/338 edges) plus the labeled pathway pool (TomatoCyc positives, MetaCyc negatives, random negatives). For each year, simulate the tough realistic condition: select 32 metabolites with minimal pathway involvement, hold out all pathways containing them, and form a test set of 40 TomatoCyc, 27 MetaCyc, and 13 random pathways, leaving 129 TomatoCyc, 65 MetaCyc, and 64 random pathways for training. Train CHESHIRE on the reduced training sets over 10 sessions per year, growing the training fraction from 10% to 100%, with 75 epochs, the CHESHIRE source-code recommended parameters, and a fixed classification threshold of 0.5. Evaluate per year and training fraction, recording AUC and the count of correctly classified central pathways among the 13 critical TomatoCyc pathways for organism survival. Decision point: CHESHIRE is expected to struggle (low AUC, poor central-pathway predictions) in 2001 and 2003 but be competitive in 2004 — compare against NetAurHPD micro-average AUCs of 0.87 (2001), 0.857 (2003), and 0.884 (2004) and the one-Augmentation variant.

## Related tools

- **CHESHIRE** (State-of-the-art hyperlink prediction baseline trained on the reduced pathway training sets (10 sessions, 10%-100% training fraction, 75 epochs, classification threshold 0.5))
- **NetAurHPD** (Reference model whose Figure 5 AUC curves and central-pathway counts serve as the comparison target for the CHESHIRE baseline) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Deep learning framework in which the model implementations (including the auralization-based competitor) are executed)

## Evaluation signals

- Reproduced CHESHIRE AUC curves must match the left panel of Figure 5 for 2001, 2003, and 2004 across the 10 training-fraction sessions
- Reproduced central-pathway prediction counts must match the right panel of Figure 5 (13 critical TomatoCyc pathways as the maximum)
- CHESHIRE should yield lower general AUC than NetAurHPD across all three years, and more incorrectly classified central pathways in 2001 and 2003, with competitive results only in 2004
- Test-set composition check: exactly 40 TomatoCyc + 27 MetaCyc + 13 random pathways held out per year, with 129/65/64 pathways remaining for training
- NetAurHPD reference micro-average AUCs (0.87, 0.857, 0.884 for 2001/2003/2004) provide sanity bounds for the comparison table

## Limitations

- The models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so per-session variability must be expected
- CHESHIRE struggles to reach high AUC and yields poor central-pathway predictions in 2001 and 2003, meaning the baseline comparison is year-dependent and not uniformly decisive
- The baseline uses a fixed classification threshold of 0.5, unlike NetAurHPD's Youden-index-optimized threshold, which should be kept in mind when interpreting metric differences

## Evidence
