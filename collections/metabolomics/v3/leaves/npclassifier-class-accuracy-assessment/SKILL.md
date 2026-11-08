---
name: npclassifier-class-accuracy-assessment
description: Use when reproducing or validating DeepSAT's reported structure-class
  performance (Top-1 accuracy 90.4% normal / 90.8% edited HSQC) on HSQC test spectra
  with ground-truth NPClassifier classes.
when_to_use_negative:
- The task is structure retrieval/annotation-rate measurement against a reference
  library rather than accuracy assessment of the classification head — use the library-search
  workflow with cosine > 0.8 thresholds instead
- Input spectra are not 1H-13C HSQC (e.g. 1D proton/carbon NMR or MS/MS spectra),
  which DeepSAT does not accept
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepSAT
  role: Multi-task CNN whose NPClassifier structure-class head, Morgan fingerprint
    head, and molecular-weight head are evaluated on encoded 128x128 HSQC tensors
  repo: github.com/mwang87/DeepSAT
- name: NPClassifier
  role: Structure-class ontology supplying the compound-class labels whose Top-1 prediction
    accuracy is computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:d52c6aea884812546bf10baf0c7dbc35d9cc45c919aef9d1ba0b3243e1aff791
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - NPClassifier
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# NPClassifier compound class accuracy assessment for DeepSAT

## Summary

Benchmark the NPClassifier structure-class prediction head of the DeepSAT two-branch CNN (normal vs multiplicity-edited 1H-13C HSQC) on a held-out HSQC test set, reporting Top-1 class accuracy alongside fingerprint cosine similarity and molecular-weight R-squared as sanity checks.

## When to use

Use when reproducing or validating DeepSAT's reported structure-class performance (Top-1 accuracy 90.4% normal / 90.8% edited HSQC) on HSQC test spectra with ground-truth NPClassifier classes.

## When NOT to use

- The task is structure retrieval/annotation-rate measurement against a reference library rather than accuracy assessment of the classification head — use the library-search workflow with cosine > 0.8 thresholds instead
- Input spectra are not 1H-13C HSQC (e.g. 1D proton/carbon NMR or MS/MS spectra), which DeepSAT does not accept

## Inputs

- Held-out test set of 1H-13C HSQC spectra (n=3982) as peak lists with 1H and 13C shifts
- Multiplicity-edited HSQC peak lists with multiplicity information
- Ground-truth NPClassifier structure classes per test compound
- Pretrained DeepSAT model weights (CC0-licensed) from github.com/mwang87/DeepSAT

## Outputs

- deepsat_evaluation_metrics.csv with Top-1 compound class accuracy per input branch
- Average cosine similarity between predicted and ground-truth Morgan fingerprints (0.8450 normal / 0.8574 edited expected)
- Molecular-weight prediction R-squared (0.9183 normal / 0.9336 edited expected)

## How to apply

Load the CC0-licensed pretrained DeepSAT weights from github.com/mwang87/DeepSAT (code under MIT). Encode held-out test HSQC spectra as fixed-size image tensors: single-channel 128x128x1 for normal HSQC and two-channel 128x128x2 for multiplicity-edited HSQC. Run the multi-task CNN's prediction heads for Morgan fingerprint bit strings, molecular weight, and NPClassifier structure class on each branch. Compute Top-1 accuracy of predicted compound classes, plus average cosine similarity between predicted and ground-truth Morgan fingerprints and molecular-weight R-squared as corroborating metrics. Accept the evaluation if Top-1 accuracy lands near 90.4% (normal) / 90.8% (edited) and average cosine near 0.8450 (normal) / 0.8574 (edited) within reasonable tolerance, then write the metrics table to deepsat_evaluation_metrics.csv.

## Related tools

- **DeepSAT** (Multi-task CNN whose NPClassifier structure-class head, Morgan fingerprint head, and molecular-weight head are evaluated on encoded 128x128 HSQC tensors) — github.com/mwang87/DeepSAT
- **NPClassifier** (Structure-class ontology supplying the compound-class labels whose Top-1 prediction accuracy is computed)

## Examples

```
curl -X POST 'https://deepsat.ucsd.edu/api/smart3/search' -H 'Content-Type: application/json' -d '[{"1H": 1.25, "13C": 30.1}]'
```

## Evaluation signals

- Computed Top-1 compound class accuracy ≈ 90.4% for the normal HSQC branch and ≈ 90.8% for the multiplicity-edited branch within reasonable tolerance
- Average cosine similarity between predicted and ground-truth Morgan fingerprints ≈ 0.8450 (normal) and ≈ 0.8574 (edited)
- Molecular-weight R-squared ≈ 0.9183 (normal) and ≈ 0.9336 (edited) confirms the multi-task heads behave consistently
- Input tensors have the required fixed shapes (128x128x1 normal, 128x128x2 edited) before inference
- Metrics table deepsat_evaluation_metrics.csv is written with one accuracy row per input branch

## Limitations

- Performance degrades for specific classes: diazotetronic acids and derivatives (F1-score 0.64) and naphthalenes (F1-score 0.66) showed low values
- Model is not fully solvent-transfer robust: over 65% of results were still correctly identified even though data were recorded in different solvents, i.e. a non-trivial fraction of queries fail
- The paper's discussion defers precision/recall/F1 detail to supplementary material, so per-class metrics may not be fully reproducible from the main text

## Evidence

- [results] Top-1 accuracy claim: "the Top-1 accuracy of normal and Edited HSQC models were 90.4% and 90.8%, respectively"
- [results] Cosine similarity metrics: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] Molecular weight R-squared: "for the normal and multiplicity edited HSQC models the results were 0.9183 and 0.9336, respectively"
- [intro] Training scope: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [readme] License terms: "The license as included for the software is MIT. Additionally, all data, models, and ontology are licensed as [CC0]"
