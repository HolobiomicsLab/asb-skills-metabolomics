---
name: model-checkpoint-persistence
description: Use when a trained GNN reaction-pair classifier has finished training
  on 14,974 RP / 14,974 non-RP pairs (8:1:1 split) and its checkpoint, prediction
  scores, and evaluation artifacts must be saved for reproducibility.
when_to_use_negative:
- The task is only threshold selection or ROC computation on already-saved scores
  — no checkpoint needs to be written.
- The model architecture or feature pipeline (graph convolutional + fingerprint +
  reaction-class features) has changed, making an existing checkpoint incompatible
  and misleading on reload.
- The classifier is being applied through the full MetDNA3/MrnAnnoAlgo3 annotation
  pipeline rather than being trained/evaluated standalone.
tools:
- name: MetDNA3
  role: Metabolite annotation pipeline whose MRN curation depends on the GNN reaction-pair
    model that this skill checkpoints and preserves
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: PyTorch
  role: Framework in which the GNN is implemented; provides the checkpoint serialization
    API (torch.save) used to persist weights
- name: scikit-learn
  role: Computes the ROC curve and metrics that are persisted alongside the checkpoint
    as the evaluation artifacts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:6336743eedc15c18f61c41cbd20242d9fbfdd44a773e394e5da14c2c2283c43c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# model-checkpoint-persistence

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Persist the trained GNN reaction-pair classifier (built with PyTorch/PyTorch Geometric for MetDNA3's metabolic reaction network curation) as a serialized checkpoint alongside its test-set prediction scores and ROC curve figure, so the reaction-relationship prediction can be reloaded, audited, and reproduced without retraining.

## When to use

Use when a trained GNN reaction-pair classifier has finished training on 14,974 RP / 14,974 non-RP pairs (8:1:1 split) and its checkpoint, prediction scores, and evaluation artifacts must be saved for reproducibility.

## When NOT to use

- The task is only threshold selection or ROC computation on already-saved scores — no checkpoint needs to be written.
- The model architecture or feature pipeline (graph convolutional + fingerprint + reaction-class features) has changed, making an existing checkpoint incompatible and misleading on reload.
- The classifier is being applied through the full MetDNA3/MrnAnnoAlgo3 annotation pipeline rather than being trained/evaluated standalone.

## Inputs

- Trained GNN reaction-pair classifier (graph convolutional features + molecular fingerprint features + reaction-class features, PyTorch/PyTorch Geometric or DGL)
- Held-out test-set prediction scores and labels (n = 2995 reaction pairs)
- Validation-split predictions used for weighted Youden index threshold selection (reported cutoff: 0.7)

## Outputs

- Serialized model checkpoint file (e.g. .pt/.pth state dict with weights and hyperparameters)
- Test-set prediction scores table (reaction pair, predicted score, predicted label)
- ROC curve figure (TPR vs. FPR, analogous to Fig. 1c)

## How to apply

After the GNN reaction-pair classifier converges on the training split (monitored on the validation split), serialize the model weights/optimizer state to a checkpoint file using the framework's native persistence API (e.g. torch.save for PyTorch models). Alongside the checkpoint, write the held-out test-set prediction scores as a structured table (one row per reaction pair with its predicted score and label, n = 2995 per the reported test set) and export the ROC curve figure (true positive rate vs. false positive rate, as in Fig. 1c) computed with scikit-learn. Rationale: the checkpoint decouples model reuse (re-applying the 0.7 weighted-Youden-derived cutoff to new metabolite pairs for MRN curation) from the costly graph/fingerprint feature construction and training procedure, and the persisted scores table and figure allow independent verification of the reported test AUC without access to raw training data.

## Related tools

- **MetDNA3** (Metabolite annotation pipeline whose MRN curation depends on the GNN reaction-pair model that this skill checkpoints and preserves) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **PyTorch** (Framework in which the GNN is implemented; provides the checkpoint serialization API (torch.save) used to persist weights)
- **scikit-learn** (Computes the ROC curve and metrics that are persisted alongside the checkpoint as the evaluation artifacts)

## Examples

```
import torch; torch.save({'state_dict': model.state_dict(), 'threshold': 0.7}, 'gnn_reaction_pair.pt')
```

## Evaluation signals

- Reloaded checkpoint reproduces identical test-set scores on the n = 2995 held-out pairs (deterministic re-inference).
- Persisted prediction scores table contains the expected number of test rows with valid score and label columns matching the reported test split.
- Recomputed ROC/AUC from the saved scores matches the persisted ROC figure (Fig. 1c) and the decision threshold of 0.7 selected by weighted Youden index.
- Checkpoint file loads without architecture mismatch errors into the GNN definition (graph convolutional + fingerprint + reaction-class feature inputs).

## Limitations

- GNN-based direct prediction of reaction pairs may include a considerable number of false positives, so the checkpoint's raw scores are not an FDR-controlled output — the article refrains from reporting an FDR for the MRN due to lack of clearly defined non-RPs under screening conditions.
- The 0.7 threshold was tuned on the validation split; it is inapplicable when applying GNN performance metrics to a filtered candidate set outside the training distribution.
- Full MetDNA3 functionality lives in the webserver; the open-source MrnAnnoAlgo3 module alone does not ship the complete model-serving path, so checkpoints must be paired with the correct library objects.

## Evidence

- [methods] Save the trained model checkpoint, the test-set prediction scores table, and the ROC curve figure.: [evidence span withheld — anchor status: not_found]
- [methods] Sweep the decision threshold on validation predictions and select the threshold that maximizes the weighted Youden index, yielding the reported 0.7 cutoff.: [evidence span withheld — anchor status: not_found]
- [results] The GNN model's reaction-pair prediction performance was evaluated with a receiver operating characteristic (ROC) curve on a held-out testing dataset of n = 2995 reaction pairs (Fig. 1c).: [evidence span withheld — anchor status: not_found]
