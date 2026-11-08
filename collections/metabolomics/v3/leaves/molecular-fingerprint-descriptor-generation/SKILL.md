---
name: molecular-fingerprint-descriptor-generation
description: Use when compiling training targets for MS/MS-to-fingerprint models from
  .msp reference libraries (e.g., MoNA/GNPS positive and negative mode spectra) whose
  blocks carry InChI, SMILES, or explicit Fingerprint rows.
when_to_use_negative:
- When the goal is the reverse direction — predicting MS/MS spectra from canonical
  SMILES (fingerprint-to-spectrum) — use IDSL_MINT's spectrum-forecasting mode instead
- When experimental spectra can already be annotated by direct library search against
  NIST 20 or LipidBlast (e.g., 638/3386 ESI+ spectra of ST002044 matched), fingerprint
  prediction is unnecessary
- When MSP blocks lack PrecursorMZ row entries or any InChI/SMILES/Fingerprint labels,
  descriptor generation cannot proceed
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: IDSL_MINT
  role: Framework that parses the .msp blocks, consumes the generated fingerprint
    descriptors as training targets, and trains the ESI+/ESI- transformer models
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI and SMILES row entries to canonical SMILES and calculates ECFP
    or MACCS Keys molecular fingerprints from them
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:3954c615bcddfc80a959372926873a5a4122d54e3b495fbf2a1a8813c151c49f
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# molecular-fingerprint-descriptor-generation

## Summary

Generation of molecular fingerprint descriptor targets (RDKit ECFP, MACCS Keys, or user-supplied bit strings) from InChI/SMILES entries in .msp reference MS/MS blocks, as required to train IDSL_MINT MS2-fingerprint transformer models.

## When to use

Use when compiling training targets for MS/MS-to-fingerprint models from .msp reference libraries (e.g., MoNA/GNPS positive and negative mode spectra) whose blocks carry InChI, SMILES, or explicit Fingerprint rows.

## When NOT to use

- When the goal is the reverse direction — predicting MS/MS spectra from canonical SMILES (fingerprint-to-spectrum) — use IDSL_MINT's spectrum-forecasting mode instead
- When experimental spectra can already be annotated by direct library search against NIST 20 or LipidBlast (e.g., 638/3386 ESI+ spectra of ST002044 matched), fingerprint prediction is unnecessary
- When MSP blocks lack PrecursorMZ row entries or any InChI/SMILES/Fingerprint labels, descriptor generation cannot proceed

## Inputs

- NIST .msp-format reference MS/MS library files (e.g., positive/negative-mode training sets from MoNA and GNPS) with InChI/SMILES or Fingerprint row entries
- User-defined fingerprint descriptor definition (ECFP or MACCS Keys parameters) declared in the trainer YAML configuration

## Outputs

- Molecular fingerprint descriptor vectors (ECFP or MACCS bit sets or user-defined bit positions) paired with each MS/MS spectrum
- Fingerprint targets consumed by IDSL_MINT transformer training to produce exported model weight files

## How to apply

Parse the reference .msp training blocks (compiled from MoNA and GNPS in positive and negative modes) and convert the InChI and SMILES row entries into canonical SMILES using RDKit. Then calculate the molecular fingerprints either with RDKit ECFP (extended-connectivity) or MACCS Keys methods, or, alternatively, parse user-provided `Fingerprint: ` bit-position entries directly from the MSP blocks (e.g., the Aspirin example lists 60 bit positions such as 15-53-85-157-246...). Labels are read case-insensitively (`SMILES: ` / `InChI: `), and when multiple similar headers occur in a block, the one with the longest content is used. The resulting descriptor vectors become token-B targets of the 'language translation' framing (m/z + intensity tokens → descriptor tokens) for the transformer models configured via MINT_MS2FP_trainer.yaml; note that every MSP block must include a `PrecursorMZ: ` row entry for parsing to succeed.

## Related tools

- **IDSL_MINT** (Framework that parses the .msp blocks, consumes the generated fingerprint descriptors as training targets, and trains the ESI+/ESI- transformer models) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI and SMILES row entries to canonical SMILES and calculates ECFP or MACCS Keys molecular fingerprints from them) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/MINT_MS2FP_trainer.yaml
```

## Evaluation signals

- Every parsed MSP block yields a fingerprint vector whose bit count matches the configured descriptor (ECFP, MACCS Keys, or the user-provided bit list)
- Fingerprints regenerated independently with RDKit from the block's InChI/SMILES match the Fingerprint bit positions in the .msp (cf. the Aspirin example block)
- Downstream transformer training loss follows a monotonically decreasing trajectory and weights are stored in the YAML-designated directory
- Training curves are qualitatively comparable to Additional file 1: Figure S1 of the paper (70 h for ESI+, 45 h for ESI-)

## Limitations

- MSP blocks must include `PrecursorMZ: ` row entries; blocks missing this row cannot be parsed
- Only `comment: ` and `comments: ` entries are used for metadata, so MoNA/GNPS public libraries may need pre-treatment before parsing
- When multiple similar headers occur within an MSP block, the longest-content header is silently selected, which may surprise users expecting the first header to be used

## Evidence

- [methods] InChI/SMILES to canonical SMILES via RDKit: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] ECFP or MACCS or user-provided fingerprints: "RDKit methods from InChI and SMILES row entries. Another option to obtain molecular fingerprints is to parse the MSP files for the user-provided fingerprints."
- [readme] Case-insensitive structure labels in MSP: "IDSL_MINT identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] PrecursorMZ required in MSP blocks: "MSP blocks must include `PrecursorMZ: ` row entries."
- [abstract] Customizable fingerprint descriptors for training: "Models are trained on user-provided reference MS/MS libraries via any customizable molecular fingerprint descriptors."
