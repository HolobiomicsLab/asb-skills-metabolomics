---
name: hsqc-spectrum-input-preparation
description: Reference for formatting 1H-13C HSQC peak lists or spectral images as
  DeepSAT inputs, including multiplicity-edited HSQC handling, before running the
  pretrained multi-task CNN for structure annotation.
when_to_use_negative:
- The input is MS/MS fragmentation data or a genome sequence — DeepSAT is trained
  only on 1H-13C HSQC spectra and GNPS/AntiSMASH-style inputs are out of scope.
- The query requires authenticated reference-spectrum matching with tight shift tolerances
  (e.g. 0.5 ppm carbon / 0.05 ppm proton library search) rather than fingerprint-based
  similarity.
- Only a 1D proton or carbon NMR spectrum is available — the CNN's input representation
  is specifically the 2D HSQC correlation spectrum.
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3336
tools:
- name: DeepSAT
  role: Pretrained multi-task CNN that consumes the prepared HSQC input and emits
    fingerprint, molecular weight, and class predictions; hosted at deepsat.ucsd.edu
    with the /api/smart3/search API and TensorFlow Serving /model/metadata endpoint.
  repo: https://github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:45f6f798cc0dce6c9aa2fd3f7d0c69a7ad236f1639842f2ea292e765b4d9735c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mwang87/DeepSAT
  tools:
  - DeepSAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hsqc-spectrum-input-preparation

## Summary

Prepare 1H-13C HSQC (or multiplicity-edited HSQC) spectral inputs so DeepSAT can decode them into 6144-bit Morgan fingerprints, molecular weights, and NPClassifier class probabilities via its CNN multi-task model.

## When to use

Reference for formatting 1H-13C HSQC peak lists or spectral images as DeepSAT inputs, including multiplicity-edited HSQC handling, before running the pretrained multi-task CNN for structure annotation.

## When NOT to use

- The input is MS/MS fragmentation data or a genome sequence — DeepSAT is trained only on 1H-13C HSQC spectra and GNPS/AntiSMASH-style inputs are out of scope.
- The query requires authenticated reference-spectrum matching with tight shift tolerances (e.g. 0.5 ppm carbon / 0.05 ppm proton library search) rather than fingerprint-based similarity.
- Only a 1D proton or carbon NMR spectrum is available — the CNN's input representation is specifically the 2D HSQC correlation spectrum.

## Inputs

- 1H-13C HSQC spectrum (normal mode), as a peak list of {1H, 13C} shift pairs in ppm
- 1H-13C multiplicity-edited HSQC spectrum as an alternative input variant
- JSON list of dicts with 1H and 13C headers for the /api/smart3/search API
- Rendered HSQC spectral image for the image-based CNN input path

## Outputs

- 6144-bit Morgan fingerprint bit string (radius 0-2) per input spectrum
- Predicted molecular weight per input spectrum
- NPClassifier superclass probability vector per input spectrum
- Ranked list of similar structures via cosine comparison against known Morgan fingerprints

## How to apply

Collect each input as a 1H-13C HSQC spectrum, either as a peak list or a rendered spectral image, and keep normal and multiplicity-edited variants distinct because DeepSAT provides separate pretrained models for both (trained on 143,467 HSQC spectra). Represent each spectrum with its proton and carbon chemical shift coordinates so the CNN can learn the spectral features that populate the Morgan fingerprint bit string (radius 0-2, 6144 bits) and the NPClassifier superclass outputs. For programmatic use, pass the peaks as a JSON list of dicts with 1H and 13C headers to the DeepSAT /api/smart3/search endpoint, or invoke the TensorFlow Serving model after confirming the input tensor names via /model/metadata. Solvent differences degrade performance, but the article reports over 65% of structures were still correctly identified across different solvents, so cross-solvent inputs are tolerable though not ideal. Feed the prepared inputs directly into the CNN rather than a chemical-shift search; the paper shows library-search baselines (thresholds 0.5 ppm carbon / 0.05 ppm proton) achieved under 3% correct identification while DeepSAT reached a fingerprint cosine score of 0.8450 (normal HSQC) and 0.8574 (multiplicity-edited HSQC).

## Related tools

- **DeepSAT** (Pretrained multi-task CNN that consumes the prepared HSQC input and emits fingerprint, molecular weight, and class predictions; hosted at deepsat.ucsd.edu with the /api/smart3/search API and TensorFlow Serving /model/metadata endpoint.) — https://github.com/mwang87/DeepSAT

## Examples

```
curl -X POST deepsat.ucsd.edu/api/smart3/search -H 'Content-Type: application/json' -d '[{"1H": 7.2, "13C": 128.5}, {"1H": 6.9, "13C": 116.3}]'
```

## Evaluation signals

- Fingerprint bit vectors returned per input have length 6144 and each entry is 0 or 1.
- NPClassifier superclass probabilities fall within [0, 1] and the per-class head sums to 1 (softmax outputs).
- Predicted vs ground-truth fingerprint cosine score is near the reported averages (0.8450 normal HSQC, 0.8574 multiplicity-edited HSQC); treat cosine > 0.8 as a 'similar' annotation threshold.
- Molecular weight prediction tracks the reported R-squared (0.9183 normal, 0.9336 multiplicity-edited); a large divergence indicates malformed peak input.
- For the API path, a successful /model/metadata response confirms the model input names match the peak JSON schema (1H, 13C headers).

## Limitations

- Solvent transfer is imperfect: fewer than all results were correctly identified across different solvents (over 65% correct), so inputs recorded in a solvent other than the training distribution may degrade.
- The model does not fully explain its decision-making; occlusion sensitivity interpretability was only demonstrated on a single compound (quercetin).
- Classification is weak for specific classes such as diazotetronic acids and derivatives (F1 0.64) and naphthalenes (F1 0.66).
- Absolute precision/recall/F1 details are deferred to supplementary tables, so per-class error rates cannot be assessed from the main text alone.

## Evidence

- [discussion] DeepSAT supports Multiplicity Edited HSQC inputs: "DeepSAT supports the use of Multiplicity Edited HSQC spectra as inputs"
- [discussion] Fingerprints are populated from HSQC features: "learned to recognize features of a molecule from HSQC spectra that could be used to populate fingerprint bit strings"
- [intro] Multi-task CNN trained on 143,467 HSQC spectra: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [readme] API accepts peaks as JSON list of dicts with 1H/13C headers: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
- [results] Similarity threshold cosine score over 0.8: "we consider two structures to be “similar” if the cosine score is over 0.8 based on the predicted and known Morgan fingerprints"
