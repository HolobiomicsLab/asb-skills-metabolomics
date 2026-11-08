---
name: roc-curve-evaluation
description: Use when a trained binary classifier on metabolite reaction-pair (RP
  vs. non-RP) predictions needs held-out performance evaluation via ROC analysis,
  threshold selection, and AUC reporting.
when_to_use_negative:
- Evaluating the filtered candidate set of the curated MRN — no clearly defined non-RPs
  exist under the screening conditions, and GNN-based performance metrics are inapplicable,
  so no FDR can be reported
- Tasks needing calibration or precision-recall analysis rather than ROC/threshold-Youden
  evaluation of an RP-vs-non-RP classifier
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2269
tools:
- name: PyTorch
  role: Framework for training the GNN reaction-pair classifier whose predictions
    are evaluated
- name: scikit-learn
  role: Computing the ROC curve and AUC on test-set prediction scores
- name: MetDNA3
  role: Pipeline whose curated metabolic reaction network (built from the GNN predictions)
    is the subject of the ROC evaluation
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:591646fcaaed4087d761e3facc71b71f26983adff77cb08896f29864e7877b60
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA3
  - PyTorch
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# roc-curve-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a binary reaction-pair classifier (the GNN used to curate MetDNA3's metabolic reaction network) by sweeping decision thresholds on a held-out test set, selecting the cutoff via the weighted Youden index, and reporting the ROC curve/AUC.

## When to use

Use when a trained binary classifier on metabolite reaction-pair (RP vs. non-RP) predictions needs held-out performance evaluation via ROC analysis, threshold selection, and AUC reporting.

## When NOT to use

- Evaluating the filtered candidate set of the curated MRN — no clearly defined non-RPs exist under the screening conditions, and GNN-based performance metrics are inapplicable, so no FDR can be reported
- Tasks needing calibration or precision-recall analysis rather than ROC/threshold-Youden evaluation of an RP-vs-non-RP classifier

## Inputs

- Curated reaction-pair dataset of 14,974 RPs and 14,974 non-RPs from knowledge databases (positive and negative pairs)
- Trained GNN reaction-pair classifier (PyTorch checkpoint)
- Held-out test split of n = 2,995 reaction pairs

## Outputs

- ROC curve (TPR vs. FPR) plot as in Fig. 1c
- ROC AUC value on the test set
- Selected decision threshold (reported: 0.7, maximizing weighted Youden index)
- Test-set prediction scores/labels table and saved model checkpoint

## How to apply

Split the balanced curated reaction-pair data — 14,974 RPs and 14,974 non-RPs — 8:1:1 into training, validation, and test sets, giving a test set of n = 2,995 pairs as reported. Build graph-convolutional molecular representations, molecular fingerprint features, and reaction-class features and train the GNN classifier (PyTorch framework), stopping on validation performance. Sweep the decision threshold on validation predictions and select the threshold maximizing the weighted Youden index, which yields the reported 0.7 cutoff. Apply the model to the held-out test set (n = 2,995), compute the ROC curve and AUC (e.g. with scikit-learn's roc_curve/auc), and plot true-positive rate vs. false-positive rate as in Fig. 1c. Save the model checkpoint, the test-set prediction-score table, and the ROC figure.

## Related tools

- **PyTorch** (Framework for training the GNN reaction-pair classifier whose predictions are evaluated)
- **scikit-learn** (Computing the ROC curve and AUC on test-set prediction scores)
- **MetDNA3** (Pipeline whose curated metabolic reaction network (built from the GNN predictions) is the subject of the ROC evaluation) — https://github.com/ZhuMetLab/MrnAnnoAlgo3

## Examples

```
from sklearn.metrics import roc_curve, auc; fpr, tpr, thr = roc_curve(y_test, scores); print(auc(fpr, tpr))
```

## Evaluation signals

- Test split matches the reported size: combined test set n = 2,995 reaction pairs drawn from the 8:1:1 split of 14,974 RPs + 14,974 non-RPs
- Threshold sweep on validation predictions yields a weighted-Youden-maximizing cutoff of 0.7
- ROC curve plotted as TPR vs. FPR reproduces Fig. 1c layout
- Saved artifacts present: model checkpoint, test-set prediction scores table, ROC figure

## Limitations

- Directly predicted reaction pairs may include a considerable number of false positives; the ROC/AUC metrics cannot be transferred to the filtered MRN candidate set because no clearly defined non-RPs exist under screening conditions
- No FDR is reported for the curated metabolic reaction network for these reasons
- Cross-tool/validation concordance may overestimate performance where chemical standards are lacking

## Evidence

- [discussion] due to the lack of clearly defined non-RPs under these screening conditions, and the inapplicability of applying GNN-based performance metrics: [evidence span withheld — anchor status: not_found]
- [results] The GNN model's reaction-pair prediction performance was evaluated with a receiver operating characteristic (ROC) curve: [evidence span withheld — anchor status: not_found]
