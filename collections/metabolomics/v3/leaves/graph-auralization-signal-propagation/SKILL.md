---
name: graph-auralization-signal-propagation
description: Use when a graph (e.g., Pearson-correlation Metabolite CN with |r|>=0.3,
  p<=0.01) must be converted into fixed-length node/hyperlink signal representations
  for deep-learning classification, such as predicting pathway presence from metabolomics
  networks.
when_to_use_negative:
- The input is already a feature/numeric table rather than a graph — auralization
  requires nodes and links, not tabular features
- Networks without meaningful link structure, where averaging member waveforms would
  not capture pathway-level information
- Tasks requiring biologically interpretable feature importances — the paper notes
  NetAurHPD 'has a low level of explainability'
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: NetAurHPD
  role: Framework containing the SignalPropagation class that implements network auralization
    over NetworkX graphs
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Implementation framework for Algorithm 1 (network auralization) and the M5
    classifier
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: M5
  role: Very deep 1D convolutional network (n_channel=32, stride=8) trained on the
    auralized hyperlink waveforms for binary pathway classification
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: centrality-learning
  role: Companion repo (centrality_from_auralized_nodes) for learning node centrality
    measures from auralized signals
  repo: https://github.com/puzis/centrality-learning
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:eea22bd07eb0ffdb24df168a6e697963639992a37c5646ce2057663d16c4b822
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
  - PyTorch
  - Rstudio
  - centrality-learning
  - pcaMethods
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# graph-auralization-signal-propagation

## Summary

Transforms a metabolite correlation network (Metabolite CN) into per-node waveform representations by propagating energy among nodes until it is evenly distributed, then averages participant waveforms into hyperlink (pathway) signals for downstream M5 classification. This is the core representation-learning step of NetAurHPD.

## When to use

Use when a graph (e.g., Pearson-correlation Metabolite CN with |r|>=0.3, p<=0.01) must be converted into fixed-length node/hyperlink signal representations for deep-learning classification, such as predicting pathway presence from metabolomics networks.

## When NOT to use

- The input is already a feature/numeric table rather than a graph — auralization requires nodes and links, not tabular features
- Networks without meaningful link structure, where averaging member waveforms would not capture pathway-level information
- Tasks requiring biologically interpretable feature importances — the paper notes NetAurHPD 'has a low level of explainability'

## Inputs

- NetworkX graph or Metabolite correlation network (nodes = metabolites, edges = Pearson correlations meeting |r|>=0.3, p<=0.01)
- Hyperlink dictionaries of the form ID: {'label': 'positive'|'negative', 'nodes': [node_ids]}
- Node list and train/test hyperlink splits with y_train/y_test tensors

## Outputs

- Per-node waveform arrays (length l = 10,000)
- Averaged per-hyperlink waveform signals for training and test hyperlinks
- Downstream M5 predictions and micro-average AUC metrics after Youden-threshold optimization

## How to apply

Construct or load the network (for NetAurHPD, a Metabolite CN built with Pearson correlation at threshold |r|>=0.3 and p<=0.01 over the metabolites), then run network auralization per Algorithm 1: energy propagates among nodes to their neighbors until total energy is evenly distributed, yielding a waveform per node (default length l=10,000). In the PyTorch implementation, instantiate the SignalPropagation class with momentum=0.999 and response_len=10000 and call networkx_auralization over train/test hyperlinks. Aggregate node waveforms into hyperlink (pathway) signals by averaging each pathway's participants' waveforms by index. The resulting signals feed the M5 convolutional network (stride 8, Adam optimizer, lr 0.0001) trained with 10-fold cross-validation; classification threshold is later optimized via Youden's J statistic.

## Related tools

- **NetAurHPD** (Framework containing the SignalPropagation class that implements network auralization over NetworkX graphs) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Implementation framework for Algorithm 1 (network auralization) and the M5 classifier) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **M5** (Very deep 1D convolutional network (n_channel=32, stride=8) trained on the auralized hyperlink waveforms for binary pathway classification) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **centrality-learning** (Companion repo (centrality_from_auralized_nodes) for learning node centrality measures from auralized signals) — https://github.com/puzis/centrality-learning

## Examples

```
instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device); train_w, test_w = instrument.networkx_auralization(train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict, nodes, how_graph=True)
```

## Evaluation signals

- Waveforms have the configured response length (l = 10,000 samples) for every node
- On tomato pericarp Metabolite CNs, micro-average AUC from 10-fold CV should approximate 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Each hyperlink waveform equals the index-wise average of its participating metabolites' waveforms
- M5 training uses Adam with lr=0.0001 and the classification threshold is set by maximizing Youden's J (TPR - FPR)

## Limitations

- NetAurHPD has low explainability and cannot give biologists a full understanding of its predictions
- Models do not consistently improve with more data; in some sessions AUCs of all models decreased
- Labeled data sparsity is the most important challenge; data augmentation (generating artificial Metabolite CNs from known reactions) is needed to stabilize training

## Evidence

- [methods] The Metabolite CN will be transformed into an aural representation (Supplementary Fig 7.c) as described in Algorithm 1: [evidence span withheld — anchor status: not_found]
- [methods] we average the waveforms of each pathway's participants by index (Supplementary Fig 7.e): [evidence span withheld — anchor status: not_found]
