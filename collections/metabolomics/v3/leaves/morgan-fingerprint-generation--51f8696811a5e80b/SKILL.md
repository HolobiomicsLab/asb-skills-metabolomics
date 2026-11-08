---
name: morgan-fingerprint-generation
description: Use when ranked structural candidate lists (SMILES) must be scored against
  ground-truth SMILES for GC-EI-MS spectra, and you need top-k accuracy or SpecTUS-vs-database-search
  win rates via Morgan-fingerprint Tanimoto similarity.
when_to_use_negative:
- Input spectra are MS/MS rather than GC-EI-MS — the paper notes existing de novo
  methods were designed and trained for MS/MS spectra, so cross-technique comparison
  via this metric setup is not directly valid.
- No ground-truth SMILES are available for the test spectra, since Tanimoto comparison
  requires reference structures.
- Input spectra are from poorly curated sources (e.g. Cayman or MONA) where the model's
  sensitivity to spectrum quality degrades scores relative to curated NIST/SWGDRUG
  data.
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: Source of the de novo SMILES candidate predictions whose Morgan-fingerprint
    Tanimoto similarity is scored against HSS search results
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Repository evaluation notebooks used to compute Acc_k, top-k Tanimoto similarity,
    and SpecTUS-vs-HSS win rates
- name: conda
  role: Environment setup via trainSpectus_build.yaml before running the evaluation
    notebooks
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:9f1e7aeb349100f9eb36b99495c266b0db91841c09079e91e0c3f92a1684f7b6
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# morgan-fingerprint-generation

## Summary

Generates Morgan fingerprints from ground-truth and predicted SMILES and uses their Tanimoto similarity to score structural candidates from EI-MS spectra — the exact metric SpecTUS uses to prove de novo annotation beats hybrid database search (HSS).

## When to use

Use when ranked structural candidate lists (SMILES) must be scored against ground-truth SMILES for GC-EI-MS spectra, and you need top-k accuracy or SpecTUS-vs-database-search win rates via Morgan-fingerprint Tanimoto similarity.

## When NOT to use

- Input spectra are MS/MS rather than GC-EI-MS — the paper notes existing de novo methods were designed and trained for MS/MS spectra, so cross-technique comparison via this metric setup is not directly valid.
- No ground-truth SMILES are available for the test spectra, since Tanimoto comparison requires reference structures.
- Input spectra are from poorly curated sources (e.g. Cayman or MONA) where the model's sensitivity to spectrum quality degrades scores relative to curated NIST/SWGDRUG data.

## Inputs

- Ranked SMILES candidate lists (per-spectrum predictions) for SpecTUS and database search methods HSS, SSS, BDC from predictions/
- NIST 20 held-out test split of 28,267 spectra with ground-truth SMILES from data/

## Outputs

- Top-k accuracy (Acc_1, Acc_10, Acc_50) per method
- Average top-k Tanimoto similarity of Morgan fingerprints (ground truth vs first k candidates)
- Win rate and at-least-as-good rate of SpecTUS vs HSS at each k
- Aggregated metrics table, e.g. spectus_vs_hss_sss_metrics.csv

## How to apply

Clone github.com/hejjack/SpecTUS and build the environment with `conda env create -f trainSpectus_build.yaml`. Load deposited predictions for all models and database search methods (HSS, SSS, BDC) from the predictions/ directory together with the NIST 20 held-out test split (28,267 spectra) and ground-truth SMILES from data/. Using the repository's evaluation notebooks, compute top-k accuracy (Acc_k) and average top-k similarity — the highest Tanimoto similarity of Morgan fingerprints between ground truth and the first k candidates — for k = 1, 10, and 50. Then compute the win rate and at-least-as-good rate of SpecTUS versus HSS for each k, treating an exact structural match as the best outcome and otherwise comparing Morgan-fingerprint Tanimoto similarity. Write the recomputed metrics to a CSV (e.g. spectus_vs_hss_sss_metrics.csv) for comparison against the reported headline numbers.

## Related tools

- **SpecTUS** (Source of the de novo SMILES candidate predictions whose Morgan-fingerprint Tanimoto similarity is scored against HSS search results) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Repository evaluation notebooks used to compute Acc_k, top-k Tanimoto similarity, and SpecTUS-vs-HSS win rates)
- **conda** (Environment setup via trainSpectus_build.yaml before running the evaluation notebooks)

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Acc1 on the NIST 20 test split reproduces at approximately 0.43.
- Acc10 reproduces at approximately 0.65.
- SpecTUS-vs-HSS win rate at k=1 is approximately 0.76 and at k=10 approximately 0.84, matching the abstract and supplementary tables.
- Exact structural matches are handled as the best outcome before falling back to Morgan-fingerprint Tanimoto comparison.
- All recomputed metrics are written to spectus_vs_hss_sss_metrics.csv with per-k values consistent across k = 1, 10, 50.

## Limitations

- The final model and its NIST 20 fine-tuned checkpoint are not redistributable because NIST 20 is proprietary; you need a NIST 20 license to retrain before reproducing predictions yourself.
- Scores are sensitive to input spectrum quality — performance is poorer on Cayman and MONA test sets than on curated NIST or SWGDRUG libraries.
- The Tanimoto-of-Morgan-fingerprint score is a proxy, not identity: it ranks 'strictly better' candidates without offering peak annotations, fragmentation trees, or reference spectra for manual correction.

## Evidence
