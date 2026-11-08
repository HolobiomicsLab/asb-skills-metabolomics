---
name: msp-spectra-preprocessing
description: Use when raw EI-MS spectra in msp format (m/z / relative-intensity peak
  lists with SMILES labels) must be filtered and discretized into integer m/z and
  binned intensity tokens for training or evaluating a SpecTUS-style translation model.
when_to_use_negative:
- The input spectra are MS/MS spectra rather than GC-EI-MS spectra — SpecTUS and this
  preprocessing were designed and trained for EI-MS, and the authors note de novo
  comparison methods for MS/MS rely on precursor ion mass or molecular formula
- The downstream model expects continuous (e.g. 4-decimal-place linear) intensity
  values rather than the 30-bin logarithmic discretization — over-simplified binning
  caused a 3–4% Acc 1 drop in the paper
- The task is spectral prediction/generation (NEIMS or RASSP forward models) rather
  than encoding spectra for translation into SMILES
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SpecTUS
  role: Source repository providing the preprocessing notebooks, YAML configs, and
    trainSpectus_build.yaml conda environment used to bin intensities and filter NIST
    spectra
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Notebooks under notebooks/ document data preprocessing step by step before
    pretraining and finetuning
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment manager used to reproduce the preprocessing/training environment
    via trainSpectus_build.yaml
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:f1fb7f77ea7f34b669d462f9df8edd9f6fa570c8fc87d887ba0a439bc5fcc933
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# msp-spectra-preprocessing

## Summary

Reconstructs the SpecTUS preprocessing pipeline that converts raw GC-EI-MS peak lists in msp format (NIST 20 records with SMILES labels) into model-ready input encodings: model-specific filtering (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 characters) followed by logarithmic intensity discretization into 30 bins using log base 1.28 with a shift of 29.

## When to use

Use when raw EI-MS spectra in msp format (m/z / relative-intensity peak lists with SMILES labels) must be filtered and discretized into integer m/z and binned intensity tokens for training or evaluating a SpecTUS-style translation model.

## When NOT to use

- The input spectra are MS/MS spectra rather than GC-EI-MS spectra — SpecTUS and this preprocessing were designed and trained for EI-MS, and the authors note de novo comparison methods for MS/MS rely on precursor ion mass or molecular formula
- The downstream model expects continuous (e.g. 4-decimal-place linear) intensity values rather than the 30-bin logarithmic discretization — over-simplified binning caused a 3–4% Acc 1 drop in the paper
- The task is spectral prediction/generation (NEIMS or RASSP forward models) rather than encoding spectra for translation into SMILES

## Inputs

- Raw EI-MS spectra as m/z / relative-intensity peak lists from NIST 20 records in msp format, each with an associated SMILES label

## Outputs

- preprocessed_spectra.csv containing filtered spectra with integer-rounded m/z values and intensities discretized into one of 30 logarithmic bins (log base 1.28, shift 29)

## How to apply

Load the raw EI-MS spectra (m/z / relative-intensity peak lists with associated SMILES labels) from the NIST 20 records in msp format. Apply the model-specific filters: discard spectra with any m/z value above 500, spectra containing more than 300 peaks, and records whose SMILES string exceeds 100 characters — these thresholds excluded approximately 3% of the NIST dataset in the paper. Encode the relative intensities of surviving spectra with logarithmic binning using log base 1.28 and a shift of 29, producing 30 discrete intensity bins (the configuration selected in the paper's intensity-binning experiment, which beat 4-decimal-place linear binning by 0.9% validation Acc 1 while cutting 10 million trainable parameters). Round m/z values to the nearest integer as part of the input encoding. Write the filtered, binned records to preprocessed_spectra.csv, following the step-by-step preprocessing notebooks shipped in the SpecTUS repository.

## Related tools

- **SpecTUS** (Source repository providing the preprocessing notebooks, YAML configs, and trainSpectus_build.yaml conda environment used to bin intensities and filter NIST spectra) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Notebooks under notebooks/ document data preprocessing step by step before pretraining and finetuning) — https://github.com/hejjack/SpecTUS
- **conda** (Environment manager used to reproduce the preprocessing/training environment via trainSpectus_build.yaml)

## Examples

```
conda env create -f trainSpectus_build.yaml; jupyter notebook notebooks/1_preprocessing.ipynb
```

## Evaluation signals

- The fraction of NIST records removed by the m/z 500 / 300-peak / 100-character-SMILES thresholds should be approximately 3%, as reported in the paper
- Every retained spectrum must be representable with exactly 30 possible intensity bins after log base 1.28 binning with shift 29
- No output spectrum contains an m/z value above 500 or more than 300 peaks; no retained SMILES exceeds 100 characters
- Binned intensity representation reproduces the paper's optimal configuration (logarithmic, 30 bins) rather than the over-simplified linear 2-decimal or 10-bin variants
- Output file preprocessed_spectra.csv contains one filtered, binned record per surviving NIST input spectrum

## Limitations

- The 30-bin logarithmic encoding trades off fidelity: over-simplified binning (linear 2-decimal, log 10 bins) caused 3–4% drops in validation Acc 1, so binning choice is model-sensitive
- Model performance is sensitive to input spectra quality — SpecTUS performed worse on the noisier Cayman and MONA test sets than on curated NIST or SWGDRUG libraries
- Training the full model on the preprocessed data requires a license for the proprietary NIST20 dataset
- The preprocessed input gives the model no intermediate representation (no peak annotations or fragmentation trees), limiting interpretability and manual correction

## Evidence
