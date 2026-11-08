---
name: weighted-mse-loss-implementation
description: Use when training a neural network to regress EI mass spectra as fixed-length
  (1000-bin) intensity vectors and the loss must weight m/z bins consistently with
  the downstream cosine-similarity library-matching score.
when_to_use_negative:
- When the goal is spectral prediction for molecules whose isotope-fragment intensity
  peaks must be modeled — NEIMS currently does not model isotope peaks, so training
  this loss will not fix that limitation
- When the downstream task scores spectra with a similarity metric whose bin weighting
  differs from the cosine similarity in eq 1 — the modified MSE is specifically tied
  to that weighting
- When the output is not a fixed-length per-bin intensity vector (e.g., peak-list
  or formula-level prediction), since the loss operates on intensity at every integral
  m/z bin
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: Tensorflow
  role: Framework in which the modified mean-squared-error loss and the MLP are constructed
    and trained
  repo: github.com/brain-research/deep-molecular-massspec
- name: Adam optimizer
  role: Stochastic-gradient-descent optimizer applied to the weighted-MSE loss over
    100,000 steps at batch size 100
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP inputs (length 4096, radius 2) whose predictions
    against NIST spectra the loss evaluates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:83c2b0c1caec15a802da498a79b5f000a86098d494a13f4d96e85b0a6264fc80
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# weighted-mse-loss-implementation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implementation of the modified mean-squared-error loss used to train the NEIMS multilayer perceptron for electron-ionization mass spectrum prediction, where the per-bin squared-error weighting follows the same m/z weighting pattern as the cosine similarity used in library matching (eq 1).

## When to use

Use when training a neural network to regress EI mass spectra as fixed-length (1000-bin) intensity vectors and the loss must weight m/z bins consistently with the downstream cosine-similarity library-matching score.

## When NOT to use

- When the goal is spectral prediction for molecules whose isotope-fragment intensity peaks must be modeled — NEIMS currently does not model isotope peaks, so training this loss will not fix that limitation
- When the downstream task scores spectra with a similarity metric whose bin weighting differs from the cosine similarity in eq 1 — the modified MSE is specifically tied to that weighting
- When the output is not a fixed-length per-bin intensity vector (e.g., peak-list or formula-level prediction), since the loss operates on intensity at every integral m/z bin

## Inputs

- Paired (molecule structure, NIST EI-MS spectrum) training records, spectra discretized into intensity values at every integral m/z bin (NIST 2017 Mass Spectral Main Library, 240,942 spectra)
- ECFP feature vectors (fingerprint length 4096, radius 2) computed with RDKit for each molecule
- Molecular mass of each molecule, used to mask predicted intensities at m/z values beyond the molecular mass

## Outputs

- Trained MLP spectrum-prediction model weights (TensorFlow)
- Predicted 1000-bin spectrum vectors for evaluation molecules, with high-m/z intensities zeroed out (forward_mlp_predictions.csv)

## How to apply

Treat spectrum prediction as multidimensional regression whose output is an intensity value at every integral m/z bin. Compute mean-squared error between predicted and observed per-bin intensities, but apply the modified weighting so that each m/z bin is weighted the same way as in the cosine similarity (eq 1) used for library matching — this keeps the training objective aligned with the evaluation metric. Optimize the parameters of the MLP (seven layers of 2000 nodes, residual connections, ReLU, dropout 0.25) with stochastic gradient descent using the Adam optimizer, training for 100,000 steps with batch size 100 on 240,942 spectra from the NIST 2017 Mass Spectral Main Library. After prediction, zero out predicted intensities at m/z values greater than the molecule's molecular mass before writing vectors to forward_mlp_predictions.csv.

## Related tools

- **Tensorflow** (Framework in which the modified mean-squared-error loss and the MLP are constructed and trained) — github.com/brain-research/deep-molecular-massspec
- **Adam optimizer** (Stochastic-gradient-descent optimizer applied to the weighted-MSE loss over 100,000 steps at batch size 100)
- **RDKit Cheminformatics package** (Generates the additive ECFP inputs (length 4096, radius 2) whose predictions against NIST spectra the loss evaluates)

## Evaluation signals

- Training loss decreases monotonically (converges) over the 100,000 steps at batch size 100
- Predicted outputs are well-formed 1000-bin intensity vectors, with intensities at m/z greater than the molecule's molecular mass set to zero
- Per-bin loss weighting matches the m/z weighting pattern of the cosine similarity in eq 1 used for library matching
- The forward-prediction head contributes to library-matching recall@10 consistent with the reported 91.8% recall-at-10 for the full model

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, so the loss cannot learn isotope patterns
- The loss presupposes spectra binned at every integral m/z; no version numbers for TensorFlow or other software are specified in the paper
- References cited by number (e.g., eq 1 weighting details) may not be fully resolvable from the section text alone

## Evidence

- [readme] Repository implements training/evaluation of this loss: [evidence span withheld — anchor status: external_doc]
