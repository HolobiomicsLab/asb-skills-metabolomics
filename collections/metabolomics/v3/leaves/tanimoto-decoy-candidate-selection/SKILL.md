---
name: tanimoto-decoy-candidate-selection
description: 'Use when benchmarking spectrum-prediction models on identification-by-retrieval:
  you need isomer-decoy candidate sets (true structure + Tanimoto-ranked same-formula
  decoys) against which to rank candidates by spectral similarity.'
when_to_use_negative:
- When evaluating de novo structure generation or spectrum-prediction accuracy itself
  — this skill benchmarks identification-by-retrieval against a candidate pool, not
  peak-level spectral prediction
- When molecular formulae for candidates or query spectra are unavailable — the decoy
  construction requires exact same-formula isomer filtering
- When only the main branch of ms-pred is available and NPLIB1 is the target dataset
  — NPLIB1 support has been retired from the main branch and requires the iceberg_1.0
  branch
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ICEBERG
  role: Primary spectrum predictor whose candidates are ranked for retrieval benchmarking
  repo: https://github.com/samgoldman97/ms-pred
- name: SCARF
  role: Baseline formula-level spectrum predictor included in the comparison ranking
  repo: https://github.com/samgoldman97/ms-pred
- name: MetFrag
  role: Combinatorial-fragmentation baseline used to rank the same candidate lists
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: PubChem
  role: Source of the candidate/decoy structures, subsetted by molecular formula via
    the pubchem data_scripts pipeline
- name: CFM-ID
  role: Baseline spectrum predictor (not retrained; run with provided instructions)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:d2bfcbce1c4ba9eb1cd2522f9b4641a3fbfd2fe59ee157a1e023840a51c509bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MetFrag
  - PubChem
  - Python
  - SCARF
  - uv
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Tanimoto-decoy candidate selection for retrospective MS/MS retrieval benchmarks

## Summary

Assembles fair retrieval candidate pools for retrospective metabolite identification by combining each test spectrum's true structure with up to 49 top-Tanimoto structural-isomer decoys sharing the same molecular formula (drawn from PubChem), then ranks candidates by predicted-vs-experimental spectral similarity to compute top-k hit rates.

## When to use

Use when benchmarking spectrum-prediction models on identification-by-retrieval: you need isomer-decoy candidate sets (true structure + Tanimoto-ranked same-formula decoys) against which to rank candidates by spectral similarity.

## When NOT to use

- When evaluating de novo structure generation or spectrum-prediction accuracy itself — this skill benchmarks identification-by-retrieval against a candidate pool, not peak-level spectral prediction
- When molecular formulae for candidates or query spectra are unavailable — the decoy construction requires exact same-formula isomer filtering
- When only the main branch of ms-pred is available and NPLIB1 is the target dataset — NPLIB1 support has been retired from the main branch and requires the iceberg_1.0 branch

## Inputs

- Processed test spectra from NPLIB1 (labels.tsv and spec_files.hdf5) and NIST20
- CASMI22 positive-mode spectra
- PubChem SMILES library (downloaded via data_scripts/pubchem/01_download_smiles.sh; pubchem_formulae_inchikey.hdf5 from Zenodo record 15529765)
- Per-molecule molecular formulae (for same-formula isomer filtering)
- Predicted spectra from ICEBERG and baselines (SCARF, NEIMS FFN/GNN, MetFrag, CFM-ID)

## Outputs

- 50-candidate retrieval lists per test spectrum (true structure + up to 49 top-Tanimoto same-formula isomer decoys)
- retrieval_accuracy_table.csv with per-model, per-dataset top-1/top-10 accuracies
- Bootstrap 95% confidence intervals on hit rates

## How to apply

Check out the iceberg_1.0 branch of ms-pred if NPLIB1 support is needed, then build the PubChem candidate library by running data_scripts/pubchem/01_download_smiles.sh, 02_make_formula_subsets.py, 03_dataset_subset.py against data/spec_datasets/nist20/labels.tsv, and 04_make_retrieval_lists.py. For each test spectrum in NPLIB1/NIST20/CASMI22, assemble a 50-candidate set consisting of the true structure plus up to 49 top-Tanimoto isomer decoys that share the exact molecular formula — isomerism (same formula, different structure) is the design principle that makes the retrieval task hard while keeping it fair across models. Predict spectra for every candidate with ICEBERG and the baselines (SCARF, NEIMS FFN/GNN, MetFrag, CFM-ID), rank candidates by spectral similarity via run_scripts/iceberg/06_run_retrieval.py, and apply NPLIB1-trained models to CASMI22 positive-mode spectra the same way. Compute top-1/top-10 hit rates with retrieval_benchmark_torchmetrics.py and bootstrap 95% confidence intervals with src/ms_pred/retrieval/bootstrap_metrics.py. Verify reproduced values against the reported margins: 29% vs 20% top-1 on NPLIB1, >5% top-10 margin on NIST20, and 12.9% vs 8.6% top-1 on CASMI22.

## Related tools

- **ICEBERG** (Primary spectrum predictor whose candidates are ranked for retrieval benchmarking) — https://github.com/samgoldman97/ms-pred
- **SCARF** (Baseline formula-level spectrum predictor included in the comparison ranking) — https://github.com/samgoldman97/ms-pred
- **MetFrag** (Combinatorial-fragmentation baseline used to rank the same candidate lists) — https://github.com/ipb-halle/MetFragRelaunched
- **PubChem** (Source of the candidate/decoy structures, subsetted by molecular formula via the pubchem data_scripts pipeline)
- **CFM-ID** (Baseline spectrum predictor (not retrained; run with provided instructions))

## Examples

```
python run_scripts/iceberg/06_run_retrieval.py && python src/ms_pred/retrieval/bootstrap_metrics.py
```

## Evaluation signals

- Each candidate set contains exactly 50 structures: the true structure plus up to 49 same-molecular-formula isomer decoys
- Reproduced top-1 accuracy on NPLIB1 should reach ~29% for ICEBERG vs ~20% for the next best model (9% absolute, nearly 46% relative improvement)
- NIST20 top-10 hit-rate margin over the next best model should exceed 5%
- CASMI22 top-1 accuracy should land at 12.9% (ICEBERG) vs 8.6% (next best)
- retrieval_accuracy_table.csv exists with per-model, per-dataset rows and bootstrap 95% CIs accompany the hit rates

## Limitations

- ICEBERG costs roughly 1 CPU second per molecule — nearly an order of magnitude more than neural baselines like SCARF — so scoring all 50 candidates per spectrum is compute-heavy
- NIST20/23 is a commercial dataset requiring a license; the ICEBERG web retrieval excludes NIST'23 structures to avoid license conflict
- There are no guarantees that ICEBERG's generated fragment explanations are the correct physical explanations, even when retrieval ranking is correct
- NPLIB1 (also referred to as CANOPUS) support is retired in the ms-pred main branch and requires the legacy iceberg_1.0 branch

## Evidence
