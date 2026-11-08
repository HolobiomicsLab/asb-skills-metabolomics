---
name: transformer-sequence-decoding-inference
description: Use when you need to decode preprocessed, base-peak-normalized MS2 spectra
  (2xN m/z/intensity arrays in HDF5, m/z range 50-1600, top-n peaks by decreasing
  intensity) into lipid shorthand species tokens using a trained transformer model
  in test/predict mode.
when_to_use_negative:
- You need to train or fine-tune the transformer weights rather than run inference
  — use the training workflow instead of test/predict mode.
- Input spectra are raw profile data that have not undergone smoothing, baseline correction,
  peak picking, base-peak normalization, and m/z trimming to 50–1600.
- You require database/spectrum-matching identification (e.g. LipiDex, GNPS, Alex123)
  rather than sequence decoding into lipid shorthand tokens.
edam_operation: http://edamontology.org/operation_3809
edam_topics:
- http://edamontology.org/topic_3344
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model that identifies lipids in MS2 spectra by autoregressively
    decoding lipid shorthand tokens from the encoded top-n peaks
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Modified PyTorch transformer module implements LipiDetective's encoder-decoder
- name: PyTorch Lightning
  role: Used for multi-GPU training of the model and for organizing the inference/test
    workflow
- name: h5py
  role: Storage format for preprocessed MS2 spectra consumed at inference time
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:03a663ffff65d5a1fe0bc4f7fa82f7bc1ec1a9361d882885a0c9101457ce0dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_3809
  edam_topics:
  - http://edamontology.org/topic_3344
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# transformer-sequence-decoding-inference

## Summary

Autoregressive decoding of lipid shorthand token sequences from MS2 spectra using LipiDetective's transformer (d_model 32, num_heads 4, num_layers 2, n_peaks 30, max_mz 1600), with greedy/beam search inference over the loaded saved model weights.

## When to use

Use when you need to decode preprocessed, base-peak-normalized MS2 spectra (2xN m/z/intensity arrays in HDF5, m/z range 50-1600, top-n peaks by decreasing intensity) into lipid shorthand species tokens using a trained transformer model in test/predict mode.

## When NOT to use

- You need to train or fine-tune the transformer weights rather than run inference — use the training workflow instead of test/predict mode.
- Input spectra are raw profile data that have not undergone smoothing, baseline correction, peak picking, base-peak normalization, and m/z trimming to 50–1600.
- You require database/spectrum-matching identification (e.g. LipiDex, GNPS, Alex123) rather than sequence decoding into lipid shorthand tokens.

## Inputs

- Preprocessed MS2 spectra stored as HDF5 files (2xN m/z/intensity arrays with lipid_species, adduct, precursor, polarity attributes), e.g. processed/test_dataset.hdf5
- mzML files for prediction mode
- LipiDetective YAML config file (workflow, test.confidence_score, batch, files.saved_model, model architecture parameters)
- Pre-trained transformer saved model weights

## Outputs

- Predicted lipid shorthand token sequences per spectrum (headgroup, fatty acid chains, adduct)
- Confidence scores per prediction when test.confidence_score: True
- CSV outputs: test_metrics.csv, test_predictions.csv, test_lipid_metrics.csv, confusion_matrix_test.csv

## How to apply

Load the preprocessed MS2 spectra from an HDF5 file where each spectrum is a 2xN m/z/intensity array carrying lipid_species, adduct, precursor, and polarity attributes, with peaks trimmed to 50–1600 m/z and base-peak normalized. Configure the LipiDetective YAML with workflow.test: True (or prediction mode for mzML inputs), workflow.load_model: True, test.confidence_score: True, batch 512, and files.saved_model pointing to the pre-trained transformer weights; the encoder consumes the top-n peaks (n_peaks 30, decimal_accuracy 1, max_mz 1600) ordered by decreasing intensity, while the decoder autoregressively emits the lipid shorthand token sequence (headgroup, fatty acid side chains, adduct). Use greedy and/or beam search decoding; for spectra from blank measurements, the model should emit the end-of-sequence token meaning 'no lipid species present'. Rationale: the transformer learns the characteristic lipid fragmentation pattern of each species rather than matching overall spectra, enabling identification independent of collision energy and even of species never seen in training.

## Related tools

- **LipiDetective** (Transformer model that identifies lipids in MS2 spectra by autoregressively decoding lipid shorthand tokens from the encoded top-n peaks) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Modified PyTorch transformer module implements LipiDetective's encoder-decoder)
- **PyTorch Lightning** (Used for multi-GPU training of the model and for organizing the inference/test workflow)
- **h5py** (Storage format for preprocessed MS2 spectra consumed at inference time)

## Examples

```
uv run lipidetective --config config/config_templates/test_config.yaml
```

## Evaluation signals

- A prediction is counted correct only if all tokens (headgroup, fatty acid chains, adduct) match the label — exact token-sequence match, not partial.
- test_metrics.csv must exist and report total prediction accuracy; on LipiDetective's held-out splits this ranged from 29.7% to 50.8% across three random splits (PG best at 67.3%).
- test_predictions.csv contains one prediction-vs-label row per test spectrum.
- For blank-measurement spectra the decoder should emit the end-of-sequence token ('no lipid species present') rather than hallucinating a lipid.
- Check the YAML: workflow.test True, workflow.load_model True, test.confidence_score True, batch 512, files.saved_model pointing to trained weights.

## Limitations

- Generalization is imperfect: total prediction accuracy on unseen lipid species ranged from 29.7% to 50.8% across three random splits, with a low of 9.7% on the 10-spectra split.
- Relevant peaks such as 224.0 m/z lack annotation in the Alex123 database, so interpretation of some attention peaks is incomplete.
- The training set was relatively small (268,720 spectra) plus 2,002 blank spectra, so performance depends on how representative those reference spectra are of the target sample matrix.
- The repository README notes prediction mode consumes mzML files while training/test mode consumes HDF5 — mixing formats without preprocessing will fail.

## Evidence

- [methods] Transformer consumes top-n peaks and autoregressively decodes lipid shorthand: "LipiDetective was implemented by modifying the transformer module provided by PyTorch"
- [abstract] Model generalizes to unseen lipid species: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
- [results] Training included blank spectra teaching the end-of-sequence token: "2002 MS2 spectra from blank measurements that do not show any target lipid species were added."
- [methods] All profile spectra underwent smoothing, baseline correction, and peak picking: [evidence span withheld — anchor status: not_found]
- [readme] README specifies YAML config and workflows for training with HDF5 or prediction with mzML: [evidence span withheld — anchor status: not_found]
- [readme] README invocation: "uv run lipidetective --config path/to/config.yaml"
