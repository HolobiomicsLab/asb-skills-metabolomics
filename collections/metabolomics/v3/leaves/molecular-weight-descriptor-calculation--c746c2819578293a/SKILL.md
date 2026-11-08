---
name: molecular-weight-descriptor-calculation
description: Reference for quantifying whether two molecular benchmark sets differ
  in structural complexity via the Ertl-Schuffenhauer synthetic accessibility score
  and molecular weight.
when_to_use_negative:
- When you need fragmentation-aware or spectral descriptors (e.g. predicted mass spectra,
  fragmentation graphs) rather than simple molecular complexity metrics — use ICEBERG
  or SCARF instead.
- When the dataset is not one of NPLIB1/NIST20 (or a similar SMILES-labeled benchmark)
  and the goal is only retrieval or elucidation accuracy, not complexity comparison.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Parse SMILES into molecule objects and compute Ertl-Schuffenhauer synthetic
    accessibility score and exact molecular weight per molecule
- name: Python
  role: Script the descriptor computation pipeline, average per dataset, and write
    the summary CSV
- name: ms-pred
  role: Source of dataset label files (NPLIB1 via legacy iceberg_1.0 branch; NIST20
    via data/spec_datasets/nist20/labels.tsv)
  repo: https://github.com/coleygroup/ms-pred
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:8c2c37b3c48af9fcd882cfc0e0d152855449ecc4582e7c38fafb5bd6b79c9b1b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - Python
  - RDKit
  - ms-pred
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# molecular-weight-descriptor-calculation

## Summary

Compute synthetic accessibility (SA) scores and molecular weights for molecules across two datasets (e.g. NPLIB1 vs NIST20) to quantify dataset-level molecular complexity and explain performance differences between spectrum prediction approaches.

## When to use

Reference for quantifying whether two molecular benchmark sets differ in structural complexity via the Ertl-Schuffenhauer synthetic accessibility score and molecular weight.

## When NOT to use

- When you need fragmentation-aware or spectral descriptors (e.g. predicted mass spectra, fragmentation graphs) rather than simple molecular complexity metrics — use ICEBERG or SCARF instead.
- When the dataset is not one of NPLIB1/NIST20 (or a similar SMILES-labeled benchmark) and the goal is only retrieval or elucidation accuracy, not complexity comparison.

## Inputs

- SMILES strings for NPLIB1 molecules (from legacy iceberg_1.0-branch dataset labels)
- NIST20 molecular structures from data/spec_datasets/nist20/labels.tsv

## Outputs

- Per-molecule SA score and molecular weight
- Dataset-level summary CSV with mean SA and mean MW for NPLIB1 vs NIST20 (4 rows, 2 datasets × 2 metrics)

## How to apply

Load molecule SMILES strings for each dataset: for NPLIB1 pull the labels file from the legacy iceberg_1.0-branch of ms-pred (main-branch support is retired), and for NIST20 read data/spec_datasets/nist20/labels.tsv. Parse each SMILES into a molecule object with RDKit, excluding structures that fail sanitization. Score every molecule with the Ertl-Schuffenhauer synthetic accessibility estimator and compute exact molecular weights of the neutral parent structure. Average SA and molecular weight within each dataset and assemble a summary table (2 datasets × 2 metrics) written to CSV. The rationale, per the article, is that NPLIB1 natural product molecules are more complex than NIST20 small organic standards, so complexity descriptors can explain dataset-dependent gaps in prediction accuracy.

## Related tools

- **RDKit** (Parse SMILES into molecule objects and compute Ertl-Schuffenhauer synthetic accessibility score and exact molecular weight per molecule)
- **Python** (Script the descriptor computation pipeline, average per dataset, and write the summary CSV)
- **ms-pred** (Source of dataset label files (NPLIB1 via legacy iceberg_1.0 branch; NIST20 via data/spec_datasets/nist20/labels.tsv)) — https://github.com/coleygroup/ms-pred

## Examples

```
from rdkit import Chem; from rdkit.Contrib.SA_Score import sascorer; mol=Chem.MolFromSmiles('CCO'); print(sascorer.calculateScore(mol), Chem.Descriptors.ExactMolWt(mol))
```

## Evaluation signals

- Verify that computed mean SA scores reproduce the reported Figure 3C-D values: SA 3.75 for NPLIB1 and 3.01 for NIST20.
- Verify that computed mean molecular weights reproduce the reported values: 413 Da for NPLIB1 and 317 Da for NIST20.
- Check that molecules failing RDKit sanitization are excluded from both datasets so per-dataset counts are consistent.
- Confirm the output CSV contains exactly 4 rows (2 datasets × 2 metrics) with matching dataset names.

## Limitations

- NPLIB1 (also referred to as CANOPUS) is no longer supported in the ms-pred main branch; the legacy iceberg_1.0 branch must be checked out to obtain its labels.
- The SA score reflects synthetic accessibility heuristics, not true synthetic feasibility, so it is a proxy for molecular complexity rather than an absolute measure.
- Molecular weight and SA score capture only two aspects of complexity; scaffold class or ring complexity may need additional descriptors.

## Evidence

- [other] NPLIB1 molecules are more complex than NIST20: NPLIB1 has an average SA score of 3.75 vs 3.01 for NIST20: [evidence span withheld — anchor status: not_found]
- [other] NPLIB1 has an average molecular weight of 413 Da vs 317 Da for NIST20: [evidence span withheld — anchor status: not_found]
- [intro] ICEBERG evaluated on both datasets to test prediction of complex natural products and small organic standards: [evidence span withheld — anchor status: not_found]
