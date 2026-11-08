---
name: transformer-encoder-decoder-implementation
description: 'Use when building a spectra-to-descriptor model from .msp training data:
  MS/MS peak lists (m/z, intensity) must be tokenized, encoded with intensity-scaled
  positional encodings, and translated into fingerprint-bit or SMILES token sequences.'
when_to_use_negative:
- The goal is spectral library matching against NIST/LipidBlast/MoNA by spectral similarity
  (e.g. spectral entropy scoring), not de novo descriptor prediction
- Input is raw LC-MS data rather than deconvoluted MS/MS spectra in .msp format —
  precursor peak extraction (e.g. via IDSL.CSA) must happen first
- Only a small, chemically narrow training set is available without ensuring the chemical
  classes of interest are represented
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: IDSL_MINT
  role: Framework providing the transformer implementation, .msp parser, YAML configuration,
    and beam-search inference
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Backend framework in which the encoder-decoder transformer is implemented
    and trained
  repo: https://pytorch.org
- name: RDKit
  role: Converts InChI/SMILES in .msp blocks to canonical SMILES and computes ECFP/MACCS
    fingerprint targets
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:fa9c13e2bf41b5de7184db8a67faa832518d5ff78fa71528677d9d18c30a6ee2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# transformer-encoder-decoder-implementation

## Summary

Reimplements the IDSL_MINT MS2Fingerprint transformer that treats MS/MS spectra-to-molecular-descriptor prediction as a token translation problem, following the 'Attention is all you need' encoder-decoder architecture in PyTorch. It tokenizes binned m/z fragments with normalized intensities and decodes fingerprint-bit (or SMILES) token sequences via beam search.

## When to use

Use when building a spectra-to-descriptor model from .msp training data: MS/MS peak lists (m/z, intensity) must be tokenized, encoded with intensity-scaled positional encodings, and translated into fingerprint-bit or SMILES token sequences.

## When NOT to use

- The goal is spectral library matching against NIST/LipidBlast/MoNA by spectral similarity (e.g. spectral entropy scoring), not de novo descriptor prediction
- Input is raw LC-MS data rather than deconvoluted MS/MS spectra in .msp format — precursor peak extraction (e.g. via IDSL.CSA) must happen first
- Only a small, chemically narrow training set is available without ensuring the chemical classes of interest are represented

## Inputs

- Reference MS/MS training library in NIST .msp format (e.g. MoNA/GNPS/LipidMaps spectra with PrecursorMZ and SMILES/InChI or Fingerprint rows)
- Custom fingerprint-bit definitions parsed from the Fingerprint: row of .msp blocks (e.g. ECFP or MACCS Keys bits computed with RDKit)
- YAML model-configuration file specifying training/inference parameters

## Outputs

- Trained transformer model weights stored in a designated directory on the decreasing training-loss trajectory
- Predicted fingerprint-bit probability vector (fixed-length) per input MS/MS spectrum
- Beam-search decoded token sequences (fingerprint bits or canonical SMILES)

## How to apply

Tokenize MS/MS peak lists from .msp blocks by binning m/z values into tokens at constant 0.01 Da intervals and pairing each token with its intensity normalized to a unit sum; the README notes MSP blocks must include PrecursorMZ entries and that IDSL_MINT parses comment/comments entries and SMILES/InChI labels case-insensitively. Build sinusoidal positional encodings scaled by the normalized peak intensities so the encoder receives combined spectral position and intensity information. Assemble the PyTorch transformer encoder-decoder (per the article, 4 layers and 2 attention heads in each stack) with a linear layer plus SoftMax output head over the fingerprint-bit vocabulary for the spectra-to-fingerprint variant. Train with cross-entropy loss using label smoothing and the Adam optimizer, saving weights in a designated directory on the decreasing training-loss trajectory, as the README instructs. For inference, use beam search decoding rather than greedy decoding; the same scaffold supports the inverse task of forecasting MS/MS fragments from canonical SMILES.

## Related tools

- **IDSL_MINT** (Framework providing the transformer implementation, .msp parser, YAML configuration, and beam-search inference) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Backend framework in which the encoder-decoder transformer is implemented and trained) — https://pytorch.org
- **RDKit** (Converts InChI/SMILES in .msp blocks to canonical SMILES and computes ECFP/MACCS fingerprint targets) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Module passes a smoke test: instantiation, one forward/backward pass on an .msp-derived batch, and one beam-search decode complete without error
- Output head yields a fixed-length fingerprint-bit probability vector (SoftMax-normalized)
- Training loss trajectory is strictly decreasing and model weights are saved/updated in the designated directory on that trajectory
- Tokenization check: m/z tokens fall on 0.01 Da grid bins and paired intensities sum to 1 after normalization
- Downstream quality: predicted fingerprints searched against LipidMaps yield top-hit true positives, with high-confidence new annotations at Tanimoto similarity ≥ 0.6

## Limitations

- Training-data quality governs model validity: in the case study, spectra of compounds already covered by LipidBlast and NIST 20 were excluded, and classes with fewer than 50 spectra were dropped
- Positive (ESI+) and negative (ESI−) modes require separately trained models; spectra from MoNA and GNPS were used separately per mode
- Performance is bounded by library coverage of user chemical classes — >80% of MS/MS spectra historically lack library hits, so models trained on user libraries may omit classes of interest

## Evidence

- [intro] Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors): "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [readme] In the training step, weights of IDSL_MINT models are stored and updated in a designated directory on the decreasing trajectory of the training loss value: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
- [readme] This innovative approach for mass spectrometry data processing has been constructed upon the transformer models delineated in the seminal paper, 'Attention is all you need': "This innovative approach for mass spectrometry data processing has been constructed upon the transformer models delineated in the seminal paper, [*'Attention is all you"
- [readme] Supports beam search inferencing.: "4) Supports beam search inferencing."
- [readme] MSP blocks must include PrecursorMZ: row entries.: "MSP blocks must include `PrecursorMZ: ` row entries."
