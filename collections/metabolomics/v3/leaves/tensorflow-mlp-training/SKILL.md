---
name: tensorflow-mlp-training
description: Use when small molecules (e.g. SDF/InChIKey inputs) must be converted
  to spectra predictions via a supervised neural network trained on NIST EI-MS spectra
  — specifically when a residual MLP regression head over 4096-bit ECFP features is
  the chosen featurization-and-prediction scheme.
when_to_use_negative:
- The input is already a feature table or precomputed spectral library — this skill
  is a supervised training/prediction pipeline, not a library-matching search step.
- You need to model isotope intensity peaks in ion fragments — NEIMS currently does
  not have a method to model these.
- The task requires predicting spectra for ionization methods other than electron
  ionization without retraining — the model was trained on EI spectra and would need
  retraining.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: RDKit Cheminformatics package
  role: Generates additive Extended Circular Fingerprints (ECFPs) of length 4096,
    radius 2, that serve as MLP input features
- name: Tensorflow
  role: Constructs and trains the seven-layer residual MLP for spectral regression
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: Adam optimizer
  role: SGD optimizer for MLP parameter updates during the 100 000-step training run
- name: Vizier
  role: Hyperparameter tuning service used to select the optimal MLP architecture
    (seven layers of 2000 nodes, dropout 0.25)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:1520045999fb64242ec8ede0f12e8fdc77757a60feb3f9a9728c785f99381b19
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/brain-research/deep-molecular-massspec
  tools:
  - Adam optimizer
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tensorflow-mlp-training

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Trains the NEIMS multilayer perceptron (MLP) in TensorFlow to predict EI mass spectra from ECFP molecular fingerprints, using a residual, ReLU-activated architecture optimized with Adam over a modified mean-squared-error loss that mirrors the cosine-similarity m/z weighting.

## When to use

Use when small molecules (e.g. SDF/InChIKey inputs) must be converted to spectra predictions via a supervised neural network trained on NIST EI-MS spectra — specifically when a residual MLP regression head over 4096-bit ECFP features is the chosen featurization-and-prediction scheme.

## When NOT to use

- The input is already a feature table or precomputed spectral library — this skill is a supervised training/prediction pipeline, not a library-matching search step.
- You need to model isotope intensity peaks in ion fragments — NEIMS currently does not have a method to model these.
- The task requires predicting spectra for ionization methods other than electron ionization without retraining — the model was trained on EI spectra and would need retraining.

## Inputs

- NIST 2017 Mass Spectral Main Library training spectra (240 942 spectra) with associated molecular structures
- Molecular structures (e.g. .sdf, InChIKey/SMILES) for featurization into ECFPs
- Trained model weights (massspec_weights.zip) for prediction-only use

## Outputs

- Predicted EI mass spectra as 1000-bin intensity vectors at every integral m/z bin
- forward_mlp_predictions.csv — predicted spectrum vectors for evaluation molecules
- Annotated .sdf output file with predicted spectra (via make_spectra_prediction.py)

## How to apply

Featurize each molecule into an additive Extended Circular Fingerprint using RDKit with fingerprint length 4096 and radius 2, hashing subgraph occurrence counts into the vector. Pass these ECFP features into a multilayer perceptron built in TensorFlow: the optimal architecture is seven layers of 2000 nodes with residual network connections between layers, ReLU activation, and a dropout rate of 0.25. Train on 240 942 spectra from the NIST 2017 Mass Spectral Main Library for 100 000 training steps with a batch size of 100, optimizing with stochastic gradient descent via the Adam optimizer. Use a modified mean-squared-error loss that follows the same m/z weighting pattern as the cosine similarity in eq 1. After training, zero out predicted intensities at m/z values greater than the molecule's molecular mass and write predicted 1000-bin spectrum vectors (e.g. forward_mlp_predictions.csv).

## Related tools

- **RDKit Cheminformatics package** (Generates additive Extended Circular Fingerprints (ECFPs) of length 4096, radius 2, that serve as MLP input features)
- **Tensorflow** (Constructs and trains the seven-layer residual MLP for spectral regression) — https://github.com/brain-research/deep-molecular-massspec
- **Adam optimizer** (SGD optimizer for MLP parameter updates during the 100 000-step training run)
- **Vizier** (Hyperparameter tuning service used to select the optimal MLP architecture (seven layers of 2000 nodes, dropout 0.25))

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Training loss converges over the 100 000 steps at batch size 100 on the NIST 2017 training split.
- Predicted spectra are well-formed 1000-bin vectors with intensities zeroed at m/z values greater than the molecule's molecular mass.
- Forward head contributes to library matching recall@10 — the full model reports 91.8% recall-at-10; compare against this benchmark.
- Predictions average ~5 ms per molecule on the trained model.
- Held-out NIST 2017 replicates query spectra should show similar library matching performance to previously reported machine learning models.

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments.
- Predictive capability is similar to previously reported machine learning models — improvement over CFM-EI is mainly in speed, not accuracy.
- The model was trained on EI spectra; it must be retrained for other ionization methods.
- Requires specific versions of tensorflow (1.13.2) and RDKit to reproduce the environment.

## Evidence
