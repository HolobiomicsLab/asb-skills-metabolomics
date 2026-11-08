---
name: mz-bin-discretization-of-mass-spectra
description: Use when a training spectrum (or predicted output) must be represented
  as a dense per-bin intensity vector — e.g. to regress spectra directly from ECFP
  features in the NEIMS forward-prediction MLP.
when_to_use_negative:
- When only peak lists / sparse peak tables (m/z, intensity pairs) are needed for
  direct library search rather than regression targets or model I/O
- When the task is matching observed spectra to an existing library without augmenting
  it with predicted spectra — the NEIMS bin-gridded pipeline is unnecessary
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Tensorflow
  role: Constructs and trains the MLP whose per-bin affine final layer emits predicted
    intensity at each integral m/z bin
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates additive ECFP fingerprints (length 4096, radius 2) that serve as
    inputs aligned to the discretized spectrum targets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:193def5cb654b07b61aca5cb1e67da76ee73e540502343d390cf735481155f26
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/brain-research/deep-molecular-massspec
  tools:
  - Adam optimizer
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mz-bin discretization of mass spectra

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Converts each electron-ionization mass spectrum into a fixed-length vector of intensity values at every integral m/z bin, the target format used by NEIMS for multidimensional regression of spectra from small molecules.

## When to use

Use when a training spectrum (or predicted output) must be represented as a dense per-bin intensity vector — e.g. to regress spectra directly from ECFP features in the NEIMS forward-prediction MLP.

## When NOT to use

- When only peak lists / sparse peak tables (m/z, intensity pairs) are needed for direct library search rather than regression targets or model I/O
- When the task is matching observed spectra to an existing library without augmenting it with predicted spectra — the NEIMS bin-gridded pipeline is unnecessary

## Inputs

- NIST 2017 Mass Spectral Main Library electron-ionization spectra with associated molecular structures (240,942 spectra for training)
- NIST 2017 replicates library spectra (noisier query-set spectra)
- Molecular structures as SDF/MOL files (e.g. examples/pentachlorobenzene.sdf) or SMILES/InChIKey from training_splits

## Outputs

- Fixed-length spectrum vectors with an intensity value at every integral m/z bin (1000-bin vectors)
- forward_mlp_predictions.csv containing predicted spectrum vectors for evaluation molecules
- Annotated SDF with predicted spectra for input molecules

## How to apply

Load each NIST EI-MS training spectrum and its associated molecular structure, then discretize the spectrum into intensity values at every integral m/z bin so the target is a fixed-length vector whose entries are the intensity at each integer m/z index. The model output is defined on this same bin grid, with a per-bin affine final layer mapping ECFP features (length 4096, radius 2) to predicted intensity. After prediction, zero out intensities at m/z values greater than the molecule's molecular mass, and write well-formed 1000-bin spectrum vectors (e.g. to forward_mlp_predictions.csv). The modified mean-squared-error loss follows the m/z weighting pattern of the cosine similarity in eq 1, so bin-level targets must be consistent between training and prediction. This bin-gridded representation is what makes library matching by spectral similarity possible against the augmented NIST library.

## Related tools

- **Tensorflow** (Constructs and trains the MLP whose per-bin affine final layer emits predicted intensity at each integral m/z bin) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates additive ECFP fingerprints (length 4096, radius 2) that serve as inputs aligned to the discretized spectrum targets)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Predicted spectra are well-formed 1000-bin vectors with one intensity entry per integral m/z bin
- Predicted intensities are zero for all m/z values greater than the molecule's molecular mass
- Training loss (modified MSE with eq 1-style m/z weighting) converges over the 100,000 training steps at batch size 100
- Forward head's discretized predictions are judged by their contribution to library matching recall@10 (91.8% for the full model, forward model as the low-m/z prediction mode)

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, so the discretized targets omit isotope structure
- Only integral m/z bins are represented; sub-integer m/z resolution is lost in the fixed-bin discretization

## Evidence

- [readme] In this task we identify molecules by matching its spectra to a library of labeled spectra. As a baseline, this library contains all of the molecules in the NIST main library: [evidence span withheld — anchor status: external_doc]
