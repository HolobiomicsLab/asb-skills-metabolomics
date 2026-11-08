---
name: label-smoothed-cross-entropy-training
description: Use when training a seq2seq transformer (e.g. IDSL_MINT MS2Fingerprint)
  on tokenized .msp MS/MS spectra to predict molecular fingerprint bits or SMILES,
  and model weights must be checkpointed on a decreasing training-loss trajectory.
when_to_use_negative:
- When the goal is spectral library matching against NIST 20 or LipidBlast rather
  than de novo descriptor prediction — IDSL_MINT is meant for spectra without library
  hits
- When the input is already molecular structures/fingerprints and the task is the
  reverse direction (fingerprints to MS/MS fragments) with different model semantics
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: IDSL_MINT
  role: Framework providing the transformer trainer, YAML configuration, .msp parser,
    and beam-search inference for spectra-to-fingerprint translation
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Deep learning backend implementing the transformer encoder-decoder, Adam optimizer,
    and label-smoothed cross-entropy loss
- name: RDKit
  role: Computes ECFP or MACCS Keys fingerprints from InChI/SMILES entries in .msp
    blocks to serve as training targets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:08d5ca24b4e16e3c1cf242af8efd36da154e406e281126537ec8e484dc0df9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# label-smoothed-cross-entropy-training

## Summary

Training procedure for the IDSL_MINT transformer models that converts MS/MS spectra into molecular fingerprints (or SMILES) by optimizing a cross-entropy loss with label smoothing using the Adam optimizer in PyTorch.

## When to use

Use when training a seq2seq transformer (e.g. IDSL_MINT MS2Fingerprint) on tokenized .msp MS/MS spectra to predict molecular fingerprint bits or SMILES, and model weights must be checkpointed on a decreasing training-loss trajectory.

## When NOT to use

- When the goal is spectral library matching against NIST 20 or LipidBlast rather than de novo descriptor prediction — IDSL_MINT is meant for spectra without library hits
- When the input is already molecular structures/fingerprints and the task is the reverse direction (fingerprints to MS/MS fragments) with different model semantics

## Inputs

- Reference MS/MS library in NIST .msp format (with PrecursorMZ and SMILES/InChI or Fingerprint row entries)
- YAML model configuration file specifying transformer parameters, fingerprint type (e.g. ECFP or MACCS Keys via RDKit), and output directory
- Customizable molecular fingerprint descriptor bit lists parsed from MSP Fingerprint rows

## Outputs

- Trained transformer model weights stored in a designated directory along the decreasing training-loss trajectory
- Predicted fixed-length molecular fingerprint-bit probability vector (SoftMax output head) or canonical SMILES sequence per input spectrum

## How to apply

Tokenize MS/MS peak lists from .msp blocks (m/z binned at 0.01 Da steps, intensities normalized to a unit sum and sorted by maximum intensity) and build the transformer encoder-decoder in PyTorch (4 layers, 2 attention heads). Train with cross-entropy loss using label smoothing and the Adam optimizer, as configured in the model's YAML file. Because fingerprints and SMILES are framed as a token-translation problem (m/z+intensity tokens translated into descriptor tokens), label smoothing softens hard targets over the fingerprint-bit/SMILES vocabulary. During training, save and update model weights in a designated directory along the decreasing trajectory of the training loss so the best checkpoint is retained. After training, decode with beam search rather than greedy decoding and verify the output head yields a fixed-length fingerprint-bit probability vector (or a SMILES sequence).

## Related tools

- **IDSL_MINT** (Framework providing the transformer trainer, YAML configuration, .msp parser, and beam-search inference for spectra-to-fingerprint translation) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Deep learning backend implementing the transformer encoder-decoder, Adam optimizer, and label-smoothed cross-entropy loss)
- **RDKit** (Computes ECFP or MACCS Keys fingerprints from InChI/SMILES entries in .msp blocks to serve as training targets)

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Training loss decreases monotonically and checkpoints are stored on the decreasing training-loss trajectory in the designated directory
- Instantiation, a forward/backward pass on an .msp-derived batch, and one beam-search decode all complete without error (script_runs)
- The output head produces a fixed-length fingerprint-bit probability vector over the fingerprint-bit vocabulary
- Predicted fingerprints retrieve the correct structure as top hit when searched against the LipidMaps database (Tanimoto similarity ≥ 0.6 counts as high-confidence)

## Limitations

- Model quality depends on the user-provided reference MS/MS library; the case-study training set covered only 6.96% and 4.58% of LipidMaps compounds in positive and negative modes
- Spectra must come from .msp blocks containing PrecursorMZ entries; multiple similar headers are resolved by taking the longest content
- Training is ion-mode specific — separate ESI+ and ESI− models were trained in the case study

## Evidence

- [intro] Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors): "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [readme] In the training step, weights of IDSL_MINT models are stored and updated in a designated directory on the decreasing trajectory of the training loss value: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
- [methods] Configure training with cross-entropy loss using label smoothing and the Adam optimizer, storing model weights in a designated directory on the decreasing training-loss trajectory: [evidence span withheld — anchor status: not_found]
- [results] m/z fragments are tokenized using constant m/z interval steps, and intensity values are sorted by maximum intensity after normalization to a unit sum of total intensities: [evidence span withheld — anchor status: not_found]
