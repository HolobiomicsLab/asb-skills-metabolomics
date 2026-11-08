---
name: waveform-dc-component-removal
description: Use when converting per-node energy-propagation traces (e.g. from Algorithm
  1 of NetAurHPD) into waveforms for downstream averaging/aggregation, and the stabilized
  steady-state energy needs to be stripped from each vector.
when_to_use_negative:
- The input is already a finalized waveform tensor (e.g. waveforms.npy) rather than
  raw propagation traces — re-running DC removal would corrupt the signal.
- The task is centrality learning from auralized nodes (e.g. centrality_from_auralized_nodes
  in the cnai-lab repo) where the stabilized component itself may carry positional
  information and is not meant to be stripped.
- The analysis goal is the propagation dynamics itself (e.g. studying how energy is
  evenly distributed), where removing the DC component discards the quantity of interest.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: NetAurHPD
  role: Framework whose auralization stage produces per-node waveforms and whose hyperlinks_waveforms
    module performs averaging; DC removal sits between auralization and aggregation.
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation platform for Algorithm 1 network auralization adapted for PyTorch;
    waveforms and DC-removed tensors are handled as PyTorch tensors.
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: SignalPropagation
  role: Class in network_auralization.py implementing the Network Auralization method;
    its output potentials are the waveforms from which the DC component is removed
    before aggregation.
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:caf2ae60b66b2c8aec8b0103de0bc56cfebd2ba233f267ff72745dbe994b1dda
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - PyTorch
  - SignalPropagation
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# waveform-dc-component-removal

## Summary

During network auralization of a Metabolite CN, each node's propagated energy over iterations forms a waveform; this skill removes the stabilized (DC) component from each node's vector so downstream aggregation compares only the transient signal that encodes the node's position.

## When to use

Use when converting per-node energy-propagation traces (e.g. from Algorithm 1 of NetAurHPD) into waveforms for downstream averaging/aggregation, and the stabilized steady-state energy needs to be stripped from each vector.

## When NOT to use

- The input is already a finalized waveform tensor (e.g. waveforms.npy) rather than raw propagation traces — re-running DC removal would corrupt the signal.
- The task is centrality learning from auralized nodes (e.g. centrality_from_auralized_nodes in the cnai-lab repo) where the stabilized component itself may carry positional information and is not meant to be stripped.
- The analysis goal is the propagation dynamics itself (e.g. studying how energy is evenly distributed), where removing the DC component discards the quantity of interest.

## Inputs

- Per-node energy-propagation traces (potential over iterations) produced by Algorithm 1 network auralization, stored as a PyTorch tensor or numpy array
- Metabolite CN adjacency information used to drive the propagation (undirected, unweighted graph with nodes and adjacency matrix)

## Outputs

- DC-removed per-node waveforms (length l, e.g. 10,000) ready for waveform averaging into pathway signals
- Saved waveform file such as waveforms.npy (numpy array or PyTorch tensor) for the next NetAurHPD stage

## How to apply

Run network auralization by propagating energy with S_t = Diag(S_{t-1}) x P, where P_{u,v} = A_{u,v}/D_u, until the total energy is evenly distributed throughout the entire network; with momentum, extend stabilization via Delta S_{t,u,v} = S_{t-1,u} x P_{u,v} + m x Delta S_{t-1,u,v} (the NetAurHPD example uses momentum=0.999 and response_len=10000). Record each node's potential over iterations as its waveform of length l (l = 10,000 per config). Then remove the DC component — the stabilized values present in every iteration — from each waveform, so that only the transient propagation profile remains. This is done before averaging the waveforms of each pathway's participants by index to produce hyperlink/pattern signals, or before saving the per-node waveforms to waveforms.npy for the next NetAurHPD stage.

## Related tools

- **NetAurHPD** (Framework whose auralization stage produces per-node waveforms and whose hyperlinks_waveforms module performs averaging; DC removal sits between auralization and aggregation.) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation platform for Algorithm 1 network auralization adapted for PyTorch; waveforms and DC-removed tensors are handled as PyTorch tensors.) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **SignalPropagation** (Class in network_auralization.py implementing the Network Auralization method; its output potentials are the waveforms from which the DC component is removed before aggregation.) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
from NetAurHPD.network_auralization import SignalPropagation; instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device)
```

## Evaluation signals

- After DC removal, the mean of each node's waveform across the final/stabilized iterations should be approximately zero, whereas the raw trace has a non-zero stabilized baseline.
- All waveforms must have identical length l (default 10,000 in the NetAurHPD config) after DC removal, so that averaging participants' waveforms by index is well-defined.
- Removing the DC component from a constant (fully stabilized) signal should yield a near-zero vector; a non-zero remainder indicates incomplete stabilization before extraction.
- Downstream averaging of participants' waveforms (per pathway) should produce one waveform per pathway/hyperlink of the same length; if aggregation fails, check DC removal and length consistency.
- The saved waveforms.npy should load as a PyTorch tensor or numpy array whose shape is (number of nodes, l) matching the Metabolite CN's node count.

## Limitations

- The waveform length l is fixed (10,000 in config); DC removal must operate on traces of that length, and shorter traces (insufficient stabilization iterations) will retain residual DC bias.
- The procedure assumes the propagation has actually reached even distribution of total energy; if iterations are truncated early, the 'stabilized values' are not true steady-state and their removal misrepresents the node.
- DC removal discards the absolute energy level; analyses that depend on absolute potential (e.g. centrality magnitude from auralized nodes) should not use this preprocessing step.

## Evidence

- [other] Record each node's potential over iterations as its waveform, remove the DC component (the stabilized values) from each vector: [evidence span withheld — anchor status: not_found]
- [other] Run the propagation until the total energy is evenly distributed throughout the entire network: [evidence span withheld — anchor status: not_found]
- [methods] we average the waveforms of each pathway's participants by index (Supplementary Fig 7.e): [evidence span withheld — anchor status: not_found]
