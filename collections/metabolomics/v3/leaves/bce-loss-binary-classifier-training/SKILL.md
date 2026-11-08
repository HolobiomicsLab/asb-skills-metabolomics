---
name: bce-loss-binary-classifier-training
description: Use when you must train a binary classifier (presence/absence, label
  per entity such as a metabolic pathway) from per-instance waveform/signals averaged
  from node representations, and need an ROC-optimized decision threshold rather than
  a default 0.5 cutoff.
when_to_use_negative:
- Inputs are tabular feature vectors or graph-structural baselines, not auralized
  waveform signals averaged over hyperlink/pathway participants
- The task needs biologically interpretable predictions — NetAurHPD 'has a low level
  of explainability'
- You need calibrated probabilities rather than thresholded binary labels; the Youden
  cutoff is optimized on TPR−FPR and sacrifices calibration
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: M5
  role: Very deep convolutional neural network (originally for environmental sound
    recognition) rebuilt with sigmoid output and trained for binary pathway classification
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Deep-learning framework in which the M5 architecture, BCE loss, Adam optimizer
    and BCE optimization are implemented
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: Adam optimizer
  role: Gradient optimizer used at learning rate 0.0001 for 300 epochs without mini-batching
- name: NetAurHPD_M5
  role: Repository module that wraps the M5 model, its training loop, and prediction
    of test hyperlink waveforms
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method/blob/master/NetAurHPD/NetAurHPD_M5.py
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:43c351cd4891c67508140fc47c615d4f5750f24f09fd4f0cac07994c0de94682
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# bce-loss-binary-classifier-training

## Summary

Trains a binary classification model on averaged auralized signals (within NetAurHPD, pathway waveforms) using the very-deep M5 CNN rebuilt from softmax regression into a sigmoid-output binary classifier, trained with Binary Cross Entropy loss and thresholded via the Youden index.

## When to use

Use when you must train a binary classifier (presence/absence, label per entity such as a metabolic pathway) from per-instance waveform/signals averaged from node representations, and need an ROC-optimized decision threshold rather than a default 0.5 cutoff.

## When NOT to use

- Inputs are tabular feature vectors or graph-structural baselines, not auralized waveform signals averaged over hyperlink/pathway participants
- The task needs biologically interpretable predictions — NetAurHPD 'has a low level of explainability'
- You need calibrated probabilities rather than thresholded binary labels; the Youden cutoff is optimized on TPR−FPR and sacrifices calibration

## Inputs

- Positive/negative hyperlink dicts of the form ID: {'label': ..., 'nodes': [...]}
- Averaged auralized pathway/hyperlink waveforms (train and test, from SignalPropagation.networkx_auralization)
- y_train / y_test label tensors

## Outputs

- Binary class predictions for each pathway/hyperlink at the optimized Youden threshold
- Micro-average AUC over 10-fold cross-validation predictions
- Accuracy, TPR, FPR, precision and F1 at the optimal cutoff

## How to apply

Build the M5 architecture as four convolution blocks, each with a convolutional layer, ReLU activation, batch normalization and a max-pool layer, followed by average pooling and a fully connected layer with a sigmoid output (README config: stride=8, n_channel=32, waveform length l=10,000). Aggregate signals per hyperlink/pathway by averaging participant waveforms by index, then train on the positive/negative instance dict (e.g. `{'label': 'positive', 'nodes': [61, 108]}`) minimizing Binary Cross Entropy loss with the Adam optimizer at learning rate 0.0001, stride size 8, for 300 epochs without mini-batching (per the paper; repository defaults in config.py are lr=0.01, epochs=40). Evaluate with 10-fold cross-validation in which each pathway appears once in the validation fold, collecting micro-average AUC over all validation predictions. Compute the Youden index — the argmax of TPR − FPR over candidate thresholds — to pick the cutoff, then report accuracy, TPR, FPR, precision and F1 at that threshold.

## Related tools

- **M5** (Very deep convolutional neural network (originally for environmental sound recognition) rebuilt with sigmoid output and trained for binary pathway classification) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Deep-learning framework in which the M5 architecture, BCE loss, Adam optimizer and BCE optimization are implemented) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **Adam optimizer** (Gradient optimizer used at learning rate 0.0001 for 300 epochs without mini-batching)
- **NetAurHPD_M5** (Repository module that wraps the M5 model, its training loop, and prediction of test hyperlink waveforms) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method/blob/master/NetAurHPD/NetAurHPD_M5.py

## Examples

```
instrument = SignalPropagation(momentum=0.999, response_len=10000, tqdm=lambda x: x, device); train_w, test_w = instrument.networkx_auralization(train_pos, train_hyperlinks, test_hyperlinks, nodes, how_graph=True); model = NetAurHPD_M5(n_input=1, n_output=1, stride=config.stride, n_channel=config.n_channel); y_pred = model.predict(train_w, y_train, test_w, y_test, lr=config.lr, total_iters=config.epochs)
```

## Evaluation signals

- Micro-average AUC over held-out validation folds lands around 0.85–0.88 (paper reports 0.87 for the 2001 tomato pericarp season, 0.857 for 2003, 0.884 for 2004)
- Accuracy at the Youden-optimized threshold is approximately 0.8 on the tomato pericarp datasets
- Every pathway in the dataset appears exactly once in the validation set across the 10-fold cross-validation
- The reported threshold equals argmax over candidate thresholds of (TPR − FPR), not a fixed 0.5
- Sigmoid output is constrained to [0,1]; predictions after thresholding are strictly binary 0/1 class labels

## Limitations

- Repository config.py defaults (lr=0.01, epochs=40) differ from the paper's hyperparameters (lr=0.0001, 300 epochs); reuse must override the config to reproduce the paper's results
- NetAurHPD has low explainability and cannot give biologists a full mechanistic understanding of predicted pathways
- Sparsity of labeled data is the main challenge; performance does not consistently improve with more augmented data — in some seasons the AUCs of all models decreased

## Evidence

- [methods] Build the M5 architecture with four convolution blocks, each comprising a convolutional layer, ReLU activation, batch normalization, and a max-pool layer, followed by average pooling and a fully connected layer with a sigmoid output.: [evidence span withheld — anchor status: not_found]
- [methods] We employed the Adam optimizer with a learning rate of 0.0001: "We employed the Adam optimizer with a learning rate of 0.0001"
- [methods] Youden index is a statistical method used to learn the optimal threshold from the True Positive Rate (TPR) and False Positive Rate (FPR): [evidence span withheld — anchor status: not_found]
- [results] In 2001 the micro-average AUC was 0.87 ... For 2003 the micro-average AUC was 0.857 ... In 2004 the micro-average AUC was 0.884: "In 2001 the micro-average AUC was 0.87"
- [readme] The M5 architecture is a very deep convolutional neural network designed for sound tasks. In this case, it is structured for binary classification tasks.: "The M5 architecture is a very deep convolutional neural network designed for sound tasks. In this case, it is structured for binary classification tasks."
