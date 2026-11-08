---
name: candidate-structure-graph-construction-from-identification-candidates
description: Use when a ready-to-identify dataset pairs candidate structures with
  MS/MS spectral scores and ROASMI retention-order probabilities, and you must rank
  structures for each peak.
when_to_use_negative:
- The goal is to identify peaks in HILIC or other non-RPLC chromatographic systems
  — ROASMI currently applies only to RPLC systems, so retention-order edge probabilities
  are unavailable without retraining.
- Candidates include compounds outside the elemental restrictions of C, H, N, O, P,
  and S — ROASMI's applicability domain does not cover such structures.
- Only MS/MS spectral scores are available and no retention-order probabilities have
  been predicted — the graph edges cannot be populated.
- Candidate pairs are already known to be indistinguishable by retention (high ROASMI
  variance) and the analysis does not require ranking — building the graph adds no
  information.
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: ROASMI
  role: predicts retention-order probabilities P_ij that become the directed edge
    weights of the fully connected candidate graph
  repo: https://github.com/FangYuan717/ROASMI
- name: msms_rt_score_integration
  role: consumes the constructed graph (candidate structures as nodes with MS/MS scores,
    retention-order probabilities as edges) via random spanning-tree ensembles to
    compute max-marginal top-k rankings
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- name: chemprop
  role: provides the D-MPNN molecular embedding module underlying ROASMI's candidate
    structure representations used to derive retention-order probabilities
  repo: https://github.com/chemprop/chemprop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:54ad03c8ea7a1bff1b9e5e26243d2c6ebb6125418fcfbe60d1df5d6925c7874c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - ROASMI
  - chemprop
  - msms_rt_score_integration
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# candidate-structure-graph-construction-from-identification-candidates

## Summary

Construct a fully connected graph over candidate molecular structures for each detected peak, with nodes carrying MS/MS scores and directed edges carrying ROASMI-predicted retention-order probabilities P_ij, as the prerequisite for probabilistic max-marginal top-k ranking.

## When to use

Use when a ready-to-identify dataset pairs candidate structures with MS/MS spectral scores and ROASMI retention-order probabilities, and you must rank structures for each peak.

## When NOT to use

- The goal is to identify peaks in HILIC or other non-RPLC chromatographic systems — ROASMI currently applies only to RPLC systems, so retention-order edge probabilities are unavailable without retraining.
- Candidates include compounds outside the elemental restrictions of C, H, N, O, P, and S — ROASMI's applicability domain does not cover such structures.
- Only MS/MS spectral scores are available and no retention-order probabilities have been predicted — the graph edges cannot be populated.
- Candidate pairs are already known to be indistinguishable by retention (high ROASMI variance) and the analysis does not require ranking — building the graph adds no information.

## Inputs

- ready-to-identify dataset CSV (e.g. identify_toy_set.csv) listing candidate structures per detected peak with MS/MS spectral scores
- ROASMI-predicted retention-order probabilities P_ij for each pair of candidate compounds (from ROASMI_predict.py output)

## Outputs

- fully connected candidate graph per peak (nodes = candidate structures with MS/MS scores; directed edges = retention-order probabilities P_ij)
- input graph consumed by the msms_rt_score_integration max-marginal top-k ranking step

## How to apply

Load the ready-to-identify dataset (e.g. 'data/identify_toy_set.csv') that contains candidate structures with MS/MS spectral scores per detected peak, plus the ROASMI-predicted retention-order probabilities P_ij for each pair of candidates. For each detected peak, create a node for every candidate structure, assigning its MS/MS score, and add a directed edge between every pair of candidates carrying the ROASMI P_ij probability that one candidate elutes before the other. The resulting fully connected graph over candidates is the input representation expected by the Eric Bach et al. probabilistic integration framework (aalto-ics-kepaco/msms_rt_score_integration), where random spanning-tree ensembles over this graph infer the maximum probability of ranking the top-k candidates, with the full candidate vector z as node labels. Candidate pairs that ROASMI flags as indistinguishable (high prediction variance, e.g. close retention pairs like 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) should be excluded or down-weighted before ranking. Note that ROASMI's P_ij applies to the RPLC system with eluent pH of around 2.7, so graphs built for other chromatographic spaces are out of scope without retraining.

## Related tools

- **ROASMI** (predicts retention-order probabilities P_ij that become the directed edge weights of the fully connected candidate graph) — https://github.com/FangYuan717/ROASMI
- **msms_rt_score_integration** (consumes the constructed graph (candidate structures as nodes with MS/MS scores, retention-order probabilities as edges) via random spanning-tree ensembles to compute max-marginal top-k rankings) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration
- **chemprop** (provides the D-MPNN molecular embedding module underlying ROASMI's candidate structure representations used to derive retention-order probabilities) — https://github.com/chemprop/chemprop

## Examples

```
python code/ROASMI_predict.py --pred_path data/predict_toy_set.csv ; python code/Identify.py
```

## Evaluation signals

- For every detected peak in the input ready-to-identify dataset, the constructed graph contains exactly one node per candidate structure listed in the CSV.
- For every pair of candidate structures within a peak, a directed edge exists carrying the ROASMI P_ij retention-order probability — no pair is missing.
- Node MS/MS scores match the spectral scores given in the input CSV for the corresponding candidate/peak.
- Candidate pairs marked indistinguishable by ROASMI (e.g. high prediction variance such as the 81% probability / 9% variance example) are excluded or flagged from the final ranked annotation output.
- The resulting graph is accepted by Identify.py in the msms_rt_score_integration workflow (script runs to completion and writes ranked annotations for every peak).

## Limitations

- ROASMI's retention-order probabilities apply only to the reversed-phase liquid chromatography system with an eluent pH of around 2.7; graphs built for other chromatographic spaces are unsupported.
- ROASMI applies to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, leaving a vast chemical space yet to be explored.
- When the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments, making some P_ij edges unreliable (indistinguishable pairs).
- Addressing RT outliers resulting from slight changes in experimental conditions requires in-depth mechanistic studies, so edge probabilities can be noisy for such cases.

## Evidence

- [methods] retention-order edge probabilities from ROASMI predict step: "To predict the retention scores of each compound in the ready-to-identify datasets, run: python code/ROASMI_predict.py"
- [methods] retention scores are integrated with MS/MS scores for ranking: "we integrate predicted retention scores with MS/MS scores by running: python code/Identify.py"
- [discussion] exclusion of indistinguishable pairs before identification: "Such pairs should be excluded from the subsequent small molecule identification process."
- [discussion] ROASMI predicts an ordered pair with probability and variance: "ROASMI predicts that peak 1 elutes before peak 2 with an 81% probability and a variance of 9%."
- [readme] ROASMI repository ships trained RPLC models: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
- [methods] input data location for the identify step: "can be referenced as 'identify_toy_set.csv' in the 'data' folder"
- [methods] RankNet learns elution orders to populate edge probabilities: "A Ranking Neural Network (RankNet) was used to learn elution orders"
