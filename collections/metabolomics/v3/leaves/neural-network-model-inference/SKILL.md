---
name: neural-network-model-inference
description: Use when you need per-molecule mass spectra predictions in milliseconds
  (e.g., to augment a spectral library with synthetic spectra for millions of candidate
  molecules) and have SMILES/SDF inputs plus the pretrained NEIMS weights.
when_to_use_negative:
- You need isotope intensity peaks for ion fragments — NEIMS has no method to model
  them
- You need quantum-chemical accuracy spectra rather than fast ML approximations, and
  runtime of minutes per molecule is acceptable
- Your molecules fall outside the NIST 2017 training domain (small organic molecules
  under electron ionization); the model may need retraining for other ionization methods
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: NEIMS
  role: Pretrained neural network model whose forward pass produces the predicted
    EI mass spectra
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates additive ECFP fingerprints (length 4096, radius 2) from input molecule
    structures
- name: Tensorflow
  role: Runtime that constructs and executes the seven-layer 2000-node MLP with residual
    connections for inference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:7dbc2b677349f15e344f3f2c8b29bfa528cc13f468c2264b03696440aa301725
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# neural-network-model-inference

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Run fast neural-network inference for electron-ionization mass spectral prediction: convert molecules to additive ECFP fingerprints (length 4096, radius 2) and pass them through the trained NEIMS MLP to predict spectra at ~5 ms per molecule, orders of magnitude faster than CFM-EI or QCEIMS.

## When to use

Use when you need per-molecule mass spectra predictions in milliseconds (e.g., to augment a spectral library with synthetic spectra for millions of candidate molecules) and have SMILES/SDF inputs plus the pretrained NEIMS weights.

## When NOT to use

- You need isotope intensity peaks for ion fragments — NEIMS has no method to model them
- You need quantum-chemical accuracy spectra rather than fast ML approximations, and runtime of minutes per molecule is acceptable
- Your molecules fall outside the NIST 2017 training domain (small organic molecules under electron ionization); the model may need retraining for other ionization methods

## Inputs

- Small-molecule structures as SDF files (e.g., examples/pentachlorobenzene.sdf) or SMILES/InChIKey from the NIST 2017 Mass Spectral Library
- Pretrained NEIMS TensorFlow model weights (massspec_weights.zip checkpoint)
- Additive ECFP fingerprints (length 4096, radius 2) generated with RDKit

## Outputs

- Predicted EI mass spectrum as an intensity vector over integral m/z bins (annotated SDF output file)
- Per-molecule inference timings (expected ~5 ms/molecule) for runtime comparison vs CFM-EI (~10 min) and QCEIMS (>=1000 s)
- Synthetic spectra for augmenting a reference library (e.g., runtime_comparison.csv tabulation)

## How to apply

Download the pretrained NEIMS weights (TensorFlow 1.13.2 checkpoint) from the deep-molecular-massspec storage bucket and load the seven-layer, 2000-node MLP with residual connections. Convert each input molecule (SDF or SMILES) to additive ECFP fingerprints of length 4096 with radius 2 using RDKit. Run inference with the bidirectional prediction head, which combines forward and reverse predictions through a gate, to produce an intensity vector at every integral m/z bin. Time per-molecule predictions and expect ~5 ms each on CPU (about 1 million molecules in 90 min). Validate by checking the output is a normalized spectrum vector suitable for library matching against the NIST 2017 reference library.

## Related tools

- **NEIMS** (Pretrained neural network model whose forward pass produces the predicted EI mass spectra) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates additive ECFP fingerprints (length 4096, radius 2) from input molecule structures)
- **Tensorflow** (Runtime that constructs and executes the seven-layer 2000-node MLP with residual connections for inference)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Average per-molecule prediction time is on the order of milliseconds (~5 ms), not seconds or minutes
- Predicted spectrum is a vector of intensities at integral m/z bins, compatible with cosine-similarity library matching
- Bidirectional output reflects combined forward and reverse predictions through the gating mechanism, not a single-direction head
- Fingerprint dimensionality matches the trained model input (4096) and radius-2 ECFP; mismatch causes inference failure
- Recall-at-10 library matching accuracy of predictions is in the reported range (91.8% reported) when used to augment the NIST library

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments
- Performance was benchmarked on the NIST 2017 Mass Spectral Library small molecules; accuracy may drop outside this chemical space
- No version numbers are given for the model or software; runtime claims assume CPU inference of the lightweight MLP

## Evidence

- [readme] README prediction command: [evidence span withheld — anchor status: not_found]
