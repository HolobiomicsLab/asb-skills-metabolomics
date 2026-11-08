---
name: dynamic-programming-shortest-path-optimization
description: Use when a complete LC-MS/MS run has per-spectrum scored candidate lists
  (e.g. from IOKR, queried from PubChem) plus a trained retention-order model, and
  you need a joint metabolite assignment consistent with predicted elution order.
when_to_use_negative:
- Identifying a single MS/MS spectrum in isolation — the graph formulation requires
  multiple spectra from a complete LC-MS/MS run ordered by retention time
- No retention-order predictor output (RankSVM weight vector w and feature map phi)
  is available — then only the D=0 IOKR-only baseline applies and the shortest-path
  machinery adds nothing
- Observed retention times of adjacent layers are equal — the hinge penalty is zero
  there, so the path degenerates to IOKR score comparison for those transitions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: IOKR
  role: Produces the per-spectrum scored candidate lists that form the graph layers
    (input-output kernel regression with spectral input kernel and molecular output
    kernel)
- name: networkx
  role: Represents the layered directed graph over candidate nodes for the shortest-path
    dynamic programming computation
- name: Python 3.5 and 3.6
  role: Hosts the dynamic programming module integrating the two predictors into a
    joint identification for a set of metabolites
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: numpy
  role: Computes the hinge penalty term D * max(0, w^T(phi(m_i,j) - phi(m_r,s))) over
    fingerprint feature maps
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:e8c4e729b6349327564119db4d31abaf7713681211280585d20e41501160a61f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python 3.5 and 3.6
  - Python and
  - R
  - joblib
  - json >=
  - networkx
  - networkx >=
  - numpy
  - pandas
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# dynamic-programming-shortest-path-optimization

## Summary

Joint metabolite identification across a complete LC-MS/MS run by building a layered directed graph over IOKR candidate lists (one layer per MS/MS spectrum, ordered by increasing retention time) and recovering the minimum-weight path with a Bellman-style dynamic programming pass.

## When to use

Use when a complete LC-MS/MS run has per-spectrum scored candidate lists (e.g. from IOKR, queried from PubChem) plus a trained retention-order model, and you need a joint metabolite assignment consistent with predicted elution order.

## When NOT to use

- Identifying a single MS/MS spectrum in isolation — the graph formulation requires multiple spectra from a complete LC-MS/MS run ordered by retention time
- No retention-order predictor output (RankSVM weight vector w and feature map phi) is available — then only the D=0 IOKR-only baseline applies and the shortest-path machinery adds nothing
- Observed retention times of adjacent layers are equal — the hinge penalty is zero there, so the path degenerates to IOKR score comparison for those transitions

## Inputs

- Per-spectrum scored candidate molecule lists from IOKR (candidates queried from PubChem by mass or predicted molecular formula), each with IOKR score y_i,j
- Measured retention times of the MS/MS spectra (layers sorted in increasing RT order)
- RankSVM weight vector w and molecular feature map phi (MACCS counting fingerprints, minmax kernel) from the retention-order prediction stage
- Penalty constant D (swept over d_lower=0 to d_upper=16)

## Outputs

- Optimal sequence of candidate molecular structures listed in retention order (one metabolite assignment per MS/MS spectrum)
- Average percentage of correctly identified molecular structures per D value (e.g. identification_accuracy_vs_D.csv)

## How to apply

Sort the per-spectrum IOKR candidate lists into layers of increasing measured retention time, with candidates within each layer ordered by decreasing IOKR score y_i,j. Connect every node n_i,j to every node n_r,s in the next layer (r = i+1) with weight d = -y_r,s + D * max(0, w^T(phi(m_i,j) - phi(m_r,s))), where w is the RankSVM weight vector and phi the MACCS counting-fingerprint feature map; set the hinge penalty to zero when the observed retention times of layers i and r are equal. Run the forward dynamic programming (Bellman) pass layer by layer, storing for each node the minimum-weight path from the first layer and its accumulated weight, with time complexity O(NK^2) for N layers and K the maximum layer size. At the last layer, pick the node with minimum total path weight and backtrack through stored predecessors to output the candidate molecular structures in retention order. The constant D balances the IOKR score against the order-conflict penalty and is swept over the repository's evaluated range d_lower=0 to d_upper=16; D=0 reproduces the IOKR-only identification baseline.

## Related tools

- **IOKR** (Produces the per-spectrum scored candidate lists that form the graph layers (input-output kernel regression with spectral input kernel and molecular output kernel))
- **networkx** (Represents the layered directed graph over candidate nodes for the shortest-path dynamic programming computation)
- **Python 3.5 and 3.6** (Hosts the dynamic programming module integrating the two predictors into a joint identification for a set of metabolites) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **numpy** (Computes the hinge penalty term D * max(0, w^T(phi(m_i,j) - phi(m_r,s))) over fingerprint feature maps)

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- D=0 must reproduce the IOKR-only identification baseline exactly, since the edge weight reduces to -y_r,s
- Confirm the recovered path is the minimum-total-weight path connecting the first layer to the last layer
- The hinge penalty term must be zero for edges between layers whose observed retention times are equal
- The joint assignment must yield more correctly identified molecular structures than IOKR scores alone — up to 24.8% improvement at D=0.0075 in the Others & target (LTSO) variant
- Runtime must scale as O(NK^2) with N layers and K the maximum layer size

## Limitations

- The hinge penalty is set to zero when the observed retention times of adjacent layers are equal, so order conflicts cannot be penalized for co-eluting spectra
- The penalty term penalizes only rank conflicts with the observed retention-time order, not absolute retention-time differences
- Requires IOKR-scored candidate lists whose quality depends on the PubChem mass/formula query; no separate repository link for IOKR is given in the article
- Quantitative accuracy values for different D are only reported via a figure (ECCB2018 report), not as tabulated numbers in the discussion

## Evidence
