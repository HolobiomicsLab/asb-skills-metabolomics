---
name: yaml-driven-deep-learning-model-configuration
description: Use when you need to reproduce or retrain IDSL_MINT transformer models
  (fingerprint or SMILES translation) from a deposited .yaml configuration plus .msp
  reference spectra in positive or negative ion mode.
when_to_use_negative:
- Your goal is annotating spectra by mass-spectral library matching (e.g. against
  NIST 20 or LipidBlast) rather than training a new data-centric model
- Your reference spectra are not in .msp format or MSP blocks lack PrecursorMZ and
  SMILES/InChI/Fingerprint entries
- You only have raw instrument data — this skill starts from curated .msp reference
  libraries, not vendor or mzML raw files
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: IDSL_MINT
  role: Framework that parses the YAML, auto-detects trainer vs inference type, and
    runs transformer-based training on .msp inputs
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Backend implementing the transformer model; device-agnostic with cuda GPU
    support
  repo: https://github.com/pytorch
- name: RDKit
  role: Converts InChI/SMILES MSP entries into canonical SMILES and computes ECFP
    or MACCS fingerprint descriptors used as training targets
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:2cfd604aac28a61aefa37ee347fcbb71c710c2e4aa24d25797139ae455b311ea
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# yaml-driven-deep-learning-model-configuration

## Summary

Configure and launch deep-learning model training in IDSL_MINT through commented YAML trainer files, so that transformer models (e.g. ESI+ and ESI− MS2Fingerprint models) are reproducibly trained from deposited .yaml + .msp inputs rather than ad-hoc scripts.

## When to use

Use when you need to reproduce or retrain IDSL_MINT transformer models (fingerprint or SMILES translation) from a deposited .yaml configuration plus .msp reference spectra in positive or negative ion mode.

## When NOT to use

- Your goal is annotating spectra by mass-spectral library matching (e.g. against NIST 20 or LipidBlast) rather than training a new data-centric model
- Your reference spectra are not in .msp format or MSP blocks lack PrecursorMZ and SMILES/InChI/Fingerprint entries
- You only have raw instrument data — this skill starts from curated .msp reference libraries, not vendor or mzML raw files

## Inputs

- Trainer YAML configuration file (e.g. MINT_MS2FP_trainer.yaml) specifying model, fingerprint type, and weight output directory
- Reference MS/MS training spectra in NIST .msp format (MoNA/GNPS public library spectra, split by ESI+ and ESI− mode)
- MSP block metadata: PrecursorMZ (required), SMILES/InChI or user-provided Fingerprint bit string, comment/comments entries

## Outputs

- Trained IDSL_MINT model weight files (one per ionization mode) written to the directory designated in the .yaml
- Training loss trajectory over epochs (qualitatively comparable to Additional file 1: Figure S1)

## How to apply

Download the trainer YAML configurations and input .msp training files (e.g. Zenodo entry [27] file#4 ms2fp_yaml.zip) and edit the MINT_MS2FP_trainer.yaml for the ionization mode of interest, pointing it at the separately compiled positive- or negative-mode MSP set (MoNA/GNPS library spectra, excluding compounds covered by LipidBlast and NIST 20). Ensure each MSP block carries the required rows: PrecursorMZ is mandatory, plus SMILES or InChI (case-insensitive) from which RDKit derives ECFP/MACCS fingerprints, or a user-provided Fingerprint: bit string. Launch training with 'MINT_workflow --yaml /path/to/yaml/file'; the package auto-detects the YAML type (training vs prediction) and, per the README, stores/updates model weights in the designated directory on the decreasing trajectory of the training loss. Budget runtime on the order of the reported case study (~70 h for ESI+, ~45 h for ESI−, reported on 35 CPU processing threads), then repeat the whole configuration step for the other ionization mode.

## Related tools

- **IDSL_MINT** (Framework that parses the YAML, auto-detects trainer vs inference type, and runs transformer-based training on .msp inputs) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Backend implementing the transformer model; device-agnostic with cuda GPU support) — https://github.com/pytorch
- **RDKit** (Converts InChI/SMILES MSP entries into canonical SMILES and computes ECFP or MACCS fingerprint descriptors used as training targets) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Model weight files for both ESI+ and ESI− models exist in the directory designated in the .yaml configuration
- Training loss decreases monotonically over the run (weights stored on the decreasing loss trajectory)
- Training curves qualitatively match Additional file 1: Figure S1 of the paper
- MSP blocks all contain PrecursorMZ and parseable SMILES/InChI (or Fingerprint) rows before training starts
- Runtime is on the order of the reported 70 h (ESI+) / 45 h (ESI−) for comparable hardware

## Limitations

- Reported training durations are long (~70 h ESI+, ~45 h ESI−), so full reproduction is resource-intensive
- The same YAML command handles training and prediction; misconfigured YAML type/paths will silently run the wrong operation unless the yaml type is detectable
- MSP parsing quirks: with multiple similar headers in a block the longest one is chosen; comment/comments entries are the mechanism for MoNA/GNPS metadata

## Evidence

- [readme] YAML auto-detection and command: "The IDSL_MINT package can automatically detect types of `yaml` file to run training or inference operations."
- [readme] Weight storage rationale: "weights of IDSL_MINT models are stored and updated in a designated directory on the decreasing trajectory of the training loss"
- [results] Case-study model training: "Next, we trained two deep learning models (ESI+ and ESI−) using our newly created IDSL.MINT framework."
- [results] Deposited configuration location: "The.yaml file and input.msp files are provided at Zenodo entry [27](file#4 ms2fp_yaml.zip)."
- [readme] MSP metadata handling for public libraries: "IDSL_MINT can extract information from `comment: ` and `comments: ` entries in *.msp* files which enables this platform to process MoNA, GNPS"
- [intro] Implementation framework: "is implemented within the python PyTorch framework"
