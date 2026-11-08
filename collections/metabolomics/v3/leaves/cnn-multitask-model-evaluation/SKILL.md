---
name: cnn-multitask-model-evaluation
description: Use when a trained NMR-based CNN must be validated on a held-out HSQC
  test set by reproducing reported Top-1 accuracy, fingerprint cosine similarity,
  and molecular-weight R-squared metrics.
when_to_use_negative:
- The goal is similarity search / annotation of new query spectra rather than benchmarking
  the model — use DeepSAT's search workflow (cosine > 0.8 threshold against reference
  fingerprints) instead.
- Input spectra are not 1H-13C HSQC (e.g. 1D proton/carbon shifts or MS/MS data) —
  DeepSAT was trained specifically on 143,467 HSQC spectra.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: DeepSAT
  role: Pretrained multi-task CNN whose two HSQC branches are run in inference mode
    to generate fingerprint, molecular weight, and structure-class predictions for
    evaluation
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:2085319daab9b4d5aa04e085e918b7c70df12d987566dc062037981af5d0114a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cnn-multitask-model-evaluation

## Summary

Reproduce/evaluate a multi-task CNN (DeepSAT) that predicts Morgan fingerprints, molecular weight, and NPClassifier structure class from 1H-13C HSQC spectra, by computing Top-1 class accuracy, fingerprint cosine similarity, and molecular-weight R-squared against reported benchmarks.

## When to use

Use when a trained NMR-based CNN must be validated on a held-out HSQC test set by reproducing reported Top-1 accuracy, fingerprint cosine similarity, and molecular-weight R-squared metrics.

## When NOT to use

- The goal is similarity search / annotation of new query spectra rather than benchmarking the model — use DeepSAT's search workflow (cosine > 0.8 threshold against reference fingerprints) instead.
- Input spectra are not 1H-13C HSQC (e.g. 1D proton/carbon shifts or MS/MS data) — DeepSAT was trained specifically on 143,467 HSQC spectra.

## Inputs

- Held-out test set of 1H-13C HSQC spectra (n=3982) with ground-truth NPClassifier structure classes
- Ground-truth Morgan fingerprint bit strings and molecular weights for test compounds
- CC0-licensed pretrained DeepSAT model weights from github.com/mwang87/DeepSAT
- Normal HSQC spectra encoded as 128x128x1 image tensors; multiplicity-edited HSQC encoded as 128x128x2 tensors

## Outputs

- Metrics table (deepsat_evaluation_metrics.csv) with Top-1 class accuracy per input branch
- Average cosine similarity between predicted and ground-truth Morgan fingerprints per branch
- R-squared values for molecular weight prediction per branch

## How to apply

Load the CC0-licensed pretrained DeepSAT weights from github.com/mwang87/DeepSAT and encode the held-out test HSQC spectra as fixed-size image tensors: single-channel 128x128x1 for normal HSQC and two-channel 128x128x2 for multiplicity-edited HSQC. Run multi-task inference through the CNN's prediction heads (Morgan fingerprint bit strings, molecular weight, NPClassifier structure class) for both input branches. Compute Top-1 compound-class accuracy, average cosine similarity between predicted and ground-truth Morgan fingerprints, and R-squared for molecular weight prediction. Accept the evaluation if Top-1 accuracy and cosine scores reproduce the paper's reported values (90.4%/90.8% Top-1; 0.8450 normal / 0.8574 edited cosine; R-squared 0.9183 normal / 0.9336 edited) within reasonable tolerance, and write the metrics table to a CSV (e.g. deepsat_evaluation_metrics.csv).

## Related tools

- **DeepSAT** (Pretrained multi-task CNN whose two HSQC branches are run in inference mode to generate fingerprint, molecular weight, and structure-class predictions for evaluation) — github.com/mwang87/DeepSAT

## Examples

```
peaks = [{'1H': 7.2, '13C': 128.5}, {'1H': 6.8, '13C': 115.3}]; requests.post('https://deepsat.ucsd.edu/api/smart3/search', json=peaks)
```

## Evaluation signals

- Top-1 compound class accuracy reaches ~90.4% (normal HSQC) and ~90.8% (multiplicity-edited HSQC) within tolerance
- Average fingerprint cosine similarity equals ~0.8450 (normal) and ~0.8574 (multiplicity-edited)
- Molecular-weight prediction R-squared equals ~0.9183 (normal) and ~0.9336 (multiplicity-edited)
- Encoded tensors match the expected shapes: 128x128x1 for normal HSQC and 128x128x2 for multiplicity-edited HSQC
- Metrics table is written as deepsat_evaluation_metrics.csv with per-branch columns

## Limitations

- Low-F1 classes exist: diazotetronic acids and derivatives (F1 0.64) and naphthalenes (F1 0.66) classify poorly despite high overall accuracy
- Solvent-transfer robustness is incomplete — over 65% of results were still correctly identified when data were recorded in different solvents, but not all
- Model decisions are only partially interpretable; occlusion sensitivity was applied to a single compound (quercetin) and 'these results do not fully explain the decision-making process of DeepSAT'

## Evidence

- [intro] DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] top1 and cosine results: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [results] cosine scores: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] molecular weight R-squared: "for the normal and multiplicity edited HSQC models the results were 0.9183 and 0.9336, respectively"
- [readme] license statement: "all data, models, and ontology are licensed as [CC0](https://creativecommons.org/share-your-work/public-domain/cc0/)"
