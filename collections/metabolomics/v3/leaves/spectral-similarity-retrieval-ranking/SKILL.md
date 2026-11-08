---
name: spectral-similarity-retrieval-ranking
description: Use when evaluating how well a spectrum-prediction model identifies an
  unknown molecule from a decoy-augmented candidate library (e.g., PubChem isomer
  sets for NPLIB1, NIST20, or CASMI22 spectra), where ranking accuracy is the metric
  of interest.
when_to_use_negative:
- The main branch no longer supports NPLIB1 (also referred as CANOPUS); use the iceberg_1.0
  branch instead if NPLIB1 retrieval is the goal
- The goal is spectrum-prediction accuracy (cosine similarity against ground-truth
  peaks) rather than candidate-list retrieval ranking
- Candidate sets are not formula-constrained isomer libraries — retrieval margins
  like 29% vs 20% top-1 assume the true structure plus same-formula Tanimoto-nearest
  decoys
edam_operation: http://edamontology.org/operation_2409
tools:
- name: ICEBERG
  role: Primary spectrum predictor whose fragmentation simulations drive candidate
    ranking in retrieval
  repo: https://github.com/samgoldman97/ms-pred
- name: SCARF
  role: Neural baseline spectrum predictor compared in the retrieval benchmark
  repo: https://github.com/samgoldman97/ms-pred
- name: MetFrag
  role: Combinatorial fragmentation baseline used for candidate scoring/ranking
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CFM-ID
  role: Rule-based baseline spectrum predictor used for candidate ranking (not retrained)
  repo: https://github.com/samgoldman97/ms-pred
- name: uv
  role: Environment manager used to set up the ms-pred runtime before running retrieval
    scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:71ac692b48c3253077a6757d0af4f5beaa6673ee62dc516ecf4adb49ca49b8ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MetFrag
  - Python
  - SCARF
  - uv
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# spectral-similarity-retrieval-ranking

## Summary

Benchmark retrospective metabolite identification by predicting MS/MS spectra for candidate structures with ICEBERG (and baselines SCARF, NEIMS FFN/GNN, MetFrag, CFM-ID) and ranking candidates against an experimental spectrum by spectral similarity, reporting top-1/top-10 hit rates.

## When to use

Use when evaluating how well a spectrum-prediction model identifies an unknown molecule from a decoy-augmented candidate library (e.g., PubChem isomer sets for NPLIB1, NIST20, or CASMI22 spectra), where ranking accuracy is the metric of interest.

## When NOT to use

- The main branch no longer supports NPLIB1 (also referred as CANOPUS); use the iceberg_1.0 branch instead if NPLIB1 retrieval is the goal
- The goal is spectrum-prediction accuracy (cosine similarity against ground-truth peaks) rather than candidate-list retrieval ranking
- Candidate sets are not formula-constrained isomer libraries — retrieval margins like 29% vs 20% top-1 assume the true structure plus same-formula Tanimoto-nearest decoys

## Inputs

- Processed test spectra with labels.tsv and spec_files.hdf5 (NPLIB1 and NIST20 test splits)
- PubChem candidate library (pubchem_formulae_inchikey.hdf5 from zenodo.org/records/15529765) with per-formula isomer decoys and retrieval lists
- CASMI22 positive-mode spectra with candidate lists
- Predicted or experimental spectra for all 50 candidates per query

## Outputs

- Per-query candidate rankings by spectral similarity
- retrieval_accuracy_table.csv with per-model, per-dataset top-1/top-10 accuracies
- Bootstrap 95% confidence intervals on top-k hit rates

## How to apply

First assemble, for each test spectrum, a 50-candidate retrieval set consisting of the true structure plus up to 49 top-Tanimoto isomer decoys sharing the same molecular formula (built via the data_scripts/pubchem/01-04 pipeline against data/spec_datasets/nist20/labels.tsv). Second, predict spectra for every candidate with ICEBERG (the two-part Generate/Score fragmentation-graph model) and the comparison models, then rank candidates by spectral similarity using run_scripts/iceberg/06_run_retrieval.py. Third, apply NPLIB1-trained models to CASMI22 positive-mode spectra to test cross-dataset generalization. Finally, compute top-1/top-10 hit rates with retrieval_benchmark_torchmetrics.py and estimate bootstrap 95% confidence intervals with src/ms_pred/retrieval/bootstrap_metrics.py; the reproduced numbers should match the reported margins (29% vs 20% top-1 on NPLIB1; >5% top-10 on NIST20; 12.9% vs 8.6% top-1 on CASMI22).

## Related tools

- **ICEBERG** (Primary spectrum predictor whose fragmentation simulations drive candidate ranking in retrieval) — https://github.com/samgoldman97/ms-pred
- **SCARF** (Neural baseline spectrum predictor compared in the retrieval benchmark) — https://github.com/samgoldman97/ms-pred
- **MetFrag** (Combinatorial fragmentation baseline used for candidate scoring/ranking) — https://github.com/ipb-halle/MetFragRelaunched
- **CFM-ID** (Rule-based baseline spectrum predictor used for candidate ranking (not retrained)) — https://github.com/samgoldman97/ms-pred
- **uv** (Environment manager used to set up the ms-pred runtime before running retrieval scripts)

## Examples

```
python src/ms_pred/retrieval/bootstrap_metrics.py  # after ranking with run_scripts/iceberg/06_run_retrieval.py and retrieval_benchmark_torchmetrics.py
```

## Evaluation signals

- Top-1 accuracy on NPLIB1 should approach the reported 29% for ICEBERG vs 20% for the next best model (9% absolute / ~46% relative improvement)
- Top-10 accuracy on NIST20 should exceed the next best model by more than 5 percentage points
- CASMI22 top-1 should reproduce 12.9% vs 8.6% when NPLIB1-trained models are applied to positive-mode spectra
- Each retrieval set must contain exactly 50 formula-matched candidates (true structure + up to 49 top-Tanimoto isomer decoys)
- Bootstrap 95% CIs from src/ms_pred/retrieval/bootstrap_metrics.py should overlap the reported margins, confirming statistical robustness

## Limitations

- NPLIB1 support is retired on the ms-pred main branch and lives only on the iceberg_1.0 branch
- NIST20/NIST23 are commercial datasets requiring a purchased license; pretrained weights on NIST'20 require proof of license, or open-source MassSpecGym-trained weights can be substituted
- ICEBERG inference costs roughly 1 CPU second per molecule, nearly an order of magnitude more than SCARF, limiting retrieval throughput over large libraries
- The PubChem retrieval path excludes NIST'23 structures to avoid license conflict, so library coverage is partial (~91 million structures)

## Evidence

- [intro] We evaluate ICEBERG on two datasets: NPLIB1 (GNPS data as used to train the CANOPUS model) and NIST20,21: [evidence span withheld — anchor status: not_found]
