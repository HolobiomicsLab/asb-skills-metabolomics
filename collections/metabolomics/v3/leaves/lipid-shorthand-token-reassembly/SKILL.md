---
name: lipid-shorthand-token-reassembly
description: Use when running LipiDetective prediction on MS2 spectra and you have
  raw decoded token sequences (headgroup, fatty-acid chain, adduct, end-of-sequence
  tokens from the 1,032-token vocabulary) that must be converted to lipid shorthand
  notation strings with confidence values.
when_to_use_negative:
- Input is already a complete, human-readable lipid annotation string or a database
  entry (e.g. from LipiDex or Compound Discoverer 3.1) — no token reassembly is needed.
- The task is raw spectra preprocessing (smoothing, baseline correction, peak picking,
  m/z 50–1600 trimming, base peak normalization) rather than post-decoding formatting.
- Working with the training/HDF5 pipeline where spectra are stored as arrays rather
  than decoded predictions.
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0154
tools:
- name: LipiDetective
  role: Emits the token sequences and confidence scores that this skill reassembles
    into lipid shorthand strings; provides the transformer model whose vocabulary
    defines valid tokens
  repo: github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Provides the transformer module whose token vocabularies and decoding heads
    produce the raw token sequence to be reassembled
- name: PyTorch Lightning
  role: Used for multi-GPU training of the model whose outputs are reassembled here;
    not directly involved in reassembly but part of the same pipeline
- name: pyOpenMS
  role: Reads the mzML prediction input files and supports writing predictions.csv
    alongside the decoded results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:b473471dfeaf25554d5ad981ae5b206ce42e01b25fda8cfacfe7d82719ccd2e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lipid-shorthand-token-reassembly

## Summary

Reassemble the token-by-token output of LipiDetective's transformer decoder into the lipid shorthand notation string (e.g. 'PC 16:0_18:1 [M+HCOO]-') and attach a confidence score, applying the 0.98 threshold before exporting results as CSV. This is the final formatting/validation layer of the autoregressive decoding pipeline.

## When to use

Use when running LipiDetective prediction on MS2 spectra and you have raw decoded token sequences (headgroup, fatty-acid chain, adduct, end-of-sequence tokens from the 1,032-token vocabulary) that must be converted to lipid shorthand notation strings with confidence values.

## When NOT to use

- Input is already a complete, human-readable lipid annotation string or a database entry (e.g. from LipiDex or Compound Discoverer 3.1) — no token reassembly is needed.
- The task is raw spectra preprocessing (smoothing, baseline correction, peak picking, m/z 50–1600 trimming, base peak normalization) rather than post-decoding formatting.
- Working with the training/HDF5 pipeline where spectra are stored as arrays rather than decoded predictions.

## Inputs

- Decoded token sequences from the transformer decoder (greedy or beam search), using the 1,032-token lipid-name vocabulary
- Model state / probability distribution at each decoding step (for confidence scoring)
- Predicted spectrum metadata (file name, polarity, spectrum index, precursor m/z)
- Confidence threshold parameter (default 0.98)

## Outputs

- predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, confidence
- Reassembled lipid species strings in shorthand notation with adduct (e.g. 'PC 16:0_18:1 [M+HCOO]-')
- Per-prediction confidence scores

## How to apply

After autoregressive decoding (greedy or beam search) over the tokenized lipid-name vocabulary, collect the emitted token sequence up to the end-of-sequence token and concatenate the tokens in order: headgroup token, fatty-acid side-chain tokens, and adduct token, inserting the underscore separator between fatty-acid chains as required by the updated lipid shorthand notation. Compute a confidence score per prediction from the model output, and apply the confidence threshold of 0.98 to decide whether to report the prediction. Write the reassembled species strings to predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence. Validate that the output string conforms to the lipid shorthand token format (headgroup + side-chain + adduct tokens) so downstream consumers can match against the Lipid Maps Structure Database or Alex123 fragments.

## Related tools

- **LipiDetective** (Emits the token sequences and confidence scores that this skill reassembles into lipid shorthand strings; provides the transformer model whose vocabulary defines valid tokens) — github.com/LipiTUM/lipidetective
- **PyTorch** (Provides the transformer module whose token vocabularies and decoding heads produce the raw token sequence to be reassembled)
- **PyTorch Lightning** (Used for multi-GPU training of the model whose outputs are reassembled here; not directly involved in reassembly but part of the same pipeline)
- **pyOpenMS** (Reads the mzML prediction input files and supports writing predictions.csv alongside the decoded results)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- predictions.csv exists and is valid CSV containing the required columns file, polarity, spectrum_index, precursor, prediction, and confidence
- Each prediction string conforms to the lipid shorthand token format: headgroup token + fatty-acid side-chain tokens + adduct token, with underscore separators between fatty-acid chains
- Every reported prediction has an associated confidence value ≥ 0.98 (the applied threshold)
- Token sequence terminates with the end-of-sequence token emitted by the decoder before reassembly
- Blank-measurement inputs (added to the training data so the model learns to predict nothing) should yield the end-of-sequence token and be filtered rather than reassembled into a species string

## Limitations

- The model was trained on a relatively small dataset of 268,720 spectra; rare or unusual lipid species may produce token sequences that do not reassemble into a valid shorthand string.
- Peaks relevant for prediction (e.g. 224.0 m/z) lack annotation in the Alex123 database, which can affect the confidence that backs a reassembled prediction.
- The varying nomenclatures in different data sources had to be manually adjusted to the updated lipid shorthand notation; token reassembly inherits this vocabulary constraint.
- Annotations by a single software can lead to high false positive identifications — the 0.98 confidence threshold is a mitigation, not a guarantee.

## Evidence

- [methods] Autoregressive decoding step described in article methods: [evidence span withheld — anchor status: not_found]
- [methods] Reassembly into shorthand notation with adduct: [evidence span withheld — anchor status: not_found]
- [methods] Output CSV schema from the source card workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Generalization to unseen lipid species from the abstract: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
- [results] Confidence threshold and blank handling from the results: "2002 MS2 spectra from blank measurements that do not show any target lipid species were added"
- [readme] LipiDetective identifies lipids in shorthand nomenclature format: [evidence span withheld — anchor status: external_doc]
- [readme] Input formats supported by the repo: [evidence span withheld — anchor status: not_found]
- [readme] Configuring the prediction run: [evidence span withheld — anchor status: external_doc]
- [other] Repos entry pointing to the GitHub source: "https://github.com/LipiTUM/lipidetective/actions/workflows/lint.yml/badge.svg"
