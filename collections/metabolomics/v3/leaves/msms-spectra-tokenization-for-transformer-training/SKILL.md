---
name: msms-spectra-tokenization-for-transformer-training
description: Use when reference MS/MS library spectra in NIST .msp format (e.g. compiled
  from MoNA/GNPS in ESI+ or ESI−) need to be converted into tokenized source/target
  sequences to train a transformer that predicts fingerprint descriptors or SMILES.
when_to_use_negative:
- Input is raw LC-MS/MS instrument data rather than extracted .msp spectra — first
  process raw files to .msp (e.g. with IDSL.CSA) before tokenizing
- The goal is direct spectral library matching against NIST 20 or LipidBlast rather
  than data-centric fingerprint prediction from user-provided references
- MSP blocks lack SMILES/InChI (or custom Fingerprint) row entries, so no target token
  sequence can be constructed for training
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3174
tools:
- name: IDSL_MINT
  role: Framework that tokenizes .msp spectra, auto-detects YAML type, and trains/inferences
    transformer models via MINT_workflow
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Backend implementing the transformer ('Attention is all you need') architecture;
    device-agnostic with cuda GPU support
  repo: https://github.com/pytorch
- name: RDKit
  role: Converts InChI and SMILES row entries to canonical SMILES and computes ECFP/MACCS
    Keys fingerprint targets
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:7b1916ea204ee07aed74260eab1b5eed4af9763e3d9d0df7e0d1debd3a423b21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# MS/MS spectra tokenization for transformer training

## Summary

Framing MS/MS-to-descriptor prediction as a language translation task: m/z and intensity pairs (token A) are tokenized from .msp entries and translated into molecular fingerprint descriptors or canonical SMILES (token B) by an IDSL_MINT transformer model built on PyTorch.

## When to use

Use when reference MS/MS library spectra in NIST .msp format (e.g. compiled from MoNA/GNPS in ESI+ or ESI−) need to be converted into tokenized source/target sequences to train a transformer that predicts fingerprint descriptors or SMILES.

## When NOT to use

- Input is raw LC-MS/MS instrument data rather than extracted .msp spectra — first process raw files to .msp (e.g. with IDSL.CSA) before tokenizing
- The goal is direct spectral library matching against NIST 20 or LipidBlast rather than data-centric fingerprint prediction from user-provided references
- MSP blocks lack SMILES/InChI (or custom Fingerprint) row entries, so no target token sequence can be constructed for training

## Inputs

- Reference MS/MS spectra in NIST .msp format (e.g. MoNA and GNPS public library spectra, split by ESI+ / ESI−)
- Trainer YAML configuration file (e.g. MINT_MS2FP_trainer.yaml from Zenodo file#4 ms2fp_yaml.zip)
- Structure annotations per MSP block: SMILES or InChI row entries, or user-provided 'Fingerprint: ' bit lists
- RDKit fingerprint parameter choice (ECFP or MACCS Keys)

## Outputs

- Trained IDSL_MINT transformer model weight files for each ionization mode (ESI+ and ESI−), stored in the directory specified in the YAML
- Training loss trajectories (qualitatively comparable to Additional file 1: Figure S1)
- A model that translates MS/MS spectra into molecular fingerprint descriptors or canonical SMILES

## How to apply

Compile the training corpus as .msp files, separately per ionization mode (the case study built ESI+ and ESI− sets from MoNA and GNPS library spectra), and ensure every MSP block contains a PrecursorMZ row plus a SMILES or InChI row, which RDKit converts to canonical SMILES. Choose the target representation: RDKit-derived Extended-connectivity fingerprints (ECFPs) or MACCS Keys computed from InChI/SMILES, or user-provided fingerprint bits parsed from the 'Fingerprint: ' row of the .msp block. Treat the spectrum as token A (m/z and intensity) and the descriptors as token B, implementing the seq2seq mapping with the transformer architecture of 'Attention is all you need' in the PyTorch backend. Parameterize training in a well-documented trainer YAML file (from Zenodo entry file#4 ms2fp_yaml.zip for the case study) and launch it with MINT_workflow, which auto-detects the YAML type; weights are stored and updated in the designated directory along the decreasing training-loss trajectory. Expect long runtimes on GPU hardware — the authors reported ~70 h for ESI+ and ~45 h for ESI− using 35 CPU processing threads. IDSL_MINT also parses 'comment: '/'comments: ' entries, enabling direct use of MoNA and GNPS libraries without pre-treatment.

## Related tools

- **IDSL_MINT** (Framework that tokenizes .msp spectra, auto-detects YAML type, and trains/inferences transformer models via MINT_workflow) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Backend implementing the transformer ('Attention is all you need') architecture; device-agnostic with cuda GPU support) — https://github.com/pytorch
- **RDKit** (Converts InChI and SMILES row entries to canonical SMILES and computes ECFP/MACCS Keys fingerprint targets) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Model weight files for both ESI+ and ESI− models are stored in the designated YAML directory
- Training loss follows a monotonically decreasing trajectory as weights are updated
- Training curves are qualitatively comparable to Additional file 1: Figure S1 of the paper
- Every MSP block passed to the trainer contains a PrecursorMZ row; SMILES/InChI entries convert to canonical SMILES via RDKit without errors
- Reported runtimes are on the order of 70 h (ESI+) and 45 h (ESI−) on comparable hardware (35 CPU threads, GPU)

## Limitations

- MSP blocks must include PrecursorMZ row entries; blocks without it cannot be processed
- If multiple similar headers occur in an MSP block, IDSL_MINT selects the one with the longest content, which may not be the intended annotation
- Training is computationally expensive (tens of hours per model), and models trained on one compiled dataset (e.g. ~7% of LipidMaps in ESI+) may exclude chemical classes of interest

## Evidence

- [intro] Tokenization framing: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [readme] Transformer architecture basis: "This innovative approach for mass spectrometry data processing has been constructed upon the transformer models delineated in the seminal paper"
- [readme] MSP block requirements: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] Fingerprint target options: "This method offers the option to calculate fingerprints using the [Extended-connectivity fingerprints (ECFPs)](https://doi.org/10.1021/ci100050t) or [MACCS Keys]"
- [results] Training set compilation by ion mode: "Spectra from MoNA and GNPS public libraries in positive and negative modes were used separately to create training sets."
- [readme] Weight storage on decreasing loss: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value"
