---
name: molecular-fingerprint-representation
description: Use when building or reusing a GNN classifier that predicts reaction
  relationships between metabolite pairs from molecular structures, requiring fingerprint
  and reaction-class features for metabolic reaction network curation.
when_to_use_negative:
- 'The goal is FDR estimation for the filtered MRN candidate set: non-RPs are not
  clearly defined under screening conditions, so GNN-based performance metrics do
  not transfer to the filtered candidate set'
- The task is annotating experimental MS1/MS2 features in a sample — fingerprint-based
  RP prediction feeds knowledge-driven networking but does not replace MS2 similarity
  constraints or MS1 m/z matching
- Metabolite structures are unavailable or ambiguous, so molecular graphs and fingerprints
  cannot be reliably encoded
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: PyTorch
  role: Framework for building and training the GNN reaction-pair classifier on graph-based
    molecular representations, fingerprints, and reaction-class features
- name: scikit-learn
  role: Computing ROC curve, AUC, and the weighted Youden-index threshold sweep (0.7
    cutoff) on held-out test predictions
- name: MetDNA3
  role: 'Downstream consumer: the GNN-predicted reaction relationships curate the
    comprehensive metabolic reaction network used for metabolite annotation'
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:1e3f1ad3be267d35df86e39060df76b7b00b40baa04a2794c0b87fbc767f6d5d
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-fingerprint-representation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes metabolite structures as graph-convolutional node features, molecular fingerprints, and reaction-class features to train a GNN reaction-pair (RP) classifier used to curate a comprehensive metabolic reaction network (MRN) for MetDNA3 annotation.

## When to use

Use when building or reusing a GNN classifier that predicts reaction relationships between metabolite pairs from molecular structures, requiring fingerprint and reaction-class features for metabolic reaction network curation.

## When NOT to use

- The goal is FDR estimation for the filtered MRN candidate set: non-RPs are not clearly defined under screening conditions, so GNN-based performance metrics do not transfer to the filtered candidate set
- The task is annotating experimental MS1/MS2 features in a sample — fingerprint-based RP prediction feeds knowledge-driven networking but does not replace MS2 similarity constraints or MS1 m/z matching
- Metabolite structures are unavailable or ambiguous, so molecular graphs and fingerprints cannot be reliably encoded

## Inputs

- Curated reaction-pair training data (14,974 RPs and 14,974 non-RPs, split 8:1:1)
- Molecular structures of metabolites for graph-convolutional and fingerprint encoding
- Reaction-class annotations for each metabolite pair

## Outputs

- Trained GNN reaction-pair classifier model checkpoint
- Test-set prediction scores table for n = 2995 reaction pairs
- ROC curve and area under the curve (Fig. 1c-style plot, TPR vs. FPR)
- Reaction-pair predictions at the 0.7 decision threshold for MRN curation

## How to apply

Assemble curated reaction-pair data (14,974 RPs and 14,974 non-RPs) and split 8:1:1 into training, validation, and test sets; the held-out test set combines to n = 2995 reaction pairs. Encode each metabolite as graph-based molecular representations (graph convolutional features), molecular fingerprint features, and reaction-class features, then train the GNN RP classifier (e.g., in PyTorch) while monitoring validation performance for early stopping. Sweep the decision threshold on validation predictions and select the value maximizing the weighted Youden index, which yields the reported 0.7 cutoff. Apply the trained model to the test set to compute scores and labels, then generate the ROC curve and AUC (true positive rate vs. false positive rate) with scikit-learn, reproducing the Fig. 1c evaluation. Save the model checkpoint, test-set prediction scores table, and ROC figure for reuse in MRN curation.

## Related tools

- **PyTorch** (Framework for building and training the GNN reaction-pair classifier on graph-based molecular representations, fingerprints, and reaction-class features)
- **scikit-learn** (Computing ROC curve, AUC, and the weighted Youden-index threshold sweep (0.7 cutoff) on held-out test predictions)
- **MetDNA3** (Downstream consumer: the GNN-predicted reaction relationships curate the comprehensive metabolic reaction network used for metabolite annotation) — https://github.com/ZhuMetLab/MrnAnnoAlgo3

## Evaluation signals

- The 8:1:1 split yields a held-out test set of n = 2995 reaction pairs, matching the reported evaluation set
- Threshold sweep on validation predictions reproduces the reported 0.7 cutoff that maximizes the weighted Youden index
- The computed ROC curve and AUC are plotted as TPR vs. FPR on test predictions, comparable to Fig. 1c
- Model checkpoint, test-set prediction scores table, and ROC figure are all saved as reusable artifacts
- Learned RP predictions feed the curated MRN whose enhanced coverage and connectivity support the two-layer networking strategy

## Limitations

- GNN-based direct prediction of reaction pairs may include a considerable number of false positives
- No FDR is reported for the curated MRN because clearly defined non-RPs are lacking under screening conditions and GNN performance metrics are inapplicable to the filtered candidate set
- Metabolites not covered by knowledge networks remain unannotated; the GNN prediction supplements but does not guarantee full reaction-network coverage

## Evidence
