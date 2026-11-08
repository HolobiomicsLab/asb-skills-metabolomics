---
name: database-search-baseline-reproduction
description: Use when benchmarking a de novo structure-annotation model against database-search
  baselines (HSS, SSS, BDC) on EI-MS spectra, or when re-deriving headline metrics
  like Acc1 = 0.43 or win rate = 0.76 from deposited predictions.
when_to_use_negative:
- Input spectra are tandem MS/MS rather than gas chromatography electron ionization
  (GC-EI) mass spectra — the comparison and the underlying model are specific to EI-MS
- The goal is to generate new predictions rather than re-evaluate the deposited ones
  — that requires an NIST 20 license and the full training pipeline, not just the
  evaluation notebooks
- The task needs explainability or peak-level justification of candidate rankings
  — the paper notes SpecTUS lacks intermediate representations such as peak annotations
  or fragmentation trees
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: SpecTUS
  role: Source of the de novo predictions and deposited prediction files being evaluated
    against the HSS/SSS/BDC baselines
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup via the trainSpectus_build.yaml specification so evaluation
    notebooks run reproducibly
- name: Jupyter Notebook
  role: Evaluation notebooks used to recompute Acc_k, top-k Tanimoto similarity, and
    win rates
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:91d9bd85183bdb3ced39c58d144799ca9cbcc27bfdad7a0cd728baa823b18085
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# database-search-baseline-reproduction

## Summary

Reproduces the SpecTUS-vs-database-search comparison (hybrid similarity search HSS, similarity search SSS, Blackwell's data compound BDC) on the 28,267-spectrum NIST 20 test split, computing top-k accuracy, Morgan-fingerprint Tanimoto similarity, and SpecTUS-vs-HSS win rates for k = 1, 10, and 50.

## When to use

Use when benchmarking a de novo structure-annotation model against database-search baselines (HSS, SSS, BDC) on EI-MS spectra, or when re-deriving headline metrics like Acc1 = 0.43 or win rate = 0.76 from deposited predictions.

## When NOT to use

- Input spectra are tandem MS/MS rather than gas chromatography electron ionization (GC-EI) mass spectra — the comparison and the underlying model are specific to EI-MS
- The goal is to generate new predictions rather than re-evaluate the deposited ones — that requires an NIST 20 license and the full training pipeline, not just the evaluation notebooks
- The task needs explainability or peak-level justification of candidate rankings — the paper notes SpecTUS lacks intermediate representations such as peak annotations or fragmentation trees

## Inputs

- Deposited test predictions of all models and database search methods (HSS, SSS, BDC) from the repository's predictions/ directory
- NIST 20 held-out test split of 28,267 spectra
- Ground-truth SMILES strings from the repository's data/ directory
- Repository evaluation notebooks (Jupyter)

## Outputs

- spectus_vs_hss_sss_metrics.csv with Acc_1, Acc_10, Acc_50 and average top-k Tanimoto similarity
- Win rate and at-least-as-good rate of SpecTUS versus HSS for each k
- Recomputed values cross-checked against the paper's headline numbers (Acc1 ~0.43, Acc10 ~0.65, win rates ~0.76 / ~0.84)

## How to apply

Clone github.com/hejjack/SpecTUS and build the environment with 'conda env create -f trainSpectus_build.yaml'. Load the deposited test predictions of all models and the database search methods (HSS, SSS, BDC) from the predictions/ directory together with the NIST 20 test split (28,267 spectra) and ground-truth SMILES from data/, rather than re-running inference. Using the repository's evaluation notebooks, compute Acc_k (fraction of exact structural matches in the top k) and average top-k similarity (highest Tanimoto similarity of Morgan fingerprints between ground truth and the first k candidates) for k = 1, 10, and 50. Compute win rate and at-least-as-good rate of SpecTUS versus HSS per k, treating exact structural match as best and otherwise comparing Morgan-fingerprint Tanimoto similarity; exact structural match must be checked before falling back to fingerprint comparison. Write all recomputed metrics to spectus_vs_hss_sss_metrics.csv and diff them against the reported values (Acc1 ~0.43, Acc10 ~0.65, win rate ~0.76 at k=1 and ~0.84 at k=10, HSS single-candidate success around 20%).

## Related tools

- **SpecTUS** (Source of the de novo predictions and deposited prediction files being evaluated against the HSS/SSS/BDC baselines) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup via the trainSpectus_build.yaml specification so evaluation notebooks run reproducibly)
- **Jupyter Notebook** (Evaluation notebooks used to recompute Acc_k, top-k Tanimoto similarity, and win rates) — https://github.com/ljocha/spectus-demo

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Acc1 recomputes to approximately 0.43 on the NIST 20 test split
- Acc10 recomputes to approximately 0.65 on the same split
- SpecTUS-vs-HSS win rate at k=1 is approximately 0.76 and at k=10 approximately 0.84
- HSS single-candidate success lands around 20%
- All recomputed metrics match the reported values reported in the abstract and supplementary tables

## Limitations

- NIST 20 is proprietary — full reproduction of the finetuning requires a license, although re-evaluation only needs the deposited predictions
- Compounds containing deuterium and compounds overlapping the training sets were filtered out of the test sets, so metrics do not cover those cases
- The NIST test split was built with an 0.8:0.1:0.1 split keeping duplicates within the same split, so results are not directly transferable to other splits
- Win-rate comparisons depend on Morgan fingerprint Tanimoto similarity, which treats near-miss structures as partially correct rather than exact matches

## Evidence

- [readme] On 76% of this test set, the single suggestion is strictly better, in terms of Tanimoto similarity of Morgan fingerprint, than the result of hybrid database search.: "On 76% of this test set, the single suggestion is strictly better, in terms of Tanimoto similarity of Morgan fingerprint, than the result of hybrid database search."
- [readme] predictions/ directory containing test predictions of all models and database search methods (HSS, SSS, BDC): "predictions/             # Test predictions of all models (hyperparameter search) and database search methods (HSS, SSS, BDC) mentioned in the paper"
- [methods] conda env create -f trainSpectus_build.yaml: "conda env create -f trainSpectus_build.yaml"
- [results] NIST dataset split into training, validation, and test sets with an 0.8 : 0.1 : 0.1 ratio, ensuring that duplicates were always placed in the same split.: [evidence span withheld — anchor status: not_found]
- [supplementary] Complete source code of SpecTUS is available on GitHub: https://github.com/hejjack/SpecTUS: "Complete source code of SpecTUS is available on GitHub: https://github.com/hejjack/SpecTUS"
