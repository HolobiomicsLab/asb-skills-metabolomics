---
name: threshold-optimization-weighted-youden-index
description: Use when a binary classifier produces continuous scores (e.g. GNN reaction-pair
  predictions in MetDNA3) and you need a data-driven decision cutoff rather than the
  default 0.5, especially when class balance or asymmetric costs make a single operating
  point critical.
when_to_use_negative:
- The classifier outputs hard binary labels rather than continuous scores — no threshold
  sweep is meaningful.
- There is no labeled validation set to tune against (e.g. the curated MRN candidate
  set itself lacks clearly defined non-RPs, so the article explicitly refrains from
  reporting an FDR there).
- You need calibration of the score's probability meaning rather than a single operating-point
  cutoff (threshold optimization does not calibrate probabilities).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MetDNA3
  role: Embeds the GNN reaction-pair model whose threshold is optimized via weighted
    Youden index; the resulting 0.7 cutoff governs MRN candidate screening
  repo: github.com/ZhuMetLab/MrnAnnoAlgo3
- name: scikit-learn
  role: Compute ROC curve/AUC and implement the weighted-Youden threshold sweep on
    validation scores
- name: PyTorch
  role: Framework for training the GNN that supplies the prediction scores being thresholded
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:412fc31887e36f6479017b6e2b9e1d111e2b3c71145c538d40e3a36231982dcc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA3
  - PyTorch
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Threshold-optimization-weighted-Youden-index

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimizes the decision threshold of a binary classifier (here, a GNN reaction-pair predictor) by sweeping candidate cutoffs on a validation set and selecting the threshold that maximizes the weighted Youden index, as done in MetDNA3 for its metabolic reaction network (MRN) model. It converts raw prediction scores into a fixed operating cutoff (reported as 0.7) used downstream for filtering candidate reaction pairs.

## When to use

Use when a binary classifier produces continuous scores (e.g. GNN reaction-pair predictions in MetDNA3) and you need a data-driven decision cutoff rather than the default 0.5, especially when class balance or asymmetric costs make a single operating point critical.

## When NOT to use

- The classifier outputs hard binary labels rather than continuous scores — no threshold sweep is meaningful.
- There is no labeled validation set to tune against (e.g. the curated MRN candidate set itself lacks clearly defined non-RPs, so the article explicitly refrains from reporting an FDR there).
- You need calibration of the score's probability meaning rather than a single operating-point cutoff (threshold optimization does not calibrate probabilities).

## Inputs

- Trained binary classifier producing continuous prediction scores (e.g. GNN reaction-pair model with graph, fingerprint, and reaction-class features)
- Held-out validation-split prediction scores with true labels (from the 14,974 RPs and 14,974 non-RPs training pool)
- Held-out test-set prediction scores with labels (n = 2995)
- Class weighting scheme or class counts for the weighted Youden computation

## Outputs

- Optimized decision threshold (0.7 in MetDNA3)
- Weighted Youden index as a function of threshold (sweep table)
- ROC curve and AUC on the test set (Fig. 1c)
- Frozen threshold file/checkpoint for downstream MRN candidate filtering

## How to apply

After training the binary classifier (e.g. a GNN trained on reported and non-reaction pairs from knowledge databases, with graph convolutional, molecular fingerprint, and reaction-class features), run inference on the held-out validation split and collect prediction scores. Sweep candidate thresholds across the score range and, at each threshold, compute the true positive rate and false positive rate; evaluate the weighted Youden index (a class-weighted variant of sensitivity + specificity − 1) to account for class imbalance between reaction pairs (RPs) and non-reaction pairs. Select the threshold maximizing the weighted Youden index — MetDNA3's data and knowledge-driven workflow yielded a cutoff of 0.7 on its 14,974 RPs and 14,974 non-RPs 8:1:1 split. Apply the frozen threshold to the held-out test set (n = 2995) to generate the ROC curve and AUC (Fig. 1c), and save the threshold alongside the model checkpoint and test-score table so downstream MRN filtering uses a reproducible cutoff.

## Related tools

- **MetDNA3** (Embeds the GNN reaction-pair model whose threshold is optimized via weighted Youden index; the resulting 0.7 cutoff governs MRN candidate screening) — github.com/ZhuMetLab/MrnAnnoAlgo3
- **scikit-learn** (Compute ROC curve/AUC and implement the weighted-Youden threshold sweep on validation scores)
- **PyTorch** (Framework for training the GNN that supplies the prediction scores being thresholded)

## Examples

```
from sklearn.metrics import roc_curve; fpr, tpr, thr = roc_curve(y_val, scores); youden = w_pos*tpr - w_neg*fpr; cutoff = thr[youden.argmax()]
```

## Evaluation signals

- The swept threshold that maximizes the weighted Youden index equals the reported 0.7 cutoff when applied to MetDNA3's validation split.
- Applying the frozen threshold to the n = 2995 test set reproduces the ROC/AUC reported in Fig. 1c.
- The threshold sweep is performed on validation scores only — no test-set labels leak into cutoff selection.
- The saved threshold remains constant when re-run on identical model checkpoints (deterministic, reproducible cutoff).
- Downstream MRN filtering consumes exactly the stored cutoff rather than re-deriving it per run.

## Limitations

- The GNN-based direct prediction of reaction pairs may include a considerable number of false positives; threshold optimization on the GNN alone does not eliminate these.
- No FDR can be reported for the final curated MRN because clearly defined non-RPs are lacking under the screening conditions, and GNN performance metrics are inapplicable to the filtered candidate set.
- Cross-tool concordance in downstream validation may overestimate performance due to lack of chemical standards.
- Performance metrics were computed on a synthetic/curated RP-vs-non-RP split; real biological reaction spaces may not match this class structure.

## Evidence

- [methods] Sweep the decision threshold on validation predictions and select the threshold that maximizes the weighted Youden index, yielding the reported 0.7 cutoff.: [evidence span withheld — anchor status: not_found]
- [results] The GNN model's reaction-pair prediction performance was evaluated with a ROC curve on a held-out testing dataset of n = 2995 reaction pairs (Fig. 1c).: [evidence span withheld — anchor status: not_found]
