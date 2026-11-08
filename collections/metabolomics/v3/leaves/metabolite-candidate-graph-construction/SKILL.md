---
name: metabolite-candidate-graph-construction
description: Use when analyzing a complete LC-MS/MS run with per-spectrum IOKR candidate
  lists (candidates from PubChem by mass or predicted formula) plus a trained RankSVM
  retention-order model, and you must jointly identify the set of metabolites.
when_to_use_negative:
- When identifying a single MS/MS spectrum in isolation — there are no retention-order
  relationships between layers, so the layered graph and shortest-path machinery collapse
  to plain IOKR ranking
- When you need actual retention time values on a common scale — use the SVR retention
  time predictor instead, since SVR was limited in its capacity to use data from other
  chromatographic systems, unlike RankSVM
- When no RankSVM retention-order model (weight vector w and feature map φ) has been
  trained for the target LC system — the edge penalty term cannot be computed
tools:
- name: IOKR
  role: Supplies the MS/MS-based scored candidate lists that populate each graph layer
    (candidates queried from PubChem)
- name: networkx
  role: Python library (>= 2.0) used to build and traverse the layered directed candidate
    graph
- name: numpy
  role: Numerical computation (>= 1.13.1) for score combination and feature map inner
    products w^T(φ(m_i,j) − φ(m_r,s))
- name: retention_order_prediction
  role: 'Main implementation repository: RankSVM order predictor and the dynamic programming
    module that integrates the two predictors into a joint identification'
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:59d3a05a7bac0e4134d0ec0bd6984962bbdc4b0719c01e35ef9a938dd49d3fc5
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
  - retention_order_prediction
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# metabolite-candidate-graph-construction

## Summary

Builds a layered directed graph for joint metabolite identification across a complete LC-MS/MS run: each layer holds the IOKR-scored candidate structures for one MS/MS spectrum (sorted by increasing retention time), and edges combine IOKR MS/MS scores with RankSVM retention-order penalties so a shortest-path (dynamic programming) solution yields the joint assignment.

## When to use

Use when analyzing a complete LC-MS/MS run with per-spectrum IOKR candidate lists (candidates from PubChem by mass or predicted formula) plus a trained RankSVM retention-order model, and you must jointly identify the set of metabolites.

## When NOT to use

- When identifying a single MS/MS spectrum in isolation — there are no retention-order relationships between layers, so the layered graph and shortest-path machinery collapse to plain IOKR ranking
- When you need actual retention time values on a common scale — use the SVR retention time predictor instead, since SVR was limited in its capacity to use data from other chromatographic systems, unlike RankSVM
- When no RankSVM retention-order model (weight vector w and feature map φ) has been trained for the target LC system — the edge penalty term cannot be computed

## Inputs

- Per-spectrum IOKR scored candidate lists (candidate molecular structures queried from PubChem by mass or predicted molecular formula, each with IOKR score y_i,j)
- Trained RankSVM weight vector w and molecular feature map φ (e.g. MACCS counting fingerprints)
- Measured retention times of the MS/MS spectra from the LC-MS/MS run
- Penalty scaling parameter Δ

## Outputs

- Layered directed graph (one layer per MS/MS spectrum, candidates as nodes, fully connected consecutive layers with combined edge weights)
- Optimal path through the layers giving the joint metabolite assignment — a sequence of candidate molecular structures in retention order
- Minimum accumulated path weight per node

## How to apply

Load the per-spectrum scored candidate lists output by IOKR and the RankSVM weight vector w with its molecular feature map φ (e.g. MACCS counting fingerprints under a minmax kernel). Sort the layers into increasing retention time of the measured MS/MS spectra — one layer per spectrum — and sort candidates within each layer by decreasing IOKR score y_i,j. Connect every node n_i,j to every node n_r,s in the next layer (r = i+1) and assign each edge the weight d = −y_r,s + Δ·max(0, w^T(φ(m_i,j) − φ(m_r,s))), setting the penalty term to zero when the observed retention times of layers i and r are equal (Δ scales the retention-order contribution). Build the graph programmatically with networkx (>= 2.0) in the Python 3.5/3.6 environment, then run the forward dynamic programming pass layer by layer, storing for each node the minimum-weight path from the first layer; select the minimum-total-weight node in the last layer and backtrack through stored predecessors to output the candidate structures in retention order. The rationale is that retention order is much better conserved between instruments than retention time, so order-based penalties transfer across heterogeneous LC systems without mapping retention times to a common scale.

## Related tools

- **IOKR** (Supplies the MS/MS-based scored candidate lists that populate each graph layer (candidates queried from PubChem))
- **networkx** (Python library (>= 2.0) used to build and traverse the layered directed candidate graph)
- **numpy** (Numerical computation (>= 1.13.1) for score combination and feature map inner products w^T(φ(m_i,j) − φ(m_r,s)))
- **retention_order_prediction** (Main implementation repository: RankSVM order predictor and the dynamic programming module that integrates the two predictors into a joint identification) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm met_ident_perf_GS_BS 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- The recovered path is the minimum-total-weight path connecting the first to the last layer (Bellman optimality check)
- The joint assignment yields more correctly identified molecular structures than IOKR MS/MS scores alone
- Layers are ordered by increasing measured retention time of the MS/MS spectra and candidates within each layer are sorted by decreasing IOKR score
- Edge penalty term max(0, w^T(φ(m_i,j) − φ(m_r,s))) evaluates to exactly zero when observed retention times of adjacent layers are equal
- Runtime scales as O(NK²), where N is the number of layers (spectra) and K is the maximum layer size

## Limitations

- Time complexity is O(NK²) with K the maximum candidate layer size, so large IOKR candidate lists per spectrum make the graph expensive to solve
- The effect of the penalty parameter Δ is only referenced via a figure; no quantitative accuracy values for different Δ settings are reported in the discussion
- No link/repository for the IOKR implementation is given in the article (only the Brouard et al. 2016 citation), so the scored candidate lists must be produced externally

## Evidence

- [methods] complexity bound: [evidence span withheld — anchor status: not_found]
