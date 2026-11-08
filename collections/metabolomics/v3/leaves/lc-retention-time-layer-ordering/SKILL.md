---
name: lc-retention-time-layer-ordering
description: Use when a complete LC-MS/MS run yields multiple MS/MS spectra with associated
  retention times and scored candidate lists from an MS/MS-based predictor like IOKR,
  and the goal is to jointly identify a set of metabolites by combining MS/MS-based
  scores with predicted retention order.
when_to_use_negative:
- Only a single MS/MS spectrum is available (no set of spectra to layer); the graph
  degenerates and the joint identification via shortest path is not applicable
- Retention times are not measured or not available for the MS/MS spectra, so layers
  cannot be ordered by increasing retention time
- The IOKR scored candidate lists or the RankSVM weight vector are missing — the edge-weight
  formula requires both the MS/MS-based score y and the retention-order predictor
  w
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3366
tools:
- name: IOKR
  role: Produces the scored candidate lists (MS/MS-based scores) whose candidates
    become the nodes of each layer in the ordered graph
- name: networkx
  role: Python library used to build and traverse the layered directed graph for the
    dynamic programming shortest-path pass
  repo: https://networkx.org/
- name: rcdkTools
  role: Computes the MACCS counting fingerprints that instantiate the molecular feature
    map φ used in the pairwise retention-order penalty term
  repo: https://github.com/bachi55/rcdkTools
- name: retention_order_prediction
  role: Implements the RankSVM order predictor and the dynamic programming module
    to integrate the two kinds of predictors into a joint identification for a set
    of metabolites
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:83ecf05380c6d081426cf02a004ee7305e111794b18baa05a398a3d78f2ed2d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python and
  - json >=
  - networkx
  - networkx >=
  - rcdkTools
  - retention_order_prediction
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# LC retention time layer ordering

## Summary

Constructs a layered shortest-path graph for joint metabolite identification by ordering each MS/MS spectrum layer by increasing retention time and sorting candidate nodes within each layer by decreasing IOKR score.

## When to use

Use when a complete LC-MS/MS run yields multiple MS/MS spectra with associated retention times and scored candidate lists from an MS/MS-based predictor like IOKR, and the goal is to jointly identify a set of metabolites by combining MS/MS-based scores with predicted retention order.

## When NOT to use

- Only a single MS/MS spectrum is available (no set of spectra to layer); the graph degenerates and the joint identification via shortest path is not applicable
- Retention times are not measured or not available for the MS/MS spectra, so layers cannot be ordered by increasing retention time
- The IOKR scored candidate lists or the RankSVM weight vector are missing — the edge-weight formula requires both the MS/MS-based score y and the retention-order predictor w

## Inputs

- Per-spectrum IOKR scored candidate lists (candidates queried from PubChem by mass or predicted molecular formula)
- Measured retention time for each MS/MS spectrum in the LC-MS/MS run
- RankSVM weight vector w trained on retention order from heterogeneous LC systems (e.g. PredRet datasets: Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old)
- Molecular feature map φ (e.g. MACCS counting fingerprints computed via rcdkTools) for each candidate molecule

## Outputs

- A layered directed acyclic graph where layer i corresponds to the i-th MS/MS spectrum in retention-time order and node n_i,j is the j-th IOKR candidate for that spectrum
- Edge-weighted connections between consecutive layers ready for a dynamic programming (shortest path) forward pass
- The optimal sequence of candidate molecular structures in retention order, retrieved by backtracking through stored predecessors

## How to apply

Take the per-spectrum IOKR scored candidate lists (candidates queried from PubChem by mass or predicted molecular formula) and the measured retention time for each MS/MS spectrum. Sort the layers into increasing retention time of the measured MS/MS spectra, creating one graph layer per spectrum. Within each layer, sort the candidate nodes by decreasing IOKR score y_i,j so that edge weights can be computed between consecutive-layer candidates. Connect every node n_i,j in layer i to every node n_r,s in the next layer r=i+1 and assign each edge the weight d = −y_r,s + Δ·max(0, w^T(φ(m_i,j) − φ(m_r,s))), setting the penalty term to zero when the observed retention times of layers i and r are equal. The retention-order RankSVM weight vector w and the molecular feature map φ (e.g. MACCS counting fingerprints computed via rcdkTools) supply the pairwise penalty. Run the forward dynamic programming (shortest path) pass layer by layer over this ordered graph to recover the sequence of candidate molecular structures in retention order; the ordering is what makes the joint identification tractable as a shortest-path problem.

## Related tools

- **IOKR** (Produces the scored candidate lists (MS/MS-based scores) whose candidates become the nodes of each layer in the ordered graph)
- **networkx** (Python library used to build and traverse the layered directed graph for the dynamic programming shortest-path pass) — https://networkx.org/
- **rcdkTools** (Computes the MACCS counting fingerprints that instantiate the molecular feature map φ used in the pairwise retention-order penalty term) — https://github.com/bachi55/rcdkTools
- **retention_order_prediction** (Implements the RankSVM order predictor and the dynamic programming module to integrate the two kinds of predictors into a joint identification for a set of metabolites) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- The recovered path connects the first layer to the last layer and is the minimum-total-weight path among all such paths, as verified by exhaustive comparison or a known shortest-path solver
- Layers are strictly non-decreasing in measured retention time and, within each layer, nodes are sorted by decreasing IOKR score y_i,j
- The joint assignment yields more correctly identified molecular structures than IOKR scores alone when evaluated on the complete LC-MS/MS run
- Time complexity of the dynamic programming pass scales as O(NK²), where N is the number of layers and K is the maximum layer size, so runtime should grow quadratically with the largest candidate layer size
- The penalty term Δ·max(0, w^T(φ(m_i,j) − φ(m_r,s))) is set to zero whenever the observed retention times of layers i and r are equal

## Limitations

- The method relies on the assumption that retention order is conserved between instruments; while retention order is much better conserved than retention time, residual instrument-specific variation remains
- Retention time measurements arising from heterogeneous LC systems and configurations are pooled for training the RankSVM, which may introduce noise if a target system is under-represented
- Edge weights are computed pairwise between all candidates in consecutive layers, so very large candidate layers inflate the O(NK²) dynamic programming cost
- Molecules with very small retention times (non-interacting with the chromatographic system) or with large retention time spread (>5%) are removed during preprocessing, so such structures are excluded from the graph

## Evidence

- [methods] Edge weight formula: [evidence span withheld — anchor status: not_found]
- [methods] Penalty zero when retention times are equal: [evidence span withheld — anchor status: not_found]
- [methods] Time complexity: [evidence span withheld — anchor status: not_found]
