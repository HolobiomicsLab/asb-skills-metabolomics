---
name: token-level-lipid-accuracy-scoring
description: Use when evaluating transformer-decoded lipid species predictions on
  held-out or unseen-species spectra and you need per-spectrum strict-match accuracy
  plus component-level token accuracy CSVs.
when_to_use_negative:
- Input is raw profile spectra (mzML) that still need smoothing, baseline correction,
  peak picking, base peak normalization, and trimming to 50–1600 m/z before inference
- Scoring is needed at the peak/fragment level (e.g. Alex123 fragment matching) rather
  than at the lipid species token level
- The task is blank/no-lipid detection, where the expected output is the end-of-sequence
  token rather than a scored species prediction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: LipiDetective
  role: Transformer model whose greedy/beam search decoded lipid species predictions
    are tokenized and scored
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch Lightning
  role: Manages per-epoch validation metric computation and CSV logging of accuracies
- name: h5py
  role: Loads the preprocessed MS2 spectra from the HDF5 file used as inference/scoring
    input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:38ea9789b7ab7ea817f395b09c5736135cd7821593f567f9a0d321903ccbc3dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# token-level-lipid-accuracy-scoring

## Summary

Scores LipiDetective lipid species predictions at token level, treating each shorthand-notation token (headgroup, fatty acid chains, adduct) as a unit and reporting strict total prediction accuracy (all tokens correct) alongside lenient lipid component accuracy (correct tokens / total tokens).

## When to use

Use when evaluating transformer-decoded lipid species predictions on held-out or unseen-species spectra and you need per-spectrum strict-match accuracy plus component-level token accuracy CSVs.

## When NOT to use

- Input is raw profile spectra (mzML) that still need smoothing, baseline correction, peak picking, base peak normalization, and trimming to 50–1600 m/z before inference
- Scoring is needed at the peak/fragment level (e.g. Alex123 fragment matching) rather than at the lipid species token level
- The task is blank/no-lipid detection, where the expected output is the end-of-sequence token rather than a scored species prediction

## Inputs

- Preprocessed MS2 spectra stored in an HDF5 file
- Validation-split YAML defining lipid species held out from training
- Pre-trained LipiDetective transformer weights (PyTorch) with the training configuration
- Decoded predictions from greedy and beam search decoders (shorthand-notation strings)

## Outputs

- Per-spectrum predictions CSV with token-level correct/incorrect annotation
- Per-epoch metrics CSV with total prediction accuracy and lipid component accuracy per lipid class

## How to apply

Run LipiDetective inference (greedy and beam search decoders) over the validation MS2 spectra loaded from the HDF5 file, using the validation-split YAML that defines which lipid species are held out from training. Parse each prediction and its label into tokens — headgroup, fatty acid chains, and adduct — in the updated lipid shorthand notation. Count a prediction correct under total prediction accuracy only when every token matches the label; compute lipid component accuracy as correctly predicted tokens divided by total tokens, which credits partially correct species assignments. Write per-spectrum predictions and per-epoch metrics to CSV files in the experiment output directory so accuracy can be tracked per validation epoch. Rationale: token-level scoring separates full species-level correctness from partial credit, which matters when the model generalizes to lipid species it has never seen a spectrum for (best reported case: PG at 67.3% total prediction accuracy on its held-out validation split).

## Related tools

- **LipiDetective** (Transformer model whose greedy/beam search decoded lipid species predictions are tokenized and scored) — https://github.com/LipiTUM/lipidetective
- **PyTorch Lightning** (Manages per-epoch validation metric computation and CSV logging of accuracies)
- **h5py** (Loads the preprocessed MS2 spectra from the HDF5 file used as inference/scoring input)

## Examples

```
uv run lipidetective --config config/config_templates/validation_unseen_species.yaml
```

## Evaluation signals

- Every spectrum appears in the output CSV with both a total prediction accuracy flag and a lipid component accuracy value
- Total prediction accuracy is 0/1 per spectrum and 1.0 only when headgroup, all fatty acid chains, and adduct tokens all match the label
- Lipid component accuracy is always >= total prediction accuracy for the same spectrum set (partial credit cannot lower the score)
- Per-epoch metrics CSV shows accuracies computed on the held-out unseen-species split defined in the validation YAML, not on training species
- Reported class-level accuracies fall in plausible ranges, e.g. PG at 67.3% total prediction accuracy on its held-out validation split

## Limitations

- A single mismatched token (e.g. wrong adduct or one chain) zeroes out total prediction accuracy even when most of the species is correct, making the strict metric pessimistic
- Accuracy figures depend on the validation-split YAML defining which species are unseen; different splits give different generalization numbers
- Scoring assumes labels were manually adjusted to the updated lipid shorthand notation; divergent source nomenclatures would corrupt token comparison

## Evidence

- [methods] Token scoring uses strict all-token matching plus partial credit.: [evidence span withheld — anchor status: not_found]
- [methods] Validation correctness requires headgroup, fatty acid chains, and adduct all to match.: [evidence span withheld — anchor status: not_found]
- [results] Generalization result on unseen species sets the accuracy benchmark.: [evidence span withheld — anchor status: not_found]
- [methods] Output artifacts are CSV files in the experiment directory.: [evidence span withheld — anchor status: not_found]
- [methods] Nomenclature must be harmonized before token-level scoring is meaningful.: "The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation"
