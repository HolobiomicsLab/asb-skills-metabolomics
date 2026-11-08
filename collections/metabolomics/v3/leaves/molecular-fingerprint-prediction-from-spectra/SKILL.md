---
name: molecular-fingerprint-prediction-from-spectra
description: Use when an unidentified analyte's GC/EI-MS spectrum has no match in
  reference libraries (NIST, Wiley, SWGDRUG) and you need to compute its molecular
  fingerprint without a known chemical structure to rank candidate NPS identities.
when_to_use_negative:
- The analyte's chemical structure is already known and can be encoded with ECFP fingerprints
  directly via RDKit
- The analyte has a confirmed match in the NIST, Wiley, or SWGDRUG reference spectrum
  libraries, so conventional library screening suffices
- You need to predict a mass spectrum from a known structure (use NEIMS instead, which
  requires a known SMILES)
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepEI
  role: Predicts the molecular fingerprint of the unknown analyte directly from its
    mass spectrum, and computes fingerprints for compounds in the synthetic NPS database
    so both sides can be compared in fingerprint space
  repo: https://github.com/jhhung/PS2MS
- name: PS2MS
  role: Orchestrating pipeline that runs DeepEI, merges fingerprints into the msp
    file, and combines them with NEIMS spectra for the integrated similarity score
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Python package required to build DeepEI from source and install via conda
    for fingerprint handling
  repo: https://www.rdkit.org/docs/Install.html
- name: NEIMS
  role: Predicts mass spectra of enumerated derivatives; its outputs feed into the
    same integrated similarity metric alongside DeepEI fingerprints
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:e697fda369bbae88f32f0fc572e9e53911709e3cacea615a63c7221543aea732
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# Molecular fingerprint prediction from spectra

## Summary

Use the DeepEI deep learning model to convert an analyte's GC/EI-MS mass spectrum directly into its molecular fingerprint without knowing its chemical structure, enabling similarity-based comparison against predicted fingerprints of enumerated NPS candidates.

## When to use

Use when an unidentified analyte's GC/EI-MS spectrum has no match in reference libraries (NIST, Wiley, SWGDRUG) and you need to compute its molecular fingerprint without a known chemical structure to rank candidate NPS identities.

## When NOT to use

- The analyte's chemical structure is already known and can be encoded with ECFP fingerprints directly via RDKit
- The analyte has a confirmed match in the NIST, Wiley, or SWGDRUG reference spectrum libraries, so conventional library screening suffices
- You need to predict a mass spectrum from a known structure (use NEIMS instead, which requires a known SMILES)

## Inputs

- Unidentified analyte's GC/EI-MS mass spectrum (70 eV electron energy)
- Mass spectra of enumerated NPS candidate compounds in the synthetic database
- NEIMS-predicted mass spectra of enumerated derivatives
- Drug core structure (e.g., cathinone) used for derivative enumeration

## Outputs

- Predicted molecular fingerprint for each unidentified analyte
- Fingerprints merged into the .msp file alongside analyte spectra
- Fingerprint similarity component of the SMSF score for each analyte-candidate pair
- Ranked list of the hundred most similar compounds per analyte

## How to apply

Run the DeepEI model on each unidentified analyte's mass spectrum to convert it into a molecular fingerprint, following the drug-detection step of the PS2MS pipeline. DeepEI is used because it maps spectra-to-fingerprints, complementing NEIMS which requires a known SMILES structure. The predicted fingerprint is then merged into the .msp file alongside the analyte's spectrum so the integrated similarity metric (SMSF) can combine mass-spectrum similarity (cosine and Jaccard with top-intensity peak overlap and ±1 Da tolerance) with fingerprint similarity. DeepEI is also applied to the compounds in the synthetic NPS database so both sides of the comparison are in fingerprint space. Evaluate fingerprint quality indirectly via top-20 recovery: over 90% (NIST) and 85% (SWGDRUG) of test substances should rank within the top 20 SMSF predictions.

## Related tools

- **DeepEI** (Predicts the molecular fingerprint of the unknown analyte directly from its mass spectrum, and computes fingerprints for compounds in the synthetic NPS database so both sides can be compared in fingerprint space) — https://github.com/jhhung/PS2MS
- **PS2MS** (Orchestrating pipeline that runs DeepEI, merges fingerprints into the msp file, and combines them with NEIMS spectra for the integrated similarity score) — https://github.com/jhhung/PS2MS
- **rdkit** (Python package required to build DeepEI from source and install via conda for fingerprint handling) — https://www.rdkit.org/docs/Install.html
- **NEIMS** (Predicts mass spectra of enumerated derivatives; its outputs feed into the same integrated similarity metric alongside DeepEI fingerprints)

## Examples

```
See DeepEI/README.md — the repository documents how to install rdkit from conda and run DeepEI to predict fingerprints, then merge them into the .msp file before running drug-detection.
```

## Evaluation signals

- Reproduce the reported SMSF top-20 recovery: >90% on the NIST dataset and >85% on the SWGDRUG dataset
- Verify fingerprints were successfully merged into the .msp file (DeepEI README step) before running drug detection
- Top-1 correct-prediction proportion should rise from below 40% to more than 50% once fingerprint similarity is incorporated via Jaccard
- Check that the drug-detection output contains the hundred most similar compounds ranked by similarity score

## Limitations

- Fingerprint prediction quality degrades for analytes far outside the training distribution; NEIMS was originally trained solely using spectra from the NIST library
- The system is designed for scenarios where the chemical structure of the NPS is not already known; direct structure prediction (e.g., DarkNPS) is still in early stages
- Requires python 3.6.9 or higher, rdkit installed from conda, and building the C++ code from source (g++-10+, CMake 3.16.0+)

## Evidence

- [methods] DeepEI converts analyte spectrum to fingerprint without explicit structure information: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [readme] System leverages two deep learning tools for spectra and fingerprints: "The system leverages two deep learning tools, NEIMS and DeepEI, to generate mass spectra and chemical fingerprints, respectively."
- [readme] DeepEI fingerprints merged into msp file: "The system also use the function of DeepEI to calculate the fingerprint of compounds of synthetic database. The system merges the fingerprint into the msp file."
- [methods] Top-1 prediction proportion rises after incorporating Jaccard similarity: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] Top-20 SMSF recovery rates on NIST and SWGDRUG datasets: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
