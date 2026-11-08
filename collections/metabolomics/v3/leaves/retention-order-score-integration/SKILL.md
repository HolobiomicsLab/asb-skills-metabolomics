---
name: retention-order-score-integration
description: Use when a full LC-MS/MS run yields one scored candidate list per spectrum
  (e.g. from IOKR over PubChem) and a RankSVM retention-order predictor is available,
  and per-spectrum top hits must be made jointly consistent in elution order.
when_to_use_negative:
- Only a single MS/MS spectrum is being analyzed with no retention-time context —
  per-spectrum IOKR scores alone are the appropriate baseline
- The chromatographic system differs so much that no RankSVM retention-order predictor
  can be trained for it (e.g. no related LC system data); the integration penalty
  term would be meaningless
- Retention times of the spectra are unavailable or unreliable, since layers must
  be sorted by increasing measured retention time and edge penalties depend on observed
  RT ordering
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IOKR
  role: Supplies the per-spectrum scored candidate lists (MS/MS-based molecular scores)
    that form the graph layers
- name: networkx
  role: Python library (>= 2.0) used in the order-prediction/integration code for
    building the layered graph and running the shortest-path dynamic programming
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: numpy
  role: Python package (>= 1.13.1) dependency of the order predictor and dynamic programming
    integration code
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: pandas
  role: Python package (>= 0.20.3) dependency for loading and handling the per-spectrum
    candidate tables
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdkTools
  role: Computes MACCS counting fingerprints (via CDK) used as the molecular feature
    map φ for the RankSVM penalty term
  repo: https://github.com/bachi55/rcdkTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:9910d7d38d99751f6338ef9d42b2fe9929d9554f43032db66af84f00375fab2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python and
  - json >=
  - networkx
  - networkx >=
  - numpy
  - pandas
  - rcdkTools
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# retention-order-score-integration

## Summary

Jointly identifies metabolites across a complete LC-MS/MS run by combining IOKR MS/MS-based candidate scores with predicted retention orders via a shortest-path dynamic program over a layered graph, improving identifications over MS/MS scores alone.

## When to use

Use when a full LC-MS/MS run yields one scored candidate list per spectrum (e.g. from IOKR over PubChem) and a RankSVM retention-order predictor is available, and per-spectrum top hits must be made jointly consistent in elution order.

## When NOT to use

- Only a single MS/MS spectrum is being analyzed with no retention-time context — per-spectrum IOKR scores alone are the appropriate baseline
- The chromatographic system differs so much that no RankSVM retention-order predictor can be trained for it (e.g. no related LC system data); the integration penalty term would be meaningless
- Retention times of the spectra are unavailable or unreliable, since layers must be sorted by increasing measured retention time and edge penalties depend on observed RT ordering

## Inputs

- Per-spectrum scored candidate lists from IOKR (candidates queried from PubChem by precursor mass or predicted molecular formula)
- Trained RankSVM weight vector w with the molecular feature map φ (e.g. MACCS counting fingerprints)
- Measured MS/MS spectra with retention times from a complete LC-MS/MS run (e.g. from MassBank, matched against GNPS)

## Outputs

- A sequence of candidate molecular structures, one per spectrum, consistent with the predicted retention order
- Minimum-total-weight path through the layered candidate graph with accumulated weights per node

## How to apply

Sort the measured MS/MS spectra by increasing retention time and build a layered directed graph with one layer per spectrum; each layer contains that spectrum's IOKR candidate molecules (queried from PubChem by mass or predicted formula) ordered by decreasing IOKR score y_i,j. Connect every node n_i,j to every node n_r,s in the next layer and weight each edge d = −y_r,s + Δ·max(0, w^T(φ(m_i,j) − φ(m_r,s))), where w is the RankSVM weight vector, φ the molecular feature map (e.g. MACCS counting fingerprints), and the penalty term is set to zero when the observed retention times of layers i and r are equal. Run a forward dynamic programming pass (Bellman, 1957) layer by layer, storing for each node the minimum-weight path from the first layer and its accumulated weight; time complexity is O(NK²) with N layers and K the maximum layer size. Select the node with minimum total weight in the last layer and backtrack through stored predecessors to output the candidate molecular structures in retention order. The Δ scaling constant trades off MS/MS score magnitude against retention-order violations and its effect should be inspected (the paper reports identification accuracy as a function of Δ).

## Related tools

- **IOKR** (Supplies the per-spectrum scored candidate lists (MS/MS-based molecular scores) that form the graph layers)
- **networkx** (Python library (>= 2.0) used in the order-prediction/integration code for building the layered graph and running the shortest-path dynamic programming) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **numpy** (Python package (>= 1.13.1) dependency of the order predictor and dynamic programming integration code) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **pandas** (Python package (>= 0.20.3) dependency for loading and handling the per-spectrum candidate tables) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdkTools** (Computes MACCS counting fingerprints (via CDK) used as the molecular feature map φ for the RankSVM penalty term) — https://github.com/bachi55/rcdkTools

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- The recovered path is the minimum-total-weight path connecting the first layer to the last layer of the graph
- The joint assignment yields more correctly identified molecular structures than IOKR scores alone on a complete LC-MS/MS run
- Each graph layer contains exactly the IOKR candidate set for one MS/MS spectrum, with candidates ordered by decreasing IOKR score y_i,j and layers sorted by increasing retention time
- The edge penalty term max(0, w^T(φ(m_i,j) − φ(m_r,s))) evaluates to zero for every edge between layers whose observed retention times are equal
- Runtime scales as O(NK²), where N is the number of layers and K is the maximum layer size

## Limitations

- The dynamic programming integration requires retention times for all measured spectra; the penalty is only zeroed out for equal observed retention times, so missing or noisy RT undermines the ordering
- The effect of the parameter Δ (scaling of the retention-order penalty) on identification accuracy is only referenced via a figure and no single optimal value is prescribed in the text
- Benefits depend on the upstream RankSVM order predictor; SVR-based retention time prediction was limited in its capacity to use data from other chromatographic systems, unlike RankSVM

## Evidence

- [methods] Dynamic programming edge-weight definition: [evidence span withheld — anchor status: not_found]
