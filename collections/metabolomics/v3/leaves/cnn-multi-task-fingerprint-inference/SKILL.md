---
name: cnn-multi-task-fingerprint-inference
description: Use when the input is a 1H-13C HSQC spectrum (normal or multiplicity-edited)
  as a peak list with 1H/13C shift pairs, and the goal is to infer a fingerprint,
  molecular weight, and structure class for later similarity search against chemical
  databases.
when_to_use_negative:
- The goal is exact library matching against curated reference spectra (e.g. MetaboMiner/COLMAR-style
  search) rather than fingerprint-based inference over chemical databases — DeepSAT's
  baseline library search on NMRshiftDB achieved under 3% correct identifications,
  and the CNN approach is designed for database-scale search, not reference-library
  lookup.
- The input spectrum is not a 1H-13C HSQC or multiplicity-edited HSQC spectrum (e.g.
  13C-only, COSY, MS/MS fragmentation spectra) — the model was trained exclusively
  on 1H-13C HSQC spectra.
- A fully explained, mechanistic structure assignment is required — the authors note
  occlusion sensitivity results 'do not fully explain the decision-making process
  of DeepSAT', and interpretability was only demonstrated on a single compound (quercetin).
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_3315
tools:
- name: DeepSAT
  role: CNN multi-task model that consumes HSQC peak lists and emits predicted Morgan
    fingerprints, molecular weights, and NPClassifier structure classes
  repo: github.com/mwang87/DeepSAT
- name: TensorFlow Serving
  role: Serves the pretrained DeepSAT models; model metadata is checked via /model/metadata
    and prediction via the /api/smart3/search endpoint
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:17b943a14ebaf7e112fc514cedd639effa69c53fcd6cc153f7e202a814221de4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  - TensorFlow Serving
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cnn-multi-task-fingerprint-inference

## Summary

Run the DeepSAT convolutional neural network, trained on 143,467 1H-13C HSQC spectra, to convert a query HSQC peak list into predicted 6144-bit Morgan fingerprint bit strings, molecular weights, and NPClassifier structure classes that can then be searched against chemical databases.

## When to use

Use when the input is a 1H-13C HSQC spectrum (normal or multiplicity-edited) as a peak list with 1H/13C shift pairs, and the goal is to infer a fingerprint, molecular weight, and structure class for later similarity search against chemical databases.

## When NOT to use

- The goal is exact library matching against curated reference spectra (e.g. MetaboMiner/COLMAR-style search) rather than fingerprint-based inference over chemical databases — DeepSAT's baseline library search on NMRshiftDB achieved under 3% correct identifications, and the CNN approach is designed for database-scale search, not reference-library lookup.
- The input spectrum is not a 1H-13C HSQC or multiplicity-edited HSQC spectrum (e.g. 13C-only, COSY, MS/MS fragmentation spectra) — the model was trained exclusively on 1H-13C HSQC spectra.
- A fully explained, mechanistic structure assignment is required — the authors note occlusion sensitivity results 'do not fully explain the decision-making process of DeepSAT', and interpretability was only demonstrated on a single compound (quercetin).

## Inputs

- 1H-13C HSQC spectrum peak list (JSON list of dicts with 1H, 13C shift headers)
- 1H-13C multiplicity-edited HSQC peak list (supported input variant)

## Outputs

- 6144-bit Morgan fingerprint bit string (radius 0-2)
- Rounded molecular weight value
- NPClassifier structure class probability vector (e.g. superclass probabilities)
- Ranked similar-structure hits (cosine score > 0.8 against known Morgan fingerprints)

## How to apply

Load the query HSQC peak list (JSON list of dicts with 1H and 13C shift headers, per the DeepSAT repository README) and pass it through the pretrained CNN multi-task model served by TensorFlow Serving. The model has task-specific output heads with distinct activations and losses: a sigmoid head trained with binary cross-entropy to populate 6144-bit Morgan fingerprint bit strings (radius 0-2), a softmax head trained with sparse categorical cross-entropy for NPClassifier structure class prediction, and a regression head scored by mean absolute percentage error for molecular weight. Choose the normal HSQC or Multiplicity Edited HSQC model to match the experimental spectrum type; DeepSAT explicitly supports multiplicity-edited HSQC inputs. Decode the head outputs into a rounded molecular weight, a 6144-bit fingerprint vector, and a class probability vector; two structures are considered 'similar' when the cosine score between the predicted and known Morgan fingerprint exceeds 0.8. Rationale: predicted fingerprints bridge spectral input and structure databases, so the model can search molecules for which no authentic NMR spectrum exists.

## Related tools

- **DeepSAT** (CNN multi-task model that consumes HSQC peak lists and emits predicted Morgan fingerprints, molecular weights, and NPClassifier structure classes) — github.com/mwang87/DeepSAT
- **TensorFlow Serving** (Serves the pretrained DeepSAT models; model metadata is checked via /model/metadata and prediction via the /api/smart3/search endpoint) — github.com/mwang87/DeepSAT

## Examples

```
curl -X POST https://deepsat.ucsd.edu/api/smart3/search -d '{"peaks":[{"1H":7.41,"13C":128.6}]}'
```

## Evaluation signals

- Fingerprint head output must be a 6144-bit vector; check length and bit values in {0,1} for every input spectrum.
- All class-head probabilities must lie in [0, 1]; sanity-check per-record softmax normalization.
- Benchmark-level expectations: average cosine score between predicted and ground-truth fingerprints of 0.8450 (normal HSQC) or 0.8574 (multiplicity-edited HSQC); R-squared for molecular weight of 0.9183/0.9336; top-1 class accuracy of 90.4%/90.8%.
- Similarity search results should use the documented threshold: cosine score over 0.8 between predicted and known Morgan fingerprints defines a 'similar' structure.
- Model input names must match those exposed at the TensorFlow Serving /model/metadata endpoint — if the model input names change, update the calling code accordingly (README).

## Limitations

- Solvent-transfer robustness is incomplete: over 65% of results were still correctly identified when data were recorded in different solvents, but a substantial fraction was not.
- Interpretability is limited — occlusion sensitivity was demonstrated only on quercetin, and even then results 'do not fully explain the decision-making process of DeepSAT'.
- Some structure classes perform poorly: diazotetronic acids and derivatives (F1-score 0.64) and naphthalenes (F1-score 0.66) showed low classification values.
- The tool keeps logs of which structures were classified but not which users queried them — a privacy/telemetry consideration when running against the public service.

## Evidence

- [results] Task-specific head activations and losses: [evidence span withheld — anchor status: not_found]
- [intro] Training corpus and model architecture: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [results] Benchmark performance of the two HSQC models: "the average cosine score was 0.8450 for the normal HSQC model, and 0.8574 for the Multiplicity Edited HSQC model"
- [results] Similarity threshold: [evidence span withheld — anchor status: not_found]
- [readme] Programmatic API contract from the README: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
- [discussion] Multiplicity-edited input support: "DeepSAT supports the use of Multiplicity Edited HSQC spectra as inputs"
