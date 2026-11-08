---
name: metabolite-correlation-network-diffusion
description: Use when a weighted Pearson-correlation Metabolite CN (undirected, |r|≥0.3,
  p≤0.01) must be converted into fixed-length node waveforms (e.g. l=10,000) that
  can be averaged into hyperlink/pathway signals for downstream signal classifiers.
when_to_use_negative:
- The input is already a feature matrix, embedding, or a table of node attributes
  — no propagation is needed.
- The network is directed or weighted in a way that does not support the P_{u,v} =
  A_{u,v}/D_u transfer-power definition.
- You need biologically interpretable pathway predictions directly; NetAurHPD's waveform
  representation has low explainability and cannot provide biologists a full understanding
  of its results.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: NetAurHPD
  role: Framework whose network-auralization module (SignalPropagation / network_auralization.py)
    implements the energy propagation that generates per-node waveforms from the Metabolite
    CN
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation backend for Algorithm 1 network auralization pseudo-code and
    waveform tensor computation
- name: NetAurHPD_M5
  role: 'Downstream consumer: M5 classifier that ingests the resulting hyperlink waveforms
    for binary classification'
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_content_hash: sha256:6d1756d4f1fcbc67eacdf4767a035478e4bea4212e86baac55be07c51d36f086
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - M5
  - NetAurHPD
  - NetAurHPD_M5
  - PyTorch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# metabolite-correlation-network-diffusion

## Summary

Transforms a Metabolite CN into per-node waveform representations by propagating a unit energy impulse through the network with constant transfer power and momentum until energy is evenly distributed, yielding a length-l waveform per node.

## When to use

Use when a weighted Pearson-correlation Metabolite CN (undirected, |r|≥0.3, p≤0.01) must be converted into fixed-length node waveforms (e.g. l=10,000) that can be averaged into hyperlink/pathway signals for downstream signal classifiers.

## When NOT to use

- The input is already a feature matrix, embedding, or a table of node attributes — no propagation is needed.
- The network is directed or weighted in a way that does not support the P_{u,v} = A_{u,v}/D_u transfer-power definition.
- You need biologically interpretable pathway predictions directly; NetAurHPD's waveform representation has low explainability and cannot provide biologists a full understanding of its results.

## Inputs

- Metabolite CN: undirected, unweighted metabolite correlation network (nodes + adjacency matrix) built with Pearson Correlation Coefficient threshold |r| ≥ 0.3 and p ≤ 0.01
- List of network nodes
- Hyperlink dictionaries with samples shaped ID: {'label': 'positive'|'negative', 'nodes': [u, v]}
- Momentum m (e.g. 0.999), response/waveform length l (e.g. 10,000), and device setting

## Outputs

- Per-node waveform tensor/array (waveforms.npy), each of length l = 10,000 with DC component removed
- Auralized networkx graph representation consumable by downstream waveform-averaging (hyperlinks_waveforms.py) and M5 classification stages

## How to apply

Load the Metabolite CN as an undirected, unweighted graph with adjacency matrix A and degree D, and inject an energy impulse into a single node. Iteratively propagate energy via S_t = Diag(S_{t-1}) × P with constant transfer power P_{u,v} = A_{u,v}/D_u, preserving a fraction m of the previous iteration's flow (momentum, ΔS_{t,u,v} = S_{t-1,u}×P_{u,v} + m×ΔS_{t-1,u,v}; the README's SignalPropagation uses momentum=0.999). Continue until total energy is evenly distributed, record each node's potential across iterations as its waveform (length l=10,000), and strip the DC component — the stabilized end-state values — from each vector. Save the per-node waveforms (PyTorch tensor or numpy array, e.g. waveforms.npy) so participants' waveforms can later be averaged into pathway or hyperlink signals. The implementation in network_auralization.py (SignalPropagation class, networkx_auralization method) is PyTorch-based and takes dicts of positive/train/test hyperlinks plus the node list.

## Related tools

- **NetAurHPD** (Framework whose network-auralization module (SignalPropagation / network_auralization.py) implements the energy propagation that generates per-node waveforms from the Metabolite CN) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation backend for Algorithm 1 network auralization pseudo-code and waveform tensor computation)
- **NetAurHPD_M5** (Downstream consumer: M5 classifier that ingests the resulting hyperlink waveforms for binary classification) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device); train_wfs, test_wfs = instrument.networkx_auralization(train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict, nodes, how_graph=True)
```

## Evaluation signals

- Each node's waveform has length l = 10,000 (response_len) and no remaining DC/stabilized component after removal
- The propagation terminates only when total energy is evenly distributed throughout the entire network (convergence criterion)
- Saved waveform file (e.g. waveforms.npy) loads as a PyTorch tensor or numpy array of shape (n_nodes, l)
- Averaging participant waveforms by index yields pathway signals of consistent dimensionality feeding the M5 model with stride 8 and 32 output channels
- Re-running with the same momentum (0.999) and response_len reproduces identical waveforms for the same Metabolite CN

## Limitations

- NetAurHPD has a low level of explainability and cannot provide biologists with a full understanding of its results
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased
- Waveform length l = 10,000 and momentum m = 0.999 are fixed defaults; propagation may be computationally heavy on large networks
- Only pathways with more than two compounds appearing in the Metabolite CN are eligible, limiting applicability to sparse pathway sets

## Evidence

- [methods] Threshold for building the Metabolite CN that feeds auralization: [evidence span withheld — anchor status: not_found]
