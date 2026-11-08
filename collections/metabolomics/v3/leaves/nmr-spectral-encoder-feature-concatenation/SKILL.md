---
name: nmr-spectral-encoder-feature-concatenation
description: Use when building or reproducing an end-to-end spectrum-to-structure
  model whose input is concatenated 1H/13C NMR spectra and whose downstream head is
  a pretrained transformer generating SMILES.
when_to_use_negative:
- Input is a substructure binary vector rather than raw spectra — use the substructure-to-structure
  transformer directly, which does not use a spectral encoder
- Only one spectrum type is available and a fused representation is assumed — the
  model concatenates features only 'if both spectra were being used as input'
- Molecules beyond 19 heavy atoms or requiring stereochemistry prediction — outside
  the demonstrated model scope
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Reference implementation containing the multitask model architecture, YAML
    config (model.model_args.structure_model_args with source_size 28045), and training/inference
    entry points
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: convolutional neural network (CNN)
  role: 1D CNN encoder that downsamples the interpolated 28000-point 1H NMR spectrum
    into a lower-dimensional feature space before concatenation
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer
  role: Encoder-decoder transformer (pretrained on the substructure-to-structure task)
    that consumes the concatenated spectral features to generate SMILES
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:f34d6f99ec27d91df0cacd0d12d7eb1ce807295c20e2de59267c124e58261b38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - convolutional neural network (CNN)
  - transformer
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nmr-spectral-encoder-feature-concatenation

## Summary

Builds the fused spectral feature vector for the NMR2Struct multitask model by passing processed 1H NMR spectra through a 1D CNN, embedding binned 13C chemical-shift vectors, and concatenating both feature sets so a single encoder output feeds the pretrained structure-assembling transformer.

## When to use

Use when building or reproducing an end-to-end spectrum-to-structure model whose input is concatenated 1H/13C NMR spectra and whose downstream head is a pretrained transformer generating SMILES.

## When NOT to use

- Input is a substructure binary vector rather than raw spectra — use the substructure-to-structure transformer directly, which does not use a spectral encoder
- Only one spectrum type is available and a fused representation is assumed — the model concatenates features only 'if both spectra were being used as input'
- Molecules beyond 19 heavy atoms or requiring stereochemistry prediction — outside the demonstrated model scope

## Inputs

- Processed 1H NMR spectrum as a 28000-point interpolated vector (-2 to 12 ppm, 0.0005 ppm resolution), optionally normalized by highest intensity
- 13C NMR chemical shifts binned into an 80-bin binary vector spanning 3.42 to 231.3 ppm
- Concatenated spectra hdf5 file with a 'spectra' dataset (one 1D vector per molecule) as consumed by NMR2Struct
- Pretrained transformer checkpoint (structure_model_ckpt) and alphabet.npy for target-token sizing

## Outputs

- Concatenated spectral feature vector conditioning the encoder-decoder transformer
- 15 SMILES structure predictions per spectrum via top-k sampling (k=5)
- Substructure probability outputs from the multitask head (957 substructures)

## How to apply

Preprocess each spectrum before encoding: linearly interpolate the 1H NMR from 32768 points to a 28000-point grid spanning -2 to 12 ppm at 0.0005 ppm resolution with intensities normalized to 0-1, and bin the 13C shifts into 80 bins spanning 3.42 to 231.3 ppm. Feed the processed 1H vector through a one-dimensional CNN to downsample it into a lower-dimensional representation, and embed the binned 13C binary vector separately. When both spectra are available as input, concatenate the CNN output with the 13C embedding into a single feature vector; use only one branch's output when a single spectrum type is available. The concatenated features then condition the transformer initialized from substructure-to-structure pretraining weights, with source_size set to 28045 and src_pad_token set to null per the README token conventions. Rationale: concatenation lets the multitask model jointly predict substructure labels and SMILES from raw spectral content without any molecular-formula input.

## Related tools

- **NMR2Struct** (Reference implementation containing the multitask model architecture, YAML config (model.model_args.structure_model_args with source_size 28045), and training/inference entry points) — https://github.com/MarklandGroup/NMR2Struct
- **convolutional neural network (CNN)** (1D CNN encoder that downsamples the interpolated 28000-point 1H NMR spectrum into a lower-dimensional feature space before concatenation) — https://github.com/MarklandGroup/NMR2Struct
- **transformer** (Encoder-decoder transformer (pretrained on the substructure-to-structure task) that consumes the concatenated spectral features to generate SMILES) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
nmr_infer_single_spectrum --config config.yaml --hnmr_file hnmr.csv --cnmr_file cnmr.txt --hnmr_shifts shifts.pkl --cnmr_shifts cshifts.pkl --ckpt structure_model_ckpt --normalize
```

## Evaluation signals

- End-to-end multitask model reproduces ~69.6% top-15 exact-structure accuracy on the Zenodo test split of simulated 1H/13C spectra
- Multitask inference config uses structure_model_args.source_size = 28045 and structure_model_args.src_pad_token = null per the README token table
- Encoded 1H branch dimension matches the 28000-point interpolation grid; 13C branch matches the 80-bin vector
- Predictions are canonicalized with RDKit and split into valid_predictions and bad_predictions in processed_predictions.h5, with target SMILES found within the first 15 predictions at the reported rate

## Limitations

- The model is trained on MestreNova-simulated spectra, which do not fully capture the nuances of real spectra; acquiring large experimental FID datasets remains a challenge
- Demonstrated only on molecules with up to 19 heavy atoms containing C, N, and O as non-hydrogen atoms
- Stereochemistry is not predicted; enabling it would require expanding the 957-substructure pretraining set with additional elements and larger motifs

## Evidence

- [results] we passed the processed 1H NMR spectra through a one-dimensional convolutional neural network (CNN) to downsample: "we passed the processed 1H NMR spectra through a one-dimensional convolutional neural network (CNN) to downsample the signal into a lower-dimensional space."
- [results] concatenated features when both spectra used: [evidence span withheld — anchor status: not_found]
- [results] linearly interpolated 1H NMR spectra interpolation grid: "linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm with a resolution of 0.0005 ppm."
- [results] 13C binning range: "we processed the spectra by extracting the chemical shifts and binning them into 80 bins spanning the shift range of 3.42 to 231.3 ppm."
- [readme] README multitask source_size token convention: "model.model_args.structure_model_args.source_size | 28045"
