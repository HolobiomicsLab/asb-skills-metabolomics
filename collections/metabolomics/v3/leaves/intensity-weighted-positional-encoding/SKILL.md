---
name: intensity-weighted-positional-encoding
description: Use when reimplementing or extending the IDSL_MINT MS2Fingerprint transformer
  — specifically when encoding .msp MS/MS peak lists (m/z + intensity pairs) as transformer
  input sequences with intensity-aware positional embeddings.
when_to_use_negative:
- The goal is MS/MS library searching (e.g. NIST 20 / LipidBlast similarity match)
  rather than de novo fingerprint prediction — use library search first and reserve
  this for spectra without hits
- The reverse task — forecasting MS/MS spectra from canonical SMILES — which is a
  separate IDSL_MINT model configuration
- MSP blocks lack the required PrecursorMZ row entry, which IDSL_MINT requires for
  parsing
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: IDSL_MINT
  role: Framework containing the MS2Fingerprint tokenizer, intensity-weighted positional
    encoding, and transformer model; configured via YAML parameter files
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Deep learning backend implementing the transformer encoder-decoder; supports
    cuda GPU processing (device-agnostic)
  repo: https://pytorch.org/get-started/locally
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:f4038139aad1cfcce642d832ac90dd73157ef18f61ac94c80cbd3f5d8dcc93a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# intensity-weighted-positional-encoding

## Summary

Implementation detail of the IDSL_MINT MS2Fingerprint transformer: MS/MS peaks are tokenized at constant 0.01 Da m/z steps with intensities normalized to a unit sum, and sinusoidal positional encodings are scaled by these normalized intensities so peak abundance informs the attention mechanism.

## When to use

Use when reimplementing or extending the IDSL_MINT MS2Fingerprint transformer — specifically when encoding .msp MS/MS peak lists (m/z + intensity pairs) as transformer input sequences with intensity-aware positional embeddings.

## When NOT to use

- The goal is MS/MS library searching (e.g. NIST 20 / LipidBlast similarity match) rather than de novo fingerprint prediction — use library search first and reserve this for spectra without hits
- The reverse task — forecasting MS/MS spectra from canonical SMILES — which is a separate IDSL_MINT model configuration
- MSP blocks lack the required PrecursorMZ row entry, which IDSL_MINT requires for parsing

## Inputs

- MS/MS peak lists in NIST .msp format (m/z and intensity pairs, with PrecursorMZ and SMILES/InChI or Fingerprint row entries)
- Fingerprint labels: RDKit-computed ECFP or MACCS Keys, or user-provided fingerprint bit lists from the .msp Fingerprint row

## Outputs

- Intensity-weighted positional encoding tensors (one per tokenized peak) feeding a 4-layer, 2-head transformer encoder-decoder
- Per-token input embeddings combining 0.01 Da binned m/z tokens with unit-sum-normalized intensities

## How to apply

Parse training .msp spectra and tokenize each fragment m/z into discrete tokens at constant m/z interval steps (0.01 Da binning), pairing each token with its intensity. Normalize intensities to a unit sum of total intensities and sort by maximum intensity. Compute standard sinusoidal positional encodings ('Attention is all you need' style) and scale them by the normalized peak intensities so the encoder receives spectral position and abundance jointly — the article frames this as a language translation problem where 'token A (m/z and intensity)' is translated into 'token B (descriptors)'. Feed these embeddings into a PyTorch transformer encoder-decoder with 4 layers and 2 attention heads per stack, and store weights in a designated directory on the decreasing training-loss trajectory.

## Related tools

- **IDSL_MINT** (Framework containing the MS2Fingerprint tokenizer, intensity-weighted positional encoding, and transformer model; configured via YAML parameter files) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Deep learning backend implementing the transformer encoder-decoder; supports cuda GPU processing (device-agnostic)) — https://pytorch.org/get-started/locally

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Smoke test: module instantiation, forward/backward pass on an .msp-derived batch, and one beam-search decode all complete without error (script_runs check)
- The output head over the fingerprint-bit vocabulary yields a fixed-length fingerprint-bit probability vector
- Tokenized input sequences show m/z binning at constant 0.01 Da steps with intensities normalized so they sum to 1
- Training weights are stored/updated in the designated directory strictly on the decreasing training-loss trajectory
- Parsing succeeds on MoNA/GNPS-derived .msp files via comment:/comments: entries with case-insensitive SMILES:/InChI: labels

## Limitations

- Requires labeled fingerprints: either RDKit-computed ECFP/MACCS from InChI/SMILES rows or user-supplied Fingerprint row entries in the .msp; unlabeled spectra cannot supervise training
- Model quality is bounded by training-library coverage — the case study trained on only 6.96% (ESI+) and 4.58% (ESI−) of LipidMaps compounds, so poorly represented chemical classes may be mispredicted
- When multiple similar headers appear in an MSP block, only the longest content is selected for parsing; missing PrecursorMZ entries break ingestion

## Evidence

- [full_text] Tokenization and intensity handling rationale: [evidence span withheld — anchor status: not_found]
- [intro] Translation framing of spectra encoding: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [readme] Transformer architecture basis: "This innovative approach for mass spectrometry data processing has been constructed upon the transformer models delineated in the seminal paper, 'Attention is all you need'"
- [readme] Weight checkpointing on training-loss trajectory: "In the training step, weights of IDSL_MINT models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
- [readme] Beam search inference support: "4) Supports beam search inferencing."
