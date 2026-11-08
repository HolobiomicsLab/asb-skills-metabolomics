---
name: transformer-inference-confidence-scoring
description: Use when you have a pre-trained LipiDetective transformer (.pth weights
  plus training config for d_model, num_heads, n_peaks) and raw mzML spectra for which
  you need decoded lipid species strings with confidence scores.
when_to_use_negative:
- You need to train or fine-tune the transformer itself rather than run inference
  over an existing .pth checkpoint
- Input spectra are profile-level raw data that have not yet undergone smoothing,
  baseline correction, peak picking, base peak normalization, and trimming to the
  50-1600 m/z range
- Your task is lipid identification by spectral library matching against a database
  rather than neural-network decoding (e.g. LipiDex-style spectrum matching)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model whose decoder head is run autoregressively to emit lipid-name
    tokens and end-of-sequence markers from MS2 peak embeddings
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Framework in which the LipiDetective transformer module and its linear output
    head over the 1,032-token vocabulary are implemented and executed for greedy/beam
    decoding
- name: PyTorch Lightning
  role: Used for training the model; the Lightning-trained checkpoint/config pair
    is what inference loads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:228d5c30c9b7e644dd7a7efec617c4355c55e05d625cf228dd833af615ac8acd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# transformer-inference-confidence-scoring

## Summary

Runs autoregressive decoding of a trained LipiDetective transformer over MS2 spectra to produce molecular lipid species predictions, using greedy or beam search over a 1,032-token lipid-name vocabulary, with per-prediction confidence scoring for downstream filtering.

## When to use

Use when you have a pre-trained LipiDetective transformer (.pth weights plus training config for d_model, num_heads, n_peaks) and raw mzML spectra for which you need decoded lipid species strings with confidence scores.

## When NOT to use

- You need to train or fine-tune the transformer itself rather than run inference over an existing .pth checkpoint
- Input spectra are profile-level raw data that have not yet undergone smoothing, baseline correction, peak picking, base peak normalization, and trimming to the 50-1600 m/z range
- Your task is lipid identification by spectral library matching against a database rather than neural-network decoding (e.g. LipiDex-style spectrum matching)

## Inputs

- Pre-trained LipiDetective transformer weights (.pth) with matching training config (d_model, num_heads, n_peaks)
- MS2 spectra for prediction, provided as mzML files (specified via YAML config)
- YAML configuration file pointing to prediction mzML files and model parameters

## Outputs

- predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence
- Lipid species strings in shorthand notation with adduct (e.g. 'PC 16:0_18:1 [M+HCOO]-')
- Optional top-3 ranked beam-search lipid species candidates per spectrum

## How to apply

Load the pre-trained LipiDetective transformer with the exact training config (d_model, num_heads, n_peaks) and run the encoder over the peak-embedding plus positional-encoding sequence built from the top n_peaks m/z values ordered by decreasing intensity. A linear head maps the transformer state to a 1,032-token output vocabulary spanning headgroup, fatty-acid chain, and adduct tokens; decode autoregressively, feeding each generated token back as input until the end-of-sequence token is emitted. Prefer greedy decoding (maximum-probability token per step) as the default, or beam search retaining the highest cumulative-probability sequences to return the best or top-3 complete lipid species strings. Reassemble tokens into lipid shorthand notation (e.g. 'PC 16:0_18:1 [M+HCOO]-'), compute a confidence score per prediction, and apply the confidence threshold of 0.98 before accepting predictions. Write decoded results to predictions.csv with columns file, polarity, spectrum_index, precursor, prediction, and confidence.

## Related tools

- **LipiDetective** (Transformer model whose decoder head is run autoregressively to emit lipid-name tokens and end-of-sequence markers from MS2 peak embeddings) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Framework in which the LipiDetective transformer module and its linear output head over the 1,032-token vocabulary are implemented and executed for greedy/beam decoding)
- **PyTorch Lightning** (Used for training the model; the Lightning-trained checkpoint/config pair is what inference loads)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- predictions.csv exists, parses as valid CSV, and contains the required columns: file, polarity, spectrum_index, precursor, prediction, confidence
- Every prediction string conforms to lipid shorthand token format: headgroup + side-chain + adduct tokens (e.g. 'PC 16:0_18:1 [M+HCOO]-')
- Greedy decoding terminates for every spectrum (end-of-sequence token emitted) or predicts nothing for blanks; blank spectra should yield the end-of-sequence token
- All reported predictions have confidence >= 0.98 after thresholding
- Beam search returns the best or top-3 complete token sequences whose cumulative probability is maximal among retained beams

## Limitations

- Spectra from blank measurements must be included in training so the model learns to emit the end-of-sequence token when no lipid species is present; without this, decoding may not terminate sensibly
- Peaks relevant to prediction (e.g. 224.0 m/z) may lack annotation in the Alex123 database, limiting interpretability of which peaks drove a decoded prediction
- Nomenclature in external data sources must be manually adjusted to the updated lipid shorthand notation before token reconstruction round-trips correctly

## Evidence

- [methods] autoregressive decoding: [evidence span withheld — anchor status: not_found]
- [full_text] greedy decoding loop: [evidence span withheld — anchor status: not_found]
- [full_text] confidence threshold: [evidence span withheld — anchor status: not_found]
- [full_text] output CSV schema: [evidence span withheld — anchor status: not_found]
- [readme] mzML prediction input: [evidence span withheld — anchor status: not_found]
