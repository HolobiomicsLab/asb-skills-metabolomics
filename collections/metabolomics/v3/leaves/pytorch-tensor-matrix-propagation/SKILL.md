---
name: pytorch-tensor-matrix-propagation
description: Use when you need a waveform (signal) representation of every node in
  an undirected graph — e.g. a Metabolite CN adjacency matrix or a networkx graph
  — as input to downstream aggregation (hyperlink waveforms, M5 classifier, centrality
  learning).
when_to_use_negative:
- The input is already a set of node embeddings or features (e.g. a PCA/PCA-methods
  output) — auralization extracts its own waveform representation, so re-running would
  duplicate the representation step
- You need a trained classifier directly; this skill only produces per-node waveforms
  and must be followed by hyperlink averaging and M5 training
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3922
tools:
- name: NetAurHPD
  role: Source framework; SignalPropagation class implements network auralization
    over networkx graphs
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Engine for the iterative S_t = Diag(S_{t-1}) × P propagation; Algorithm 1
    is adapted for PyTorch implementation
- name: cnai-lab/centrality-learning
  role: Sibling repository where the auralized-node waveforms are used for centrality
    learning (centrality_from_auralized_nodes)
  repo: https://github.com/puzis/centrality-learning
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:694b2289c26887e03cd7a3172ac463ddf580b25b1fc98d57089b792fea84c7ed
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - NetAurHPD
  - PyTorch
  - cnai-lab/centrality-learning
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# pytorch-tensor-matrix-propagation

## Summary

PyTorch implementation of the network auralization energy-propagation scheme from NetAurHPD: nodes receive an energy impulse and the flow S_t = Diag(S_{t-1}) × P is iterated with constant transfer power P_{u,v} = A_{u,v}/D_u until energy is evenly distributed, yielding a per-node waveform.

## When to use

Use when you need a waveform (signal) representation of every node in an undirected graph — e.g. a Metabolite CN adjacency matrix or a networkx graph — as input to downstream aggregation (hyperlink waveforms, M5 classifier, centrality learning).

## When NOT to use

- The input is already a set of node embeddings or features (e.g. a PCA/PCA-methods output) — auralization extracts its own waveform representation, so re-running would duplicate the representation step
- You need a trained classifier directly; this skill only produces per-node waveforms and must be followed by hyperlink averaging and M5 training

## Inputs

- Undirected, unweighted graph (Metabolite CN adjacency matrix or networkx graph)
- Positive/negative hyperlink dicts in the shape {'label': 'positive'|'negative', 'nodes': [u, v]}
- Nodes list
- PyTorch device

## Outputs

- Per-node waveform tensor or numpy array (e.g. waveforms.npy), length l=10000 per node, DC component removed

## How to apply

Load the undirected, unweighted Metabolite CN and instantiate the SignalPropagation class with momentum m=0.999, response_len=10000, and a PyTorch device. Initialize the energy vector with an impulse on one node and iterate S_t = Diag(S_{t-1}) × P, where the constant transfer power is P_{u,v} = A_{u,v}/D_u (adjacency divided by node degree), until the total energy is evenly distributed across the entire network. Momentum is applied per step via ΔS_{t,u,v} = S_{t-1,u} × P_{u,v} + m × ΔS_{t-1,u,v} to extend the stabilization horizon. Record each node's potential across iterations as its waveform, then remove the DC component (the stabilized values) from each vector; default waveform length is l=10000 samples per node. Invoke networkx_auralization() on train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict and the nodes list with how_graph=True to auralize over a networkx graph, and save the resulting per-node waveforms (e.g. waveforms.npy) for the next NetAurHPD stage.

## Related tools

- **NetAurHPD** (Source framework; SignalPropagation class implements network auralization over networkx graphs) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Engine for the iterative S_t = Diag(S_{t-1}) × P propagation; Algorithm 1 is adapted for PyTorch implementation)
- **cnai-lab/centrality-learning** (Sibling repository where the auralized-node waveforms are used for centrality learning (centrality_from_auralized_nodes)) — https://github.com/puzis/centrality-learning

## Examples

```
instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device); train_waveforms, test_waveforms = instrument.networkx_auralization(train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict, nodes, how_graph=True)
```

## Evaluation signals

- Waveform length equals the configured response_len (l = 10,000 samples per node)
- Each node's waveform has zero DC component after removal of the stabilized values
- At convergence the total energy is evenly distributed across the entire network
- Downstream aggregation: averaging participant waveforms must reproduce each pathway/hyperlink signal consumed by M5
- Re-running with the same momentum (0.999) and seed yields consistent waveforms across nodes

## Limitations

- Stabilization horizon depends on the momentum parameter m; setting it too low underdamps the propagation
- NetAurHPD as a whole has low explainability — auralization output is not directly interpretable biologically
- Waveform length l=10000 and momentum are defaults from config; other graphs may require tuning
- Directed or weighted graphs are outside the described scheme (constant transfer power assumes unweighted, undirected edges with P_{u,v} = A_{u,v}/D_u)

## Evidence

- [methods] Transfer power formula from task card: [evidence span withheld — anchor status: not_found]
