---
name: deep-convolutional-architecture-implementation
description: Use when binary class labels (e.g., pathway presence/absence) must be
  predicted from 1D waveform signals derived from network auralization, and a very
  deep 1D convolutional architecture with BCE loss is required.
when_to_use_negative:
- The task requires explaining predictions to biologists — NetAurHPD has low explainability,
  and the article notes it 'cannot provide biologists with a full understanding of
  its results'
- Inputs are network features rather than per-node/hyperlink waveform signals — use
  a GNN-style baseline (e.g., CHESHIRE) or the graph-auralization step first to obtain
  waveforms
- The classification problem is multi-class softmax regression rather than binary
  presence/absence prediction of pathway targets
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework in which the M5 architecture is built and trained (Algorithm 1 pseudo-code
    adapted for PyTorch implementation)
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: NetAurHPD_M5
  role: Module implementing the M5 architecture, its BCE training loop, and evaluation
    (NetAurHPD_M5.py)
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: result_optimization_and_analysis
  role: Module computing Youden's J threshold and evaluation metrics (AUC, accuracy,
    recall, precision, F1, FPR) after M5 prediction
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: config
  role: Hyperparameter module defining stride=8, n_channel=32, epochs, lr for the
    M5 layer
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:83e50ca9cfadf9d89c0f30694fb528262031acebf1e6fb75e0463061b6fa82ef
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
  - config
  - result_optimization_and_analysis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# deep-convolutional-architecture-implementation

## Summary

Implements and retrains the M5 very-deep 1D CNN (originally built for environmental sound recognition) as a binary classifier in PyTorch, using BCE loss with the Adam optimizer, to predict pathway presence/absence from averaged auralized pathway waveforms in NetAurHPD.

## When to use

Use when binary class labels (e.g., pathway presence/absence) must be predicted from 1D waveform signals derived from network auralization, and a very deep 1D convolutional architecture with BCE loss is required.

## When NOT to use

- The task requires explaining predictions to biologists — NetAurHPD has low explainability, and the article notes it 'cannot provide biologists with a full understanding of its results'
- Inputs are network features rather than per-node/hyperlink waveform signals — use a GNN-style baseline (e.g., CHESHIRE) or the graph-auralization step first to obtain waveforms
- The classification problem is multi-class softmax regression rather than binary presence/absence prediction of pathway targets

## Inputs

- Pathway waveform signals: per-pathway averaged auralized waveforms of length l=10000 from the metabolite correlation network
- Binary labels per pathway (positive/negative targets, e.g., dicts of the form {'label': 'positive', 'nodes': [...]})
- Train/test waveform tensors y_train, y_test

## Outputs

- Trained M5 binary classifier (PyTorch model)
- Predicted probabilities/scores per pathway (y_pred) after threshold optimization
- Micro-average AUC from 10-fold cross-validation, plus accuracy, TPR, FPR, precision, and F1 at the Youden-optimal threshold

## How to apply

Build M5 as four convolution blocks, each of a convolutional layer, ReLU activation, batch normalization, and max-pool layer, followed by average pooling and a fully connected layer with a sigmoid output (instantiated as NetAurHPD_M5(n_input=1, n_output=1, stride=8, n_channel=32) per the repo config). Convert the original softmax regressor into a classifier by feeding each pathway's averaged waveform (participants' node waveforms averaged by index, waveform length l=10000) as input. Train with Binary Cross Entropy loss and the Adam optimizer; the paper uses lr=0.0001 with stride 8 for 300 epochs without mini-batching (the repo config defaults differ: epochs=40, lr=0.01). Evaluate with 10-fold cross-validation collecting micro-average AUC over all validation predictions, then compute the Youden index (argmax of TPR - FPR over thresholds) to pick the cutoff before reporting accuracy, TPR, FPR, precision, and F1.

## Related tools

- **PyTorch** (Framework in which the M5 architecture is built and trained (Algorithm 1 pseudo-code adapted for PyTorch implementation)) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **NetAurHPD_M5** (Module implementing the M5 architecture, its BCE training loop, and evaluation (NetAurHPD_M5.py)) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **result_optimization_and_analysis** (Module computing Youden's J threshold and evaluation metrics (AUC, accuracy, recall, precision, F1, FPR) after M5 prediction) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **config** (Hyperparameter module defining stride=8, n_channel=32, epochs, lr for the M5 layer) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
NetAurHPD_DL_architecture = NetAurHPD_M5(n_input=1, n_output=1, stride=config.stride, n_channel=config.n_channel); y_pred = NetAurHPD_DL_architecture.predict(train_hyperlinks_waveforms, y_train, test_hyperlinks_waveforms, y_test, lr=config.lr, total_iters=config.epochs)
```

## Evaluation signals

- Micro-average AUC on held-out folds reaches ~0.85–0.88 as reported for tomato pericarp datasets (0.87 in 2001, 0.857 in 2003, 0.884 in 2004)
- Accuracy at the Youden-optimal threshold is around 0.8
- Model converges under BCE loss with Adam at the configured learning rate over the full epoch budget without mini-batching
- Metrics reported only after threshold optimization via the Youden index (argmax of TPR - FPR), not at the default 0.5 cutoff
- Input waveform tensors match the configured shape: stride=8, n_channel=32, waveform length l=10000

## Limitations

- NetAurHPD has a low level of explainability and cannot provide biologists with a full understanding of its results
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased
- Sparsity of labeled data is the most important challenge in metabolic pathway prediction
- Repo config defaults (epochs=40, lr=0.01) differ from the paper's training settings (lr=0.0001, 300 epochs), so hyperparameters must be set explicitly

## Evidence

- [methods] we will train M5 as classification model (Supplementary Fig 7.g) to predict the class of each pathway: [evidence span withheld — anchor status: not_found]
