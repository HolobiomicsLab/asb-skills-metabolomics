---
name: casmi2016-benchmark-evaluation
description: Use when a trained MS2-to-structure model needs standardized head-to-head
  comparison on the CASMI 2016 Challenge set, e.g. to report direct ISMILES accuracy
  and fingerprint-similarity top-1 ranking accuracy on positive-mode spectra.
when_to_use_negative:
- Data are real-world environmental samples spiked with the 96-pollutant mixture rather
  than the CASMI 2016 benchmark — use the sample-level evaluation (77.1% vs SIRIUS/TraceFinder)
  instead.
- Input spectra are negative-ion mode or lack the high-resolution two-decimal mass
  accuracy required by MSThunder's dataset rules.
- Only raw vendor files (ThermoFisher, Agilent) are available and cannot be converted
  via MSConvert, since the current version does not support offline raw-data processing.
edam_operation: http://edamontology.org/operation_0223
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSThunder
  role: Transformer model performing direct formula-to-ISMILES prediction and candidate
    ranking for each benchmark spectrum
  repo: https://github.com/LQZ0123/MSThunder
- name: PubChemPy
  role: Retrieves candidate structures from PubChem for each confirmed precursor formula
    and converts SMILES to ISMILES
- name: RDKit
  role: Computes weighted molecular fingerprints used to score and rank PubChem candidate
    structures against the predicted ISMILES
- name: Python 3.10.5
  role: Runtime environment for the MSThunder model and interface (PyTorch 1.13.0+cu116,
    pubchempy 1.0.4, rdkit 2023.9.2)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.enceco.2025.07.022
    title: 'MSThunder: A deep learning-based nontargeted analytical framework for
      the accurate and rapid identification of unknown organic pollutants in water'
  source_content_hash: sha256:8fb71c516c0e11731683ebab59a1d91a8b5cd273b1ca40b1ac43906ec3443b5c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LQZ0123/MSThunder
  repo_url: https://github.com/LQZ0123/MSThunder
  tools:
  - MSConvert
  - MSThunder
  - PubChemPy
  - Python 3.10.5
  - RDKit
  edam_operation: http://edamontology.org/operation_0223
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# casmi2016-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a formula-to-ISMILES transformer model (MSThunder) on the CASMI 2016 Challenge positive-mode benchmark of 127 MS2 spectra, reporting direct-prediction accuracy (52.8%) and top-1 candidate-ranking accuracy (59.1%) against ground-truth structures.

## When to use

Use when a trained MS2-to-structure model needs standardized head-to-head comparison on the CASMI 2016 Challenge set, e.g. to report direct ISMILES accuracy and fingerprint-similarity top-1 ranking accuracy on positive-mode spectra.

## When NOT to use

- Data are real-world environmental samples spiked with the 96-pollutant mixture rather than the CASMI 2016 benchmark — use the sample-level evaluation (77.1% vs SIRIUS/TraceFinder) instead.
- Input spectra are negative-ion mode or lack the high-resolution two-decimal mass accuracy required by MSThunder's dataset rules.
- Only raw vendor files (ThermoFisher, Agilent) are available and cannot be converted via MSConvert, since the current version does not support offline raw-data processing.

## Inputs

- 127 CASMI 2016 Challenge positive-mode MS2 spectra (batch-processed file format from MSThunder)
- Confirmed precursor formula for each spectrum
- CASMI 2016 ground-truth chemical structures
- PubChem candidate structures retrieved per precursor formula (SMILES)
- Pretrained MSThunder transformer model weights (msthunder.rar + msthunderfile.rar from Zenodo)

## Outputs

- Per-spectrum predicted ISMILES sequence
- Ranked candidate list per spectrum scored by weighted molecular fingerprint similarity
- Aggregate direct-prediction accuracy (target 52.8%)
- Aggregate top-1 ranking accuracy (target 59.1%)
- Results table with per-spectrum prediction outcomes and rankings

## How to apply

Load the 127 CASMI 2016 positive-mode challenge spectra from the MSThunder batch-processed file data (msthunderfile.rar from Zenodo 10.5281/zenodo.12602805, password '2024V1.0'). For each spectrum, derive the precursor formula, then run the trained transformer (torch 1.13.0+cu116, Python 3.10.5) to predict an Isomeric SMILES (ISMILES) sequence directly from the formula. Retrieve candidate structures from PubChem via PubChemPy for each confirmed precursor formula and convert their SMILES to standardized ISMILES. Compute weighted molecular fingerprints (RDKit 2023.9.2) for the predicted ISMILES and each candidate, rank candidates by fingerprint similarity, and record the top-1 hit. Score each spectrum as (a) direct prediction correct — predicted ISMILES matches the ground-truth structure — and (b) top-1 ranking correct; expected aggregates are 52.8% and 59.1% respectively, with a prediction speed around 0.32 s per spectrum as the reference throughput.

## Related tools

- **MSThunder** (Transformer model performing direct formula-to-ISMILES prediction and candidate ranking for each benchmark spectrum) — https://github.com/LQZ0123/MSThunder
- **PubChemPy** (Retrieves candidate structures from PubChem for each confirmed precursor formula and converts SMILES to ISMILES)
- **RDKit** (Computes weighted molecular fingerprints used to score and rank PubChem candidate structures against the predicted ISMILES)
- **Python 3.10.5** (Runtime environment for the MSThunder model and interface (PyTorch 1.13.0+cu116, pubchempy 1.0.4, rdkit 2023.9.2))

## Examples

```
python -c "import pubchempy, rdkit, torch; from msthunder import predict; print(predict('CASMI2016_positive_batch.pkl', ion_mode='Positive'))"
```

## Evaluation signals

- Direct-prediction accuracy on the 127 positive-mode CASMI 2016 spectra should approach 52.8%.
- Top-1 ranking accuracy after fingerprint-similarity scoring should approach 59.1%.
- Prediction speed should be near 0.32 s per spectrum, matching the reported throughput.
- Each predicted ISMILES must be a valid isomeric SMILES within the 130-character length limit applied during training.
- The ranked candidate list should contain PubChem structures matching the confirmed precursor formula.

## Limitations

- Benchmark restricted to positive-ion mode CASMI 2016 spectra; negative-mode performance is not covered here.
- Only the top-1 ranking result is reported, so near-miss candidates (top-2/3 correct) are not captured in the headline metric.
- Offline processing of raw data is unsupported; spectra must already be in batch-processed form.
- ISMILES predictions rely on confirmed precursor formulas; formula errors propagate to all downstream candidates.

## Evidence

- [abstract] Direct top-1 prediction accuracy of 52.8%: "MSThunder achieved 59.1 % top-1 accuracy in the benchmark (CASMI 2016 Challenge), outperforming other major existing models."
- [other] ISMILES filter length rule: "ii) SMILES that cannot be replaced by ISMILES and/or ISMILES with a character length exceeding 130 were excluded"
- [other] Candidate retrieval workflow: "Upon initial confirmation of the precursor formula, it was utilized for structural (ISMILES) retrieval in PubChem via PubChemPy"
- [readme] Runtime environment: "The model and interface of MSThunder were written by Python 3.10.5 with associated packages, consisting of PySide6 6.6.0, torch 1.13.0+cu116"
- [other] Benchmark validation framing: "The method validation was conducted by the benchmark dataset (CASMI 2016 Challenge)"
