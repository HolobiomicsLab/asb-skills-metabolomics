---
name: top-k-accuracy-evaluation
description: Use when a candidate-generation model outputs ranked SMILES predictions
  per spectrum and you need Acc_k and Sim_k metrics for k=1,10,50 on held-out EI-MS
  test splits.
when_to_use_negative:
- 'Spectra are MS/MS rather than GC-EI-MS: existing de novo models like MassGenie
  rely on precursor ion mass or molecular formula cues SpecTUS does not use, so cross-comparison
  with this protocol is not straightforward.'
- Input spectra come from poorly curated libraries (e.g. Cayman, MONA) whose quality
  differs from rigorously curated NIST/SWGDRUG splits — headline numbers will not
  transfer.
- You need intermediate explanations such as peak annotations, fragmentation trees,
  or reference spectra for candidates; the top-k metrics do not provide model justification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SpecTUS
  role: Autoregressive model that translates EI-MS spectra into ranked SMILES candidates
    whose top-k predictions are scored
  repo: https://github.com/hejjack/SpecTUS
- name: NEIMS
  role: Synthetic spectra generator whose generated test half is evaluated with Acc1/Acc10/Sim1/Sim10
- name: RASSP
  role: Synthetic spectra generator whose generated test half is evaluated separately
    with the same metrics
- name: Jupyter Notebook
  role: Repository evaluation notebooks recomputing Acc_k and Sim_k and SpecTUS-vs-HSS
    win rates from deposited predictions
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup via trainSpectus_build.yaml to run evaluation notebooks
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_content_hash: sha256:aaeac834513ca1dd61ca6db104007cbec884f979cc8441a8553d5648d87c06f4
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - NEIMS
  - RASSP
  - SpecTUS
  - conda
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# top-k-accuracy-evaluation

## Summary

Evaluate structure-annotation models by computing top-k accuracy (Acc_k) and average top-k Tanimoto similarity (Sim_k) of Morgan fingerprints between ground-truth SMILES and ranked candidates, following the SpecTUS evaluation protocol.

## When to use

Use when a candidate-generation model outputs ranked SMILES predictions per spectrum and you need Acc_k and Sim_k metrics for k=1,10,50 on held-out EI-MS test splits.

## When NOT to use

- Spectra are MS/MS rather than GC-EI-MS: existing de novo models like MassGenie rely on precursor ion mass or molecular formula cues SpecTUS does not use, so cross-comparison with this protocol is not straightforward.
- Input spectra come from poorly curated libraries (e.g. Cayman, MONA) whose quality differs from rigorously curated NIST/SWGDRUG splits — headline numbers will not transfer.
- You need intermediate explanations such as peak annotations, fragmentation trees, or reference spectra for candidates; the top-k metrics do not provide model justification.

## Inputs

- Held-out test spectra (NIST 20 test split of 28,267 EI-MS spectra, or 5% synthetic test partition of NEIMS-/RASSP-generated synth1 and synth2)
- Ground-truth SMILES strings for each test spectrum
- Ranked SMILES candidate lists per spectrum (from model inference or deposited predictions/ directory for HSS, SSS, BDC)
- Morgan fingerprint representation of ground-truth and candidate structures

## Outputs

- Acc1 and Acc10 (and Acc50) exact-reconstruction rates per test set or generator
- Sim1 and Sim10 (and Sim50) average highest-Tanimoto-similarity scores
- Win-rate and at-least-as-good rate of SpecTUS vs. HSS per k
- Aggregated CSV, e.g. synthetic_test_metrics.csv or spectus_vs_hss_sss_metrics.csv

## How to apply

Run autoregressive inference to produce ranked SMILES candidate lists (e.g., k=1 and k=10 for held-out synthetic NEIMS/RASSP spectra; k=1, 10, and 50 when comparing against HSS/SSS/BDC database search on the NIST 20 test split of 28,267 spectra). Compute Acc_k as the proportion of test spectra whose ground-truth structure appears exactly among the top-k predictions. Compute Sim_k as, per query, the highest Tanimoto similarity between the Morgan fingerprint of the ground-truth molecule and each of the first k candidates, averaged over all queries, treating exact structural match as the best outcome. When benchmarking against database search, load deposited predictions from the repository's predictions/ directory, evaluate each method identically, and compute win and at-least-as-good rates versus hybrid search per k. Write metrics to CSV with rows per generator/method and columns Acc_k, Sim_k, validating against reported headline values.

## Related tools

- **SpecTUS** (Autoregressive model that translates EI-MS spectra into ranked SMILES candidates whose top-k predictions are scored) — https://github.com/hejjack/SpecTUS
- **NEIMS** (Synthetic spectra generator whose generated test half is evaluated with Acc1/Acc10/Sim1/Sim10)
- **RASSP** (Synthetic spectra generator whose generated test half is evaluated separately with the same metrics)
- **Jupyter Notebook** (Repository evaluation notebooks recomputing Acc_k and Sim_k and SpecTUS-vs-HSS win rates from deposited predictions) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup via trainSpectus_build.yaml to run evaluation notebooks) — https://github.com/hejjack/SpecTUS

## Examples

```
conda env create -f trainSpectus_build.yaml && git clone https://github.com/hejjack/SpecTUS.git
```

## Evaluation signals

- Acc1 reproduces at approximately 0.43 and Acc10 at approximately 0.65 on the NIST 20 test split
- SpecTUS-vs-HSS win rate at k=1 approximately 0.76 and at k=10 approximately 0.84
- Reproduced Acc1 on synthetic spectra matches reported exact-reconstruction rates: 38% RASSP-generated, 29% NEIMS-generated, 3% NIST at pretraining stage
- Metrics follow paper definitions: ground truth among top-k predictions; highest Morgan-fingerprint Tanimoto similarity averaged over queries
- HSS single-candidate success lands around 20%

## Limitations

- SpecTUS predictions cannot be explicitly justified and lack intermediate representations (peak annotations, fragmentation trees) or reference spectra for candidates
- Performance is sensitive to input spectra quality, shown by poorer results on Cayman and MONA test sets vs. NIST or SWGDRUG
- NIST20 is proprietary, requiring a license to retrain; pretrained-checkpoint performance on real spectra remains low (3% on NIST at end of pretraining)
- Comparison with other de novo approaches is difficult as all existing models were trained on MS/MS spectra, not GC-EI-MS

## Evidence

- [methods] top-k accuracy (Acc_k) and average top-k similarity (highest Tanimoto similarity of Morgan fingerprints between ground truth and the first k candidates) for k = 1, 10, and 50: [evidence span withheld — anchor status: not_found]
- [results] At the end of the pretraining stage, the model's exact-structure reconstruction remained relatively low: 38% for RASSP-generated spectra, 29% for NEIMS-generated spectra: [evidence span withheld — anchor status: not_found]
