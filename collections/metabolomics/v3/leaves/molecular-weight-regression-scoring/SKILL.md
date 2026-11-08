---
name: molecular-weight-regression-scoring
description: Use when a 1H-13C HSQC spectrum (normal or multiplicity-edited) must
  yield a predicted molecular weight via DeepSAT's pretrained CNN regression head,
  and the result must be validated against the paper's R-squared benchmarks.
when_to_use_negative:
- The input is not a 1H-13C HSQC spectrum (e.g., MS/MS fragmentation spectra, 1D proton/carbon
  NMR) — DeepSAT consumes only HSQC data
- The goal is compound class or fingerprint prediction scoring (use class-accuracy
  or cosine-similarity skills) rather than molecular weight regression
- No trained model weights or matching TensorFlow Serving model metadata are available;
  the README warns that model input names must match between the serving endpoint
  and the code
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: DeepSAT
  role: Pretrained multi-task CNN whose molecular weight regression head (MAPE-trained)
    is run on encoded HSQC tensors to produce weight predictions
  repo: https://github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:f00d1d224fc5634249ee1e31ebd895102d2977491fed407b9a7340e0513e28d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-weight-regression-scoring

## Summary

Runs DeepSAT's multi-task CNN molecular weight regression head on 1H-13C HSQC spectra (128x128x1 normal or 128x128x2 multiplicity-edited tensors) and scores predictions with mean absolute percentage error (MAPE) and R-squared against reported values (R² = 0.9183 normal / 0.9336 edited).

## When to use

Use when a 1H-13C HSQC spectrum (normal or multiplicity-edited) must yield a predicted molecular weight via DeepSAT's pretrained CNN regression head, and the result must be validated against the paper's R-squared benchmarks.

## When NOT to use

- The input is not a 1H-13C HSQC spectrum (e.g., MS/MS fragmentation spectra, 1D proton/carbon NMR) — DeepSAT consumes only HSQC data
- The goal is compound class or fingerprint prediction scoring (use class-accuracy or cosine-similarity skills) rather than molecular weight regression
- No trained model weights or matching TensorFlow Serving model metadata are available; the README warns that model input names must match between the serving endpoint and the code

## Inputs

- 1H-13C HSQC spectra as peak lists (JSON list of dicts with 1H, 13C headers) or rendered spectral images
- Encoded image tensors: 128x128x1 (normal HSQC) or 128x128x2 (multiplicity-edited HSQC)
- Ground-truth molecular weights for held-out test compounds (n=3982) for R-squared scoring

## Outputs

- Rounded molecular weight prediction per input spectrum (one record in predictions.csv)
- R-squared score of predicted vs. ground-truth molecular weights, compared to 0.9183 (normal) / 0.9336 (multiplicity-edited)

## How to apply

Load the CC0-licensed pretrained DeepSAT weights from github.com/mwang87/DeepSAT and encode each input spectrum as a fixed-size image tensor: single-channel 128x128x1 for normal HSQC or two-channel 128x128x2 for multiplicity-edited HSQC. Run multi-task inference, where the molecular weight head is one of three heads trained with distinct losses — mean absolute percentage error for weight regression, sigmoid + binary cross-entropy for the 6144-bit Morgan fingerprint, and softmax + sparse categorical cross-entropy for NPClassifier class prediction. Decode the regression head into a rounded molecular weight value per spectrum and record it in predictions.csv alongside the other heads' outputs. Score quality by computing R-squared between predicted and ground-truth weights; the paper reports 0.9183 for the normal HSQC model and 0.9336 for the multiplicity-edited model, so computed values should fall near those figures within reasonable tolerance.

## Related tools

- **DeepSAT** (Pretrained multi-task CNN whose molecular weight regression head (MAPE-trained) is run on encoded HSQC tensors to produce weight predictions) — https://github.com/mwang87/DeepSAT

## Examples

```
curl -X POST https://deepsat.ucsd.edu/api/smart3/search -d '[{"1H": 3.12, "13C": 52.4}]'
```

## Evaluation signals

- Computed R-squared between predicted and ground-truth molecular weights is close to 0.9183 (normal HSQC) or 0.9336 (multiplicity-edited HSQC) within reasonable tolerance
- predictions.csv contains one record per input spectrum with a decoded, rounded molecular weight value
- Molecular weight outputs are plausible non-negative real values consistent with the regression (MAPE) head, not probabilities or bit vectors
- The other parallel heads validate simultaneously: fingerprint bit vectors of length 6144 and superclass probabilities in [0, 1], with average fingerprint cosine near 0.8450/0.8574

## Limitations

- Molecular weight regression is only one of three heads; DeepSAT's overall utility depends on the joint fingerprint/class/weight output, and head-specific errors can propagate to downstream database search
- Solvent-transfer robustness is incomplete — over 65% (not all) of results were correctly identified when data were recorded in different solvents
- TensorFlow Serving model metadata input names must match the deployed model; if model input names change, the code must be updated (README)
- The evaluation is tied to the specific 143,467-spectrum training corpus and held-out test set (n=3982); performance on out-of-distribution spectra is not characterized

## Evidence

- [results] R-squared for molecular weight prediction was 0.9183 (normal HSQC) and 0.9336 (multiplicity edited HSQC): [evidence span withheld — anchor status: not_found]
- [other] mean absolute percentage error for molecular weight regression: [evidence span withheld — anchor status: not_found]
- [intro] DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [readme] You can put in your peaks as a json list of dicts, with 1H,13C as headers.: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
