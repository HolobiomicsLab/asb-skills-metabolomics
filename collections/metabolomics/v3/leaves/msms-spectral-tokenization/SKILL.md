---
name: msms-spectral-tokenization
description: Use when raw MS/MS spectra in .msp format must be converted into transformer-ready
  token sequences (m/z binned into tokens, intensities normalized) for training or
  inference with IDSL_MINT-style encoder-decoder models.
when_to_use_negative:
- The goal is traditional library matching (e.g. spectral entropy score ≥ 0.75 against
  NIST 20 or LipidBlast) rather than deep-learning translation of spectra into descriptors
- The .msp blocks lack PrecursorMZ row entries or any SMILES/InChI/fingerprint labels,
  since IDSL_MINT requires PrecursorMZ and identifies structures via SMILES/InChI
  labels
- Input is already a fingerprint-bit vector or descriptor table (tokenization applies
  to raw peak lists, not to precomputed descriptors)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: IDSL_MINT
  role: Framework that performs the .msp parsing, m/z tokenization, intensity normalization,
    and transformer training/inference via YAML-configured MINT_workflow runs
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Implements the transformer encoder-decoder stack, cross-entropy training loop
    with label smoothing, Adam optimizer, and beam-search decoding; device-agnostic
    with cuda support
  repo: https://github.com/pytorch
- name: RDKit
  role: Converts InChI and SMILES row entries in MSP blocks into canonical SMILES
    and computes ECFP/MACCS fingerprint labels used as target tokens
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:c7660013cc9cebe34d9755ee8ac0f2d92e23fa948b6061a0e8a54bb37d27bc5f
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# msms-spectral-tokenization

## Summary

Tokenize MS/MS peak lists from NIST .msp spectra into discrete m/z-interval tokens paired with normalized intensities, so a transformer (IDSL_MINT) can treat spectra-to-fingerprint or spectra-to-SMILES prediction as a language translation problem.

## When to use

Use when raw MS/MS spectra in .msp format must be converted into transformer-ready token sequences (m/z binned into tokens, intensities normalized) for training or inference with IDSL_MINT-style encoder-decoder models.

## When NOT to use

- The goal is traditional library matching (e.g. spectral entropy score ≥ 0.75 against NIST 20 or LipidBlast) rather than deep-learning translation of spectra into descriptors
- The .msp blocks lack PrecursorMZ row entries or any SMILES/InChI/fingerprint labels, since IDSL_MINT requires PrecursorMZ and identifies structures via SMILES/InChI labels
- Input is already a fingerprint-bit vector or descriptor table (tokenization applies to raw peak lists, not to precomputed descriptors)

## Inputs

- NIST .msp MS/MS spectral library files (MoNA, GNPS, LipidMaps, NIST 20 exports) with peak lists (m/z intensity pairs) and PrecursorMZ row entries
- Optional per-molecule metadata in MSP blocks: InChI/SMILES labels, custom Fingerprint bit strings, Precursor_type, Ion_mode, Collision_energy
- YAML model-configuration file specifying training/prediction parameters

## Outputs

- Tokenized spectra: sequences of m/z-interval tokens (0.01 Da steps) paired with unit-sum-normalized intensities
- Intensity-weighted sinusoidal positional encodings for the transformer encoder input
- For spectra-to-fingerprint: a fixed-length fingerprint-bit probability vector after SoftMax

## How to apply

Parse MS/MS peak lists from .msp blocks (which must contain PrecursorMZ entries, and may carry InChI/SMILES and Fingerprint rows) and bin m/z values into tokens using constant m/z interval steps (0.01 Da steps for the MS2Fingerprint model). Normalize each spectrum's intensity vector to a unit sum of total intensities, then sort the intensity values by maximum intensity. Build sinusoidal positional encodings scaled by the normalized peak intensities so the encoder receives both spectral position and intensity information. Feed the tokenized spectra into a PyTorch transformer encoder-decoder (4 layers, 2 attention heads per stack) whose output head is a linear layer plus SoftMax over the fingerprint-bit vocabulary, trained with cross-entropy loss with label smoothing and the Adam optimizer. Rationale: the paper explicitly frames spectrum-to-descriptor conversion as token translation — token A (m/z and intensity) translated into token B (descriptors or SMILES) — following the 'Attention is all you need' architecture.

## Related tools

- **IDSL_MINT** (Framework that performs the .msp parsing, m/z tokenization, intensity normalization, and transformer training/inference via YAML-configured MINT_workflow runs) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Implements the transformer encoder-decoder stack, cross-entropy training loop with label smoothing, Adam optimizer, and beam-search decoding; device-agnostic with cuda support) — https://github.com/pytorch
- **RDKit** (Converts InChI and SMILES row entries in MSP blocks into canonical SMILES and computes ECFP/MACCS fingerprint labels used as target tokens) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Each tokenized spectrum corresponds to 0.01 Da m/z bins, and the per-spectrum intensity vector sums to 1 (unit-sum normalization)
- Smoke test passes script_runs: module instantiation, forward/backward pass on a sample .msp-derived batch, and one beam-search decode complete without error
- The spectra-to-fingerprint output head emits a fixed-length fingerprint-bit probability vector after SoftMax
- Model weights are stored/updated in the designated directory on a decreasing training-loss trajectory
- Tokenizer handles MoNA/GNPS-style .msp files, including 'comment: '/'comments: ' entries and longest-content selection for duplicate headers

## Limitations

- MSP parsing constraints: blocks must include PrecursorMZ rows; when multiple similar headers exist, only the longest-content one is parsed, which may silently drop shorter annotations
- Tokenization depends on the .msp reference library (LipidMaps case study used only 6.96% and 4.58% of compounds in ESI+ and ESI− respectively), so chemical classes absent from the training library will not tokenize into meaningful descriptors
- Device/configuration must be declared in the YAML and PyTorch installed for the target device; incompatible setups will fail before tokenization runs

## Evidence

- [intro] Spectrum-to-descriptor framing as token translation: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [full_text] Tokenization details (m/z intervals and intensity normalization): [evidence span withheld — anchor status: not_found]
- [readme] MSP input requirements: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] MSP parsing breadth for public libraries: "IDSL_MINT can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS, and other public library"
- [readme] Training-loop behavior tied to tokenized inputs: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
