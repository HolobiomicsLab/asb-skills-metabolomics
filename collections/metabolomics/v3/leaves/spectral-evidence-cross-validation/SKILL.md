---
name: spectral-evidence-cross-validation
description: Use when a SMART 2.0 (or comparable NMR-based CNN) class probability
  assignment must be verified against the underlying 1D/2D NMR and LC-MS/MS spectra
  before the compound annotation is accepted.
when_to_use_negative:
- When the compound structure is already confirmed — e.g., the planar structure has
  already been confirmed by 1D/2D NMR and LC-MS2 analysis — no cross-validation of
  a predicted class is needed.
- When only SMILES strings (not spectra) are available and the goal is programmatic
  structural classification; use the NP Classifier /classify?smiles= API instead of
  spectral evidence checking.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SMART 2.0
  role: NMR-based convolutional neural network classifier whose top class predictions
    for each compound spectrum are cross-validated against 1D/2D NMR and LC-MS/MS
    evidence
- name: molecular networking
  role: MS2-based molecular networking used alongside SMART 2.0 to prioritize and
    guide fractionation, providing MS/MS context for annotation checks
- name: atomic sort
  role: Recently appearing method against which the SMART 2.0 class assignments of
    the cyclic peptides were benchmarked
- name: NP Classifier
  role: Deployed classifier service (via /classify?smiles= endpoint) providing the
    natural-product class ontology underlying assignments; dockerized with docker
    and docker-compose
  repo: github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:d0d77b7cdc10679fcd571c3acb6c55821d8e3249fa4d9df09c5c136e7afe39aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NP Classifier
  - SMART 2.0
  - atomic sort
  - molecular networking
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Spectral Evidence Cross-Validation

## Summary

Validates machine-learning natural-product class assignments (e.g., SMART 2.0 NMR-based CNN predictions) by cross-checking each predicted class against the raw supporting spectral evidence — 1D/2D NMR and LC-MS/MS data — before committing the annotation.

## When to use

Use when a SMART 2.0 (or comparable NMR-based CNN) class probability assignment must be verified against the underlying 1D/2D NMR and LC-MS/MS spectra before the compound annotation is accepted.

## When NOT to use

- When the compound structure is already confirmed — e.g., the planar structure has already been confirmed by 1D/2D NMR and LC-MS2 analysis — no cross-validation of a predicted class is needed.
- When only SMILES strings (not spectra) are available and the goal is programmatic structural classification; use the NP Classifier /classify?smiles= API instead of spectral evidence checking.

## Inputs

- Cyclic-peptide (or macrolide) compound spectra used for SMART 2.0 analysis (NMR spectra for CNN classification)
- Supporting 1D/2D NMR data for each compound
- Supporting LC-MS/MS (MS2) data for each compound
- SMART 2.0 class-probability predictions per compound

## Outputs

- Per-compound annotation/characterization table with columns: compound name, SMART 2.0 class assignment, supporting spectral evidence
- cyclic_peptide_classifications.csv

## How to apply

For each compound spectrum (e.g., the cyclic-peptide case-study spectra), run the spectrum through the SMART 2.0 NMR-based convolutional neural network classifier and record the top predicted natural-product class. Then cross-check that top assignment against the supporting spectral evidence described in the case study — 1D/2D NMR signals and LC-MS/MS fragments that substantiate the structural features the class implies (e.g., planar structure confirmation, as done for the swinholide/luminaolide hybrid symplocolide A). Optionally benchmark the assignment against the alternative 'atomic sort' method for the same spectra. Assemble a per-compound table with columns for compound name, SMART 2.0 class assignment, and supporting spectral evidence, then write the result to cyclic_peptide_classifications.csv. The rationale is that combined traditional (manual NMR/MS interpretation) and deep-learning-assisted annotation guards against uncritical acceptance of CNN output, which the paper frames as overcoming longstanding challenges in natural products drug discovery.

## Related tools

- **SMART 2.0** (NMR-based convolutional neural network classifier whose top class predictions for each compound spectrum are cross-validated against 1D/2D NMR and LC-MS/MS evidence)
- **molecular networking** (MS2-based molecular networking used alongside SMART 2.0 to prioritize and guide fractionation, providing MS/MS context for annotation checks)
- **atomic sort** (Recently appearing method against which the SMART 2.0 class assignments of the cyclic peptides were benchmarked)
- **NP Classifier** (Deployed classifier service (via /classify?smiles= endpoint) providing the natural-product class ontology underlying assignments; dockerized with docker and docker-compose) — github.com/mwang87/NP-Classifier

## Examples

```
curl "http://localhost/classify?smiles=<SMILES>"
```

## Evaluation signals

- Every compound in the output table has a recorded top SMART 2.0 class assignment and a non-empty supporting spectral evidence entry.
- Each accepted class assignment is consistent with independently confirmed structural features, as demonstrated for symplocolide A whose planar structure was confirmed as a swinholide A / luminaolide B hybrid by 1D/2D NMR and LC-MS2 analysis.
- The cyclic-peptide assignments agree with the benchmark comparison against the 'atomic sort' method.
- Output file cyclic_peptide_classifications.csv contains exactly the three required columns: compound name, SMART 2.0 class assignment, supporting spectral evidence.
- Compounds annotated in the study (swinholide A, samholides A–I, and new derivatives) are all represented in the table.

## Limitations

- SMART 2.0 is applied for mixture analysis; predicted classes on incompletely fractionated extracts may require manual deconvolution before cross-validation is meaningful.
- The classifier deployment depends on fixed model layer names ('input_2048', 'input_4096' as inputs, 'output' as output); if model metadata changes, the code must be updated.
- Model conversion requires python with tensorflow version 2.3.0 to convert keras models into HDF5 TF2 models.

## Evidence
