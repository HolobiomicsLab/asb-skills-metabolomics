---
name: morgan-fingerprint-bit-decoding
description: Use when the goal is to turn raw 1H-13C HSQC peak lists (normal or multiplicity-edited)
  into a machine-comparable structural fingerprint (6144-bit Morgan string, radius
  0-2) plus molecular weight and superclass probabilities for library-wide similarity
  searching.
when_to_use_negative:
- Input is already a molecular structure or SMILES from which a Morgan fingerprint
  can be generated directly — DeepSAT's head is for predicting fingerprints from HSQC
  spectra, not for conventional RDKit-style fingerprint computation
- Only 1D proton or carbon NMR shifts are available and no 1H-13C HSQC spectrum exists
  — the skill requires HSQC spectra as the sole spectral input
- The task is quantitative metabolite concentration estimation rather than structure
  annotation/similarity search — the model's regression head predicts molecular weight,
  not abundance
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3316
tools:
- name: DeepSAT
  role: Pretrained convolutional neural network multi-task model whose sigmoid/softmax/MAPE
    output heads are decoded into the 6144-bit Morgan fingerprint, molecular weight,
    and NPClassifier class probabilities
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:8cfa3d2d674f26f260fb6a007d391a2fed8f36369c33cbffcd89230fe8517231
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# morgan-fingerprint-bit-decoding

## Summary

Decode DeepSAT's multi-task CNN output heads on a 1H-13C HSQC spectrum into a 6144-bit radius 0-2 Morgan fingerprint bit string alongside a rounded molecular weight and NPClassifier class probabilities, which are then compared to known structures by cosine similarity.

## When to use

Use when the goal is to turn raw 1H-13C HSQC peak lists (normal or multiplicity-edited) into a machine-comparable structural fingerprint (6144-bit Morgan string, radius 0-2) plus molecular weight and superclass probabilities for library-wide similarity searching.

## When NOT to use

- Input is already a molecular structure or SMILES from which a Morgan fingerprint can be generated directly — DeepSAT's head is for predicting fingerprints from HSQC spectra, not for conventional RDKit-style fingerprint computation
- Only 1D proton or carbon NMR shifts are available and no 1H-13C HSQC spectrum exists — the skill requires HSQC spectra as the sole spectral input
- The task is quantitative metabolite concentration estimation rather than structure annotation/similarity search — the model's regression head predicts molecular weight, not abundance

## Inputs

- 1H-13C HSQC spectrum peak list (JSON list of dicts with 1H and 13C shift headers, as accepted by the DeepSAT API)
- 1H-13C multiplicity-edited HSQC spectra (alternatively supported input mode)
- Rendered HSQC spectral image representation
- Pretrained DeepSAT CNN multi-task model

## Outputs

- 6144-bit Morgan fingerprint bit string per input spectrum (radius 0-2)
- Rounded molecular weight prediction per spectrum
- NPClassifier superclass probability vector per spectrum
- Structured per-spectrum record row in predictions.csv
- Cosine similarity scores against fingerprints of known small molecules

## How to apply

Load the input 1H-13C HSQC spectrum (peak list with 1H and 13C shift headers, or a rendered spectral image) and run it through DeepSAT's pretrained convolutional neural network multi-task model, which was trained on 143,467 reference HSQC spectra. Decode the three task-specific heads separately: apply sigmoid activation with binary cross-entropy to obtain the 6144-bit Morgan fingerprint bit vector (radius 0-2), softmax with sparse categorical cross-entropy for the NPClassifier superclass probability vector, and the mean-absolute-percentage-error regression head for molecular weight (round to an integer value). Write one structured record per spectrum, appending the fingerprint bits, molecular weight, and class probabilities as rows in predictions.csv. Validate that predictions.csv exists with one record per input spectrum, all superclass probabilities fall within [0, 1] and sum per class head, and each fingerprint bit vector has length 6144. Structural similarity to known molecules is then assessed by cosine score over the predicted vs known Morgan fingerprints, with the article's similarity/annotation threshold set at cosine score > 0.8.

## Related tools

- **DeepSAT** (Pretrained convolutional neural network multi-task model whose sigmoid/softmax/MAPE output heads are decoded into the 6144-bit Morgan fingerprint, molecular weight, and NPClassifier class probabilities) — github.com/mwang87/DeepSAT

## Examples

```
POST peaks=[{'1H': 7.24, '13C': 128.4}, ...] to /api/smart3/search; decode sigmoid/softmax/MAPE heads into a 6144-bit Morgan fingerprint (radius 0-2), rounded MW, and NPClassifier probabilities.
```

## Evaluation signals

- predictions.csv exists with exactly one record per input HSQC spectrum
- Each fingerprint bit vector is length 6144 (radius 0-2 Morgan representation)
- All NPClassifier superclass probabilities are in [0, 1] and sum per class head
- Molecular weight output is a rounded numerical value from the regression head
- Predicted vs ground-truth fingerprints achieve average cosine score ~0.8450 (normal HSQC) or ~0.8574 (multiplicity-edited HSQC)

## Limitations

- Interpretability is incomplete: occlusion sensitivity analysis was limited to a single compound (quercetin) and 'these results do not fully explain the decision-making process of DeepSAT'
- Solvent transfer is imperfect: over 65% of results were correctly identified when data were recorded in a different solvent, but not all
- Some structure classes show poor F1-scores (e.g. diazotetronic acids and derivatives 0.64, naphthalenes 0.66), so class-level predictions for such subclasses should be treated cautiously
- Peak-match retrieval baselines require tight thresholds (0.5 ppm carbon, 0.05 ppm proton) that are distinct from the fingerprint cosine threshold

## Evidence

- [results] sigmoid with binary cross-entropy for fingerprint prediction: [evidence span withheld — anchor status: not_found]
- [intro] 143,467 HSQC spectra training corpus: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] cosine similarity threshold of 0.8 for structural similarity: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
- [results] average cosine score benchmarks: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [readme] programmatic API for peak input: "You can put in your peaks as a json list of dicts, with 1H,13C as headers"
