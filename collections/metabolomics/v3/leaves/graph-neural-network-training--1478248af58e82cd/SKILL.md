---
name: graph-neural-network-training
description: Use when building a knowledge-driven metabolic reaction network requires
  predicting metabolite-metabolite reaction pairs with a GNN, and when ROC/AUC evaluation
  of that predictor on a held-out test split is needed.
when_to_use_negative:
- When the candidate reaction pairs have already been screened/filtered into the working
  MRN — GNN performance metrics are inapplicable to the filtered candidate set, and
  no FDR can be reported for the curated MRN
- When a curated MRN already covers the metabolites of interest and no GNN-based prediction
  of new reaction relationships is needed
- When clearly defined non-RPs (negative examples) cannot be constructed for the screening
  conditions, since training and performance assessment require them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3174
tools:
- name: MetDNA3
  role: Ecosystem whose MRN is curated using the GNN-predicted reaction relationships;
    the GNN module supports knowledge-driven network construction
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: PyTorch
  role: GNN framework used to build graph convolutional molecular representations
    and train the reaction-pair classifier
- name: scikit-learn
  role: Used to compute the ROC curve and AUC on the held-out test-set predictions
    (Fig. 1c)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:15bca0c4b2a6e334794e18fb9b4dd4a04c7a61689d0977aeb81c2b1a2f6e07d3
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# graph-neural-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a graph neural network (GNN) classifier on reported and non-reaction pairs from knowledge databases to predict potential reaction relationships between metabolites, then evaluates it with a ROC curve on a held-out test set for curating a comprehensive metabolic reaction network (MRN) in MetDNA3.

## When to use

Use when building a knowledge-driven metabolic reaction network requires predicting metabolite-metabolite reaction pairs with a GNN, and when ROC/AUC evaluation of that predictor on a held-out test split is needed.

## When NOT to use

- When the candidate reaction pairs have already been screened/filtered into the working MRN — GNN performance metrics are inapplicable to the filtered candidate set, and no FDR can be reported for the curated MRN
- When a curated MRN already covers the metabolites of interest and no GNN-based prediction of new reaction relationships is needed
- When clearly defined non-RPs (negative examples) cannot be constructed for the screening conditions, since training and performance assessment require them

## Inputs

- Curated reaction-pair training table with 14,974 reported RPs and 14,974 non-RPs from knowledge databases
- Molecular structures of metabolites (for graph convolutional and fingerprint features)
- Reaction-class annotations for each candidate pair
- 8:1:1 train/validation/test split specification (test n = 2995)

## Outputs

- Trained GNN reaction-pair classifier model checkpoint
- Test-set prediction scores table for the n = 2995 held-out reaction pairs
- ROC curve figure (true positive rate vs. false positive rate, as in Fig. 1c) with AUC
- Decision threshold (0.7, selected by maximum weighted Youden index on validation)

## How to apply

Curate a balanced reaction-pair dataset of 14,974 reported RPs and 14,974 non-RPs drawn from knowledge databases, then split it 8:1:1 into training, validation, and test sets (test set n = 2995). Build three feature streams as model inputs: graph-based molecular representations via graph convolutional layers, molecular fingerprint features, and reaction-class features, implemented in a GNN framework such as PyTorch (e.g. PyTorch Geometric / DGL). Train the binary reaction-pair classifier on the training split while monitoring validation performance for early stopping. Sweep the decision threshold on validation predictions and select the cutoff that maximizes the weighted Youden index, which yields the reported 0.7 cutoff. Apply the trained model to the n = 2995 held-out test set, compute the ROC curve and AUC on the prediction scores, and plot true positive rate vs. false positive rate as in Fig. 1c using scikit-learn. Save the model checkpoint, the test-set prediction-score table, and the ROC figure as reproducible artifacts.

## Related tools

- **MetDNA3** (Ecosystem whose MRN is curated using the GNN-predicted reaction relationships; the GNN module supports knowledge-driven network construction) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **PyTorch** (GNN framework used to build graph convolutional molecular representations and train the reaction-pair classifier)
- **scikit-learn** (Used to compute the ROC curve and AUC on the held-out test-set predictions (Fig. 1c))

## Examples

```
from sklearn.metrics import roc_curve, auc; fpr, tpr, _ = roc_curve(y_test, gnn_scores); roc_auc = auc(fpr, tpr)
```

## Evaluation signals

- Class balance holds across the split: 14,974 RPs and 14,974 non-RPs divided 8:1:1, giving a test set of n = 2995 pairs
- The decision threshold chosen on the validation split by the weighted Youden index equals the reported 0.7 cutoff
- The ROC curve on the n = 2995 test set is monotone in TPR/FPR with a computable AUC consistent with Fig. 1c
- A saved model checkpoint plus test-set prediction-score table allow exact re-plotting of the ROC curve
- Validation-split monitoring stops training before overfitting, and validation performance tracks test AUC without large divergence

## Limitations

- Direct GNN prediction of reaction pairs may include a considerable number of false positives, requiring downstream two-step pre-screening of the MRN
- Because non-RPs are not clearly defined under screening conditions, GNN-based performance metrics cannot be transferred to the filtered candidate set and no FDR is reported for the MRN
- Performance is limited by the coverage of the knowledge databases used to source reported and non-reaction pairs

## Evidence

- [discussion] Metrics inapplicable to filtered set: [evidence span withheld — anchor status: not_found]
