---
name: topk-hit-rate-benchmark-evaluation
description: 'Use when you need to benchmark spectrum prediction models by candidate
  retrieval: a test MS/MS spectrum, a fixed candidate set of same-formula isomer structures,
  and you want top-k identification accuracy.'
when_to_use_negative:
- The main branch of ms-pred has retired NPLIB1 support — use the iceberg_1.0 branch
  if the benchmark includes NPLIB1/CANOPUS data
- You are evaluating spectrum prediction accuracy (intensity/cosine error on held-out
  spectra) rather than candidate retrieval/identification
- Your candidate sets are not formula-matched isomer libraries (e.g., free-form database
  search without molecular formula constraints)
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: ICEBERG
  role: Primary model whose predicted fragmentation-graph spectra are ranked against
    candidates for top-k accuracy
  repo: https://github.com/coleygroup/ms-pred
- name: SCARF
  role: Subformula-classification baseline included in the same retrieval ranking
    comparison
  repo: https://github.com/coleygroup/ms-pred
- name: MetFrag
  role: Combinatorial fragmentation baseline run on candidates and ranked by spectral
    similarity
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CFM-ID
  role: Spectral prediction baseline (not retrained; run per provided instructions)
    in the top-k comparison
- name: NEIMS (FFN/GNN)
  role: Neistra-style spectrum prediction baselines trained with equivalent settings
    for fair top-k comparison
  repo: https://github.com/coleygroup/ms-pred
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:ddd52c2ce0a8347b02a835cd5350bda3bbfa56035afea3c83dc0a7e997737afe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MetFrag
  - NEIMS (FFN/GNN)
  - Python
  - SCARF
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# topk-hit-rate-benchmark-evaluation

## Summary

Evaluates retrospective structure identification by ranking candidate molecules with predicted-vs-experimental spectral similarity and reporting top-1/top-10 hit rates (with bootstrap 95% CIs) for ICEBERG versus baseline models on NPLIB1, NIST20, and CASMI22.

## When to use

Use when you need to benchmark spectrum prediction models by candidate retrieval: a test MS/MS spectrum, a fixed candidate set of same-formula isomer structures, and you want top-k identification accuracy.

## When NOT to use

- The main branch of ms-pred has retired NPLIB1 support — use the iceberg_1.0 branch if the benchmark includes NPLIB1/CANOPUS data
- You are evaluating spectrum prediction accuracy (intensity/cosine error on held-out spectra) rather than candidate retrieval/identification
- Your candidate sets are not formula-matched isomer libraries (e.g., free-form database search without molecular formula constraints)

## Inputs

- Processed test spectra with labels.tsv and spec_files.hdf5 (NPLIB1, NIST20 test splits; CASMI22 positive-mode spectra)
- PubChem candidate library built via data_scripts/pubchem/01_download_smiles.sh through 04_make_retrieval_lists.py
- 50-candidate retrieval lists: true structure plus up to 49 top-Tanimoto same-formula isomer decoys per spectrum
- Predicted spectra for all candidates from ICEBERG and baselines (SCARF, NEIMS FFN/GNN, MetFrag, CFM-ID)

## Outputs

- retrieval_accuracy_table.csv with per-model, per-dataset top-1/top-10 accuracies
- Bootstrap 95% confidence intervals for top-k hit rates per model/dataset
- Reproduced margin checks (29% vs 20% top-1 NPLIB1; 12.9% vs 8.6% top-1 CASMI22)

## How to apply

For each test spectrum, assemble a 50-candidate set consisting of the true structure plus up to 49 top-Tanimoto isomer decoys sharing the same molecular formula (PubChem decoys built via the 01–04 pubchem data_scripts against data/spec_datasets/nist20/labels.tsv). Predict spectra for all candidates with ICEBERG and each baseline (SCARF, NEIMS FFN/GNN, MetFrag, CFM-ID), rank candidates by spectral similarity using run_scripts/iceberg/06_run_retrieval.py, and apply the NPLIB1-trained models to CASMI22 positive-mode spectra the same way. Compute top-1/top-10 hit rates with retrieval_benchmark_torchmetrics.py, then estimate bootstrap 95% confidence intervals with src/ms_pred/retrieval/bootstrap_metrics.py, writing retrieval_accuracy_table.csv with per-model, per-dataset top-k accuracies. Verify reproduced values against the reported margins: 29% vs 20% top-1 on NPLIB1 (a 9% absolute / nearly 46% relative gain for ICEBERG), >5% top-10 on NIST20, and 12.9% vs 8.6% top-1 on CASMI22.

## Related tools

- **ICEBERG** (Primary model whose predicted fragmentation-graph spectra are ranked against candidates for top-k accuracy) — https://github.com/coleygroup/ms-pred
- **SCARF** (Subformula-classification baseline included in the same retrieval ranking comparison) — https://github.com/coleygroup/ms-pred
- **MetFrag** (Combinatorial fragmentation baseline run on candidates and ranked by spectral similarity) — https://github.com/ipb-halle/MetFragRelaunched
- **CFM-ID** (Spectral prediction baseline (not retrained; run per provided instructions) in the top-k comparison)
- **NEIMS (FFN/GNN)** (Neistra-style spectrum prediction baselines trained with equivalent settings for fair top-k comparison) — https://github.com/coleygroup/ms-pred

## Examples

```
python src/ms_pred/retrieval/bootstrap_metrics.py  # after retrieval_benchmark_torchmetrics.py and 06_run_retrieval.py
```

## Evaluation signals

- Reproduced top-1 accuracy on NPLIB1 matches the reported 29% vs 20% margin (9% absolute, ~46% relative over the next best model)
- Reproduced top-1 accuracy on CASMI22 matches 12.9% vs 8.6%
- Top-10 accuracy on NIST20 exceeds the next best model by >5%
- Every candidate set contains exactly the true structure plus ≤49 same-molecular-formula isomer decoys (50 candidates total)
- Bootstrap 95% CIs computed via src/ms_pred/retrieval/bootstrap_metrics.py accompany each reported top-k accuracy

## Limitations

- ICEBERG inference costs roughly 1 CPU second per molecule, nearly an order of magnitude more than SCARF, which limits throughput when scoring large candidate libraries
- NPLIB1 support is retired on the ms-pred main branch; the iceberg_1.0 branch must be used
- There are no guarantees that ICEBERG's generated fragment explanations are the correct physical explanations, due to equivalent-mass atom/bond rearrangements and the cosine-similarity training objective

## Evidence

- [results] In retrospective retrieval on NPLIB1, ICEBERG improves top-1 accuracy over the next best model by 9% absolute (nearly 46% relative): [evidence span withheld — anchor status: not_found]
