---
name: transformer-seq2seq-model-training
description: Use when you have reference MS/MS spectra in .msp format (e.g. from MoNA
  or GNPS) plus SMILES/InChI or fingerprint entries, and need to train a custom data-centric
  model rather than reuse an external pretrained one.
when_to_use_negative:
- You only need to run inference/prediction with an already-trained model — use a
  prediction YAML instead of a trainer YAML
- 'Your spectra lack PrecursorMZ: entries or usable SMILES/InChI labels in the MSP
  blocks — IDSL_MINT parsing depends on these rows'
- The goal is direct library annotation with curated commercial libraries (e.g. NIST
  20, LipidBlast) rather than training a custom model — library matching was done
  separately from model training in this workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IDSL_MINT
  role: Framework that executes the transformer training from a YAML config, parsing
    .msp inputs and storing model weights on the decreasing loss trajectory
  repo: https://github.com/idslme/IDSL_MINT
- name: PyTorch
  role: Deep learning backend implementing the transformer model; cuda GPU processing
    supported and device-agnostic
- name: RDKit
  role: Converts InChI/SMILES row entries in MSP blocks to canonical SMILES and computes
    ECFP/MACCS fingerprint descriptors used as training targets
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:6e0091d1f914bd66f73e24a18f0945d6adb43eb787f1f839f68cece5a89e75d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Transformer-seq2seq model training

## Summary

Train transformer (attention-based) seq2seq models with IDSL_MINT that translate tokenized MS/MS spectra (m/z–intensity pairs) into target tokens such as molecular fingerprint descriptors, using YAML-configured runs over .msp reference libraries.

## When to use

Use when you have reference MS/MS spectra in .msp format (e.g. from MoNA or GNPS) plus SMILES/InChI or fingerprint entries, and need to train a custom data-centric model rather than reuse an external pretrained one.

## When NOT to use

- You only need to run inference/prediction with an already-trained model — use a prediction YAML instead of a trainer YAML
- Your spectra lack PrecursorMZ: entries or usable SMILES/InChI labels in the MSP blocks — IDSL_MINT parsing depends on these rows
- The goal is direct library annotation with curated commercial libraries (e.g. NIST 20, LipidBlast) rather than training a custom model — library matching was done separately from model training in this workflow

## Inputs

- Reference MS/MS spectra in NIST .msp format (with PrecursorMZ:, Ion_mode:, SMILES:/InChI: entries) from MoNA/GNPS public libraries
- Trainer YAML configuration file (e.g. MINT_MS2FP_trainer.yaml) specifying fingerprint type, training set path, and weight output directory
- Optionally user-provided fingerprint bit lists in 'Fingerprint: ' rows of the MSP blocks

## Outputs

- Trained IDSL_MINT model weight files stored in the YAML-designated directory (one model per ionization mode, e.g. ESI+ and ESI−)
- Training loss trajectory (expected to decrease monotonically; comparable to Additional file 1: Figure S1)

## How to apply

Frame spectra-to-descriptor prediction as a token-translation problem (token A = m/z and intensity, token B = descriptors) and train a transformer model ('Attention is all you need' architecture) on the reference library. Extract trainer YAML configurations (e.g. from Zenodo file#4 ms2fp_yaml.zip), point MINT_MS2FP_trainer.yaml at the positive- or negative-mode MSP set, and run the package via 'MINT_workflow --yaml /path/to/yaml/file' — the package auto-detects the YAML type for training vs inference. Compile mode-specific training sets separately (e.g. ESI+ and ESI− from MoNA and GNPS public library spectra) so each model sees one ionization mode. IDSL_MINT converts InChI/SMILES row entries to canonical SMILES via RDKit, requires PrecursorMZ: entries in each MSP block, and can parse custom 'Fingerprint: ' bit lists or ECFP/MACCS RDKit fingerprints. Model weights are stored and updated in a designated directory along the decreasing trajectory of the training loss; the authors reported ~70 h (ESI+) and ~45 h (ESI−) of training on 35 processing threads with GPU support.

## Related tools

- **IDSL_MINT** (Framework that executes the transformer training from a YAML config, parsing .msp inputs and storing model weights on the decreasing loss trajectory) — https://github.com/idslme/IDSL_MINT
- **PyTorch** (Deep learning backend implementing the transformer model; cuda GPU processing supported and device-agnostic)
- **RDKit** (Converts InChI/SMILES row entries in MSP blocks to canonical SMILES and computes ECFP/MACCS fingerprint descriptors used as training targets) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/MINT_MS2FP_trainer.yaml
```

## Evaluation signals

- Model weight files for both ESI+ and ESI− models are present in the directory designated by the YAML config
- Training loss decreases monotonically over the run, with trajectories qualitatively comparable to Additional file 1: Figure S1
- Training durations are on the same order as reported (≈70 h for ESI+ and ≈45 h for ESI− on 35 processing threads)
- MSP blocks parsed correctly: every block contains a PrecursorMZ: entry and SMILES:/InChI: labels are detected case-insensitively

## Limitations

- Training is computationally heavy — reported runtimes of ~70 h (ESI+) and ~45 h (ESI−) per model
- InChI/SMILES must be present in MSP row entries or supplied as custom 'Fingerprint: ' bit lists; without them the descriptor targets cannot be generated
- MSP compatibility requires 'PrecursorMZ: ' entries; MoNA/GNPS comment: /comments: entries are parsed, but multiple similar headers fall back to the longest content

## Evidence

- [results] Training of two ionization-mode transformer models: "Next, we trained two deep learning models (ESI+ and ESI−) using our newly created IDSL.MINT framework."
- [intro] Spectra-to-descriptor framed as token translation: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
- [readme] CLI invocation and YAML auto-detection: "run the below bash command to perform calculations. The **IDSL_MINT** package can automatically detect types of `yaml` file to run training or inference operations."
- [readme] Weight storage tied to decreasing loss: "In the training step, weights of **IDSL_MINT** models are stored and updated in a designated directory on the decreasing trajectory of the training loss value to ensure optimal performance and"
- [other] Training resource reporting (threads): [evidence span withheld — anchor status: not_found]
