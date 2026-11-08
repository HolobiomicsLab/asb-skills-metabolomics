---
name: fingerprint-bit-softmax-classification
description: Use when training or reconstructing a transformer that maps tokenized
  MS/MS peaks (.msp-derived m/z + intensity tokens) to a fixed-length molecular fingerprint-bit
  vector, rather than to SMILES text.
when_to_use_negative:
- The goal is translating MS/MS spectra into canonical SMILES or InChI strings — that
  variant uses autoregressive token decoding (beam search), not a fingerprint-bit
  SoftMax head
- The input is raw LC-MS/MS data (e.g. mzML) rather than extracted, deconvoluted spectra
  in .msp format — use IDSL.CSA-type preprocessing first
- The goal is direct spectral library matching (NIST MS Search, MS-DIAL, LipidBlast)
  rather than descriptor prediction
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: IDSL_MINT
  role: Framework providing the .msp parser, YAML configuration, tokenizer, and the
    ms2fp training/inference workflow wrapping the fingerprint-bit SoftMax model
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Backend for the transformer encoder-decoder, the linear+SoftMax output head,
    cross-entropy/label-smoothing loss, and Adam optimizer
  repo: https://pytorch.org/get-started/locally
- name: RDKit
  role: Computes the target fingerprint bits (ECFP or MACCS Keys) from InChI and SMILES
    row entries in the .msp blocks to define the SoftMax class vocabulary
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:e004d6e67208fb601fca3e0a53a8861c43e99fb8222fe62d9330a9835aa55322
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fingerprint-bit-softmax-classification

## Summary

Implements the spectra-to-fingerprint variant of IDSL_MINT's transformer, where a linear layer followed by a SoftMax head emits class probabilities over a molecular fingerprint-bit vocabulary (ECFP or MACCS keys derived via RDKit), so MS/MS spectra are 'translated' into fingerprint descriptors as a token-translation problem.

## When to use

Use when training or reconstructing a transformer that maps tokenized MS/MS peaks (.msp-derived m/z + intensity tokens) to a fixed-length molecular fingerprint-bit vector, rather than to SMILES text.

## When NOT to use

- The goal is translating MS/MS spectra into canonical SMILES or InChI strings — that variant uses autoregressive token decoding (beam search), not a fingerprint-bit SoftMax head
- The input is raw LC-MS/MS data (e.g. mzML) rather than extracted, deconvoluted spectra in .msp format — use IDSL.CSA-type preprocessing first
- The goal is direct spectral library matching (NIST MS Search, MS-DIAL, LipidBlast) rather than descriptor prediction

## Inputs

- Reference MS/MS library in NIST .msp format (must include PrecursorMZ rows and SMILES/InChI or 'Fingerprint: ' bit-list rows)
- Custom fingerprint definitions (ECFP/MACCS via RDKit or user-supplied fingerprint bits parsed from MSP blocks)
- IDSL_MINT YAML configuration file for the spectra-to-fingerprint (ms2fp) mode

## Outputs

- Trained transformer model weights stored in a designated directory on the decreasing training-loss trajectory
- A fixed-length fingerprint-bit probability vector per input MS/MS spectrum (SoftMax output head)
- Predicted molecular fingerprint descriptors for query MS/MS spectra

## How to apply

Tokenize the .msp peak lists by binning m/z values at constant 0.01 Da interval steps and pair each token with its intensity normalized to a unit sum of total intensities; build sinusoidal positional encodings scaled by these normalized intensities for the encoder. Construct the PyTorch transformer encoder-decoder (4 layers, 2 attention heads each side) and terminate it with a linear layer plus SoftMax over the fingerprint-bit vocabulary, where the target bits come from RDKit ECFP or MACCS Keys computed from the InChI/SMILES rows, or from user-provided 'Fingerprint: ' rows in the .msp blocks. Train with cross-entropy loss using label smoothing and the Adam optimizer, configuring all parameters (number of bits, tokenizer settings, layers) in a YAML file executed via MINT_workflow. The SoftMax head makes this a per-bit multi-class classification formulation, unlike the SMILES variant which decodes token sequences with beam search.

## Related tools

- **IDSL_MINT** (Framework providing the .msp parser, YAML configuration, tokenizer, and the ms2fp training/inference workflow wrapping the fingerprint-bit SoftMax model) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Backend for the transformer encoder-decoder, the linear+SoftMax output head, cross-entropy/label-smoothing loss, and Adam optimizer) — https://pytorch.org/get-started/locally
- **RDKit** (Computes the target fingerprint bits (ECFP or MACCS Keys) from InChI and SMILES row entries in the .msp blocks to define the SoftMax class vocabulary) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- The output head produces a fixed-length fingerprint-bit probability vector for each input spectrum (SoftMax over the bit vocabulary)
- Model weights are stored in the designated directory only along a decreasing training-loss trajectory
- Smoke test passes: module instantiation, forward/backward pass on a sample .msp-derived batch, and (for the sibling variant) a beam-search decode complete without error (script_runs + file_exists for mint_transformer.py)
- Downstream validation: predicted fingerprints searched against LipidMaps yield top-hit true positives; new annotations counted at Tanimoto similarity >= 0.6
- Case-study accuracy is evaluated overall and per lipid class for the ST002044 test study (positive and negative ESI modes trained separately)

## Limitations

- A large proportion (>80%) of MS/MS spectra have no hit in existing libraries, so training libraries assembled for the head must be curated carefully (e.g. spectra overlapping evaluation libraries like LipidBlast and NIST 20 must be excluded)
- MSP blocks must include PrecursorMZ row entries, and IDSL_MINT only parses SMILES:/InChI: labels and comment:/comments: entries; when multiple similar headers exist, the longest content is selected
- Trained models are data-centric and may not generalize to chemical classes absent from the user-provided training set; the case-study training set covered only 6.96% (ESI+) and 4.58% (ESI-) of LipidMaps compounds

## Evidence

- [intro] Framing of spectra-to-descriptor translation: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [abstract] Framework purpose: "Our deep learning framework, Integrated Data Science Laboratory for Metabolomics and Exposomics—Mass INTerpreter (IDSL_MINT) can translate MS/MS spectra into molecular fingerprint descriptors."
- [readme] Fingerprint bit options for the classification head: [evidence span withheld — anchor status: not_found]
- [readme] Weight-storage criterion during training: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
