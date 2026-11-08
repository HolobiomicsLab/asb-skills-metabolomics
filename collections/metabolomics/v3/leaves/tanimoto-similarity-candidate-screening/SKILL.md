---
name: tanimoto-similarity-candidate-screening
description: 'Use when evaluating retrieval-based MS2C compound identification: you
  have a real MS/MS spectrum Y_i for a molecule X_i, C2MS models to score, and need
  a controlled candidate pool (e.g. 50 compounds including X_i plus PubChem neighbours)
  to compute Top-1/3/5 accuracy.'
when_to_use_negative:
- Input spectra are already paired with a fixed spectral library and no new candidate
  sampling is needed (the task is library-only MS2C, not augmented retrieval)
- You need formula or substructure annotation for the unknown spectrum itself rather
  than a ranked candidate list from chemical similarity
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: RDKit
  role: Computes chemical fingerprints and Tanimoto similarity used to select the
    49 PubChem molecules most chemically similar to X_i
- name: PyTorch
  role: Backend for running the FraGNNet and baseline C2MS models that generate predicted
    spectra for every candidate in each candidate set
  repo: https://github.com/FraGNNet/fragnnet
- name: PyTorch Lightning
  role: Training/evaluation harness for the C2MS models used inside the retrieval
    benchmark
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: FraGNNet
  role: Primary C2MS model whose predicted spectra are compared against the real spectrum
    Y_i to rank candidates
  repo: https://github.com/FraGNNet/fragnnet
- name: Python
  role: Glue language for fingerprint generation, similarity ranking, and Top-k aggregation
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:b1a28a86ccc9feac6059d2bfaf5494ba196ca89757e3345b731600bb4581b073
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/FraGNNet/fragnnet
  tools:
  - FraGNNet
  - PyTorch
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tanimoto-similarity-candidate-screening

## Summary

Builds retrieval candidate sets for the MS2C problem by pairing each true molecule X_i with 49 PubChem molecules selected by highest Tanimoto similarity between RDKit chemical fingerprints, then ranks candidates by predicted-spectrum similarity to evaluate Top-k compound identification.

## When to use

Use when evaluating retrieval-based MS2C compound identification: you have a real MS/MS spectrum Y_i for a molecule X_i, C2MS models to score, and need a controlled candidate pool (e.g. 50 compounds including X_i plus PubChem neighbours) to compute Top-1/3/5 accuracy.

## When NOT to use

- Input spectra are already paired with a fixed spectral library and no new candidate sampling is needed (the task is library-only MS2C, not augmented retrieval)
- You need formula or substructure annotation for the unknown spectrum itself rather than a ranked candidate list from chemical similarity

## Inputs

- Held-out NIST20 MS/MS test spectrum-molecule pairs (~4,000 spectra)
- PubChem compound database entries with structures for fingerprinting
- RDKit chemical fingerprints for each molecule
- Trained C2MS models (FraGNNet-D4, ICEBERG, MassFormer, NEIMS) or their predicted spectra
- Real test spectrum Y_i for each X_i

## Outputs

- Per-model Top-1 / Top-3 / Top-5 accuracy (%) on the InChIKey split and Scaffold split
- Mean ± standard deviation over 5 random seeds per cell of the Top-k table
- Rank of the true compound within each 50-candidate set

## How to apply

For each test molecule X_i from the ~4,000 held-out NIST20 MS/MS spectrum-molecule pairs, compute Tanimoto similarity between RDKit chemical fingerprints of X_i and PubChem candidates. Select the 49 PubChem molecules with highest Tanimoto similarity to X_i and add X_i itself to form a 50-compound candidate set C_i; the Tanimoto screen ensures hard negatives that make Top-k accuracy a meaningful metric. Predict an MS/MS spectrum for every candidate with each C2MS model (e.g. FraGNNet-D4, ICEBERG, MassFormer, NEIMS), score each predicted spectrum against the real spectrum Y_i with spectral-similarity metrics such as binned cosine or Hungarian cosine at 10 ppm tolerance (with or without square-root intensity transformation), rank the 50 candidates by similarity, record the rank of the true compound, and compute Top-1/Top-3/Top-5 percentages on both the InChIKey and Scaffold splits, reporting mean ± standard deviation over 5 random seeds. Run the whole procedure under at least 5 seeds because candidate sampling introduces variance.

## Related tools

- **RDKit** (Computes chemical fingerprints and Tanimoto similarity used to select the 49 PubChem molecules most chemically similar to X_i)
- **PyTorch** (Backend for running the FraGNNet and baseline C2MS models that generate predicted spectra for every candidate in each candidate set) — https://github.com/FraGNNet/fragnnet
- **PyTorch Lightning** (Training/evaluation harness for the C2MS models used inside the retrieval benchmark) — https://github.com/Lightning-AI/pytorch-lightning
- **FraGNNet** (Primary C2MS model whose predicted spectra are compared against the real spectrum Y_i to rank candidates) — https://github.com/FraGNNet/fragnnet
- **Python** (Glue language for fingerprint generation, similarity ranking, and Top-k aggregation)

## Examples

```
from rdkit import DataStructs; sims = DataStructs.BulkTanimotoSimilarity(fp_i, pubchem_fps); top49 = sorted(range(len(sims)), key=lambda j: sims[j], reverse=True)[1:50]
```

## Evaluation signals

- FraGNNet-D4 Top-1 accuracy reproduces Table 2 values: 36.0 ± 0.4 on the InChIKey split and 30.3 ± 0.6 on the Scaffold split
- Candidate set size is exactly 50 compounds per test molecule (X_i + 49 PubChem neighbours)
- Top-k accuracies are reported as mean ± standard deviation across 5 independent random seeds
- Both InChIKey and Scaffold split accuracies are reported for every model (FraGNNet-D4, ICEBERG, MassFormer, NEIMS)

## Limitations

- Candidate sampling introduces variance; accuracies must be averaged over multiple random seeds (5 seeds in the paper) to be stable
- Performance was evaluated on the NIST20 test set with a fixed candidate size of 50; other candidate pool sizes or library sources are not benchmarked here
- Ranking quality depends on the chosen spectral similarity metric (e.g. binned cosine vs. Hungarian cosine at 10 ppm with or without square-root intensity transform); the paper selects the best method per model

## Evidence

- [methods] a candidate set C_i is constructed from X_i and 49 other molecules sampled from PubChem (Kim et al., 2019): [evidence span withheld — anchor status: not_found]
- [methods] Each C_i in C_i - {X_i} is selected to have high chemical similarity with X_i as measured by Tanimoto similarity between chemical fingerprints: [evidence span withheld — anchor status: not_found]
- [methods] We evaluated C2MS performance on a held-out portion of the NIST20 MS/MS dataset: [evidence span withheld — anchor status: not_found]
- [methods] The FraGNNet model and baselines were implemented in Python, using Pytorch: [evidence span withheld — anchor status: not_found]
