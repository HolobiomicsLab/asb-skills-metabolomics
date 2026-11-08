---
name: lipid-ms2-spectral-identification
description: Use when preprocessed MS2 lipid spectra (HDF5 2xN m/z/intensity arrays,
  or mzML for prediction) must be assigned molecular lipid species by a learned fragmentation
  model instead of spectral-library database matching.
when_to_use_negative:
- Input is raw profile data that has not undergone smoothing, baseline correction,
  peak picking, base peak normalization, and trimming to 50-1600 m/z — the model expects
  preprocessed peak lists
- The goal is de novo structure elucidation of unknown metabolites rather than assignment
  of molecular lipid species in the standard lipid shorthand nomenclature
- No trained LipiDetective model matching the polarity/adduct regime is available
  (workflow.load_model requires existing saved weights)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model that encodes the top-n MS2 peaks and autoregressively decodes
    the lipid shorthand token sequence
  repo: github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Framework providing the modified transformer module used for encoding/decoding
- name: PyTorch Lightning
  role: Used for multi-GPU training of the model
- name: h5py
  role: Reads/writes the HDF5 files holding the preprocessed MS2 spectra and their
    lipid metadata
- name: pyOpenMS
  role: Used in preprocessing of raw spectra into the peak-list input format
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:bad744a5934e0b26af5a5827dc4b4433a3f072559b56b0c4bc11b198c5e04684
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-ms2-spectral-identification

## Summary

Identifies molecular lipid species in tandem MS2 spectra using LipiDetective, a transformer trained on reference spectra from lipid standards, by autoregressively decoding a lipid shorthand token sequence (headgroup, fatty acid chains, adduct) from the top-n peaks of a spectrum.

## When to use

Use when preprocessed MS2 lipid spectra (HDF5 2xN m/z/intensity arrays, or mzML for prediction) must be assigned molecular lipid species by a learned fragmentation model instead of spectral-library database matching.

## When NOT to use

- Input is raw profile data that has not undergone smoothing, baseline correction, peak picking, base peak normalization, and trimming to 50-1600 m/z — the model expects preprocessed peak lists
- The goal is de novo structure elucidation of unknown metabolites rather than assignment of molecular lipid species in the standard lipid shorthand nomenclature
- No trained LipiDetective model matching the polarity/adduct regime is available (workflow.load_model requires existing saved weights)

## Inputs

- Preprocessed MS2 spectra in HDF5 format (2xN m/z/intensity arrays with lipid_species, adduct, precursor, polarity attributes), e.g. processed/test_dataset.hdf5
- LipiDetective YAML configuration file (paths to HDF5 training/test files or mzML prediction files, model hyperparameters, workflow switches)
- Pre-trained transformer weights (files.saved_model)

## Outputs

- test_predictions.csv (one prediction-vs-label row per test spectrum)
- test_metrics.csv (per-step loss and total prediction accuracy)
- test_lipid_metrics.csv (per-lipid precision/recall/F1)
- confusion_matrix_test.csv
- Per-spectrum predictions and per-epoch metrics CSVs in the experiment output directory

## How to apply

Preprocess spectra first (smoothing, baseline correction, peak picking, base peak normalization, trimming to 50-1600 m/z, exclusion of noisy spectra above a 0.01 base-peak-normalized median intensity threshold) and store them as HDF5 with lipid_species, adduct, precursor, and polarity attributes. Configure a YAML config pointing to the pre-trained transformer weights (files.saved_model) and set workflow.test: True, workflow.load_model: True, and test.confidence_score: True with batch 512. Feed each spectrum as the top-n peaks (n_peaks 30) ordered by decreasing intensity into the transformer encoder (d_model 32, num_heads 4, num_layers 2, max_mz 1600, decimal_accuracy 1) and decode the lipid shorthand token sequence autoregressively with greedy or beam search. Score a prediction as correct only when every token (headgroup, fatty acid side chains, adduct) matches the label; additionally report lipid component accuracy (correct tokens / total tokens) for partial-credit diagnostics.

## Related tools

- **LipiDetective** (Transformer model that encodes the top-n MS2 peaks and autoregressively decodes the lipid shorthand token sequence) — github.com/LipiTUM/lipidetective
- **PyTorch** (Framework providing the modified transformer module used for encoding/decoding)
- **PyTorch Lightning** (Used for multi-GPU training of the model)
- **h5py** (Reads/writes the HDF5 files holding the preprocessed MS2 spectra and their lipid metadata)
- **pyOpenMS** (Used in preprocessing of raw spectra into the peak-list input format)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- Total prediction accuracy is computed by exact token-sequence match — only if all tokens match is the prediction classified as correct
- test_metrics.csv exists and reports the total prediction accuracy; test_predictions.csv contains exactly one prediction-vs-label row per test spectrum
- Held-out validation accuracies should fall in the observed ranges (29.7%-50.8% across three random splits; 9.7% on a 10-spectra split, 39.9% on a 1000-spectra split, 67.3% total accuracy for PG on its unseen-species validation split)
- Lipid component accuracy (correct tokens / total tokens) is reported alongside total prediction accuracy so partial matches can be diagnosed
- Predictions must be syntactically valid lipid shorthand token sequences (headgroup, fatty acid chains, adduct) or an end-of-sequence prediction for blanks

## Limitations

- Generalization to truly unseen lipid species is limited — best total prediction accuracy on a held-out validation split was 67.3% (PG)
- Training data is relatively small (268,720 spectra), and 2002 blank spectra were added so the model learns to predict nothing when no lipid species is present
- Some peaks relevant to prediction (e.g. 224.0 m/z) lack annotation in the Alex123 database, limiting interpretability of those peaks
- Identifications can be hard to match back to exact spectra because spectrum IDs are rarely supplied with published raw data

## Evidence

- [abstract] abstract_claim: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
- [methods] trimming: "all MS2 spectra were trimmed to an m/z range of 50 - 1600 m/z"
- [methods] implementation: "LipiDetective was implemented by modifying the transformer module provided by PyTorch"
- [readme] readme_purpose: "LipiDetective is a deep learning framework designed for the identification of molecular lipid species from tandem mass spectra."
- [readme] readme_cli: [evidence span withheld — anchor status: not_found]
