---
name: reaction-pair-classification
description: Use when you must curate or expand a metabolic reaction network by predicting
  RP vs non-RP relationships between metabolites, and you have labeled reaction pairs
  from knowledge databases plus a defined held-out test set for ROC evaluation.
when_to_use_negative:
- Do not apply GNN-based performance metrics (e.g., ROC/FDR) to the filtered candidate
  MRN after screening — the paper explicitly refrains from reporting an MRN FDR because
  non-RPs under screening conditions are not clearly defined.
- Do not use this skill when the goal is downstream annotation propagation or metabolite
  annotation itself; this skill only curates the reaction relationships that feed
  the MRN layer.
- Do not use when no labeled non-RP (negative) examples can be defined for the reaction
  space of interest.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3474
tools:
- name: MetDNA3
  role: Pipeline whose metabolic reaction network curation embeds the GNN reaction-pair
    predictions; algorithm code available as MrnAnnoAlgo3
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: PyTorch
  role: Deep-learning framework for building and training the graph neural network
    classifier (e.g., via PyTorch Geometric / DGL)
- name: scikit-learn
  role: Computing the ROC curve, AUC, and weighted Youden-index threshold sweep on
    validation and test predictions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:e17d6d8840bc711a0c34cf1e5b2d7bd750b4e9f63dec2b2500aa6934ec79c410
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
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# reaction-pair-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a graph neural network (GNN) classifier on curated reaction-pair (RP) and non-reaction-pair (non-RP) examples to predict reaction relationships between metabolites, then evaluate with an ROC curve on a held-out test set, for curating a comprehensive metabolic reaction network (MRN) as done in MetDNA3.

## When to use

Use when you must curate or expand a metabolic reaction network by predicting RP vs non-RP relationships between metabolites, and you have labeled reaction pairs from knowledge databases plus a defined held-out test set for ROC evaluation.

## When NOT to use

- Do not apply GNN-based performance metrics (e.g., ROC/FDR) to the filtered candidate MRN after screening — the paper explicitly refrains from reporting an MRN FDR because non-RPs under screening conditions are not clearly defined.
- Do not use this skill when the goal is downstream annotation propagation or metabolite annotation itself; this skill only curates the reaction relationships that feed the MRN layer.
- Do not use when no labeled non-RP (negative) examples can be defined for the reaction space of interest.

## Inputs

- Curated reaction-pair training data: 14,974 RPs and 14,974 non-RPs sampled from knowledge databases
- Molecular structures of metabolite candidates (for graph-convolutional and fingerprint features)
- Reaction-class annotations for the paired metabolites

## Outputs

- Trained GNN reaction-pair classifier checkpoint
- Test-set prediction score table (n = 2995 pairs with scores and labels)
- ROC curve figure (true-positive rate vs false-positive rate) and AUC value
- Selected decision threshold (0.7 via weighted Youden index)

## How to apply

Load the curated reaction-pair data containing 14,974 RPs and 14,974 non-RPs and split them 8:1:1 into training, validation, and test sets. Build three input feature families in a GNN framework (e.g., PyTorch Geometric or DGL): graph-convolutional molecular representations, molecular fingerprint features, and reaction-class features. Train the reaction-pair classifier on the training split while monitoring the validation split for early stopping. Sweep the decision threshold on validation predictions and select the one maximizing the weighted Youden index — this yields the reported 0.7 cutoff. Apply the trained model to the held-out test set of n = 2995 reaction pairs, compute the ROC curve and AUC on the prediction scores, and plot true-positive vs false-positive rate as in Fig. 1c. Save the model checkpoint, the test-set prediction-score table, and the ROC figure.

## Related tools

- **MetDNA3** (Pipeline whose metabolic reaction network curation embeds the GNN reaction-pair predictions; algorithm code available as MrnAnnoAlgo3) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **PyTorch** (Deep-learning framework for building and training the graph neural network classifier (e.g., via PyTorch Geometric / DGL))
- **scikit-learn** (Computing the ROC curve, AUC, and weighted Youden-index threshold sweep on validation and test predictions)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, thr = roc_curve(y_test, gnn_scores); print('AUC', roc_auc_score(y_test, gnn_scores))
```

## Evaluation signals

- Split invariant: exactly 14,974 RPs and 14,974 non-RPs divided 8:1:1 into training, validation, and test sets.
- Test set size check: the held-out test set must contain n = 2995 reaction pairs for the Fig. 1c ROC evaluation.
- Threshold check: the decision threshold swept on validation predictions should be 0.7, the value that maximizes the weighted Youden index.
- Output artifact check: a saved model checkpoint, a test-set prediction-score table, and an ROC plot (TPR vs FPR) with an AUC value must all exist.
- Reproducibility check: rerunning the split-training-evaluate workflow should reproduce the reported test-set ROC curve on the n = 2995 pairs.

## Limitations

- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, which motivated a two-step pre-screening strategy for the MRN.
- No FDR is reported for the curated MRN: non-RPs under screening conditions are not clearly defined, and GNN performance metrics are incompatible with the filtered candidate set.
- Non-RP negatives are artificial constructs drawn from knowledge databases, so ROC performance may not translate to real screening conditions.

## Evidence

- [results] ROC evaluation on held-out test set: [evidence span withheld — anchor status: not_found]
- [intro] GNN curation of the MRN: [evidence span withheld — anchor status: not_found]
