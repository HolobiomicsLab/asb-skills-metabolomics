---
name: win-rate-statistical-comparison
description: Use when ranked candidate structures from a de novo model and one or
  more database-search methods exist for the same spectra and you must quantify which
  method is better per spectrum and at k = 1, 10, 50.
when_to_use_negative:
- Comparing de novo models designed and trained for MS/MS spectra against GC-EI-MS
  results — the article notes such head-to-head comparison is challenging because
  all existing de novo approaches target MS/MS, not GC-EI-MS
- Predictions that cannot be decoded to SMILES, since the comparison metric is Tanimoto
  similarity of Morgan fingerprints computed from structures
- Spectra sets whose compounds overlap the training data — all test sets must be filtered
  to exclude compounds present in the synthetic or experimental training datasets
  before comparison
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0092
tools:
- name: SpecTUS
  role: Source of the de novo predictions being compared and of the evaluation notebooks/scripts
    that compute Acc_k and win rates
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Runs the repository's model-evaluation and model-comparison notebooks that
    recompute the metrics
  repo: https://github.com/ljocha/spectus-demo
- name: conda
  role: Creates the pinned Python environment (trainSpectus_build.yaml) required to
    execute the evaluation notebooks reproducibly
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:23795ac93b54054941e3f63bd1de9aff7117418df3f973e649639f492c27e9ea
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# win-rate-statistical-comparison

## Summary

Protocol for statistically comparing de novo structure predictions (e.g., SpecTUS) against database-search methods (HSS, SSS, BDC) on an EI-MS test set, using top-k accuracy (Acc_k), Morgan-fingerprint Tanimoto similarity, and per-spectrum win / at-least-as-good rates.

## When to use

Use when ranked candidate structures from a de novo model and one or more database-search methods exist for the same spectra and you must quantify which method is better per spectrum and at k = 1, 10, 50.

## When NOT to use

- Comparing de novo models designed and trained for MS/MS spectra against GC-EI-MS results — the article notes such head-to-head comparison is challenging because all existing de novo approaches target MS/MS, not GC-EI-MS
- Predictions that cannot be decoded to SMILES, since the comparison metric is Tanimoto similarity of Morgan fingerprints computed from structures
- Spectra sets whose compounds overlap the training data — all test sets must be filtered to exclude compounds present in the synthetic or experimental training datasets before comparison

## Inputs

- Deposited ranked predictions of all models and database search methods (HSS, SSS, BDC) from the SpecTUS predictions/ directory
- NIST 20 held-out test split of 28,267 spectra
- Ground-truth SMILES strings for the test compounds (from data/)
- Repository evaluation notebooks from the SpecTUS notebooks/ directory

## Outputs

- spectus_vs_hss_sss_metrics.csv with recomputed Acc_k and average top-k Tanimoto similarity for k = 1, 10, 50
- Per-k win rate and at-least-as-good rate of SpecTUS versus HSS
- Head-to-head comparison table matching the paper's supplementary metrics

## How to apply

Clone github.com/hejjack/SpecTUS and build the environment with 'conda env create -f trainSpectus_build.yaml'. Load deposited predictions for all models and database-search methods (HSS, SSS, BDC) from the predictions/ directory together with the NIST 20 test split (28,267 spectra) and ground-truth SMILES from data/. Using the repository's evaluation notebooks, compute top-k accuracy (Acc_k) and average top-k similarity — the highest Tanimoto similarity of Morgan fingerprints between the ground truth and the first k candidates — for k = 1, 10, and 50. For each k, compute the win rate and at-least-as-good rate of SpecTUS versus HSS, treating an exact structural match as the best outcome and otherwise comparing Morgan-fingerprint Tanimoto similarity; record all recomputed metrics to spectus_vs_hss_sss_metrics.csv. The rationale is that a single accuracy figure hides head-to-head behavior: on 76% of spectra the top-1 suggestion is strictly better than hybrid search even though absolute Acc1 is 43%.

## Related tools

- **SpecTUS** (Source of the de novo predictions being compared and of the evaluation notebooks/scripts that compute Acc_k and win rates) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Runs the repository's model-evaluation and model-comparison notebooks that recompute the metrics) — https://github.com/ljocha/spectus-demo
- **conda** (Creates the pinned Python environment (trainSpectus_build.yaml) required to execute the evaluation notebooks reproducibly)

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Recomputed Acc1 on the NIST 20 test split must be approximately 0.43 (43% perfect single-suggestion reconstructions)
- Recomputed Acc10 must be approximately 0.65 (65% perfect reconstructions with ten suggestions)
- SpecTUS-vs-HSS win rate at k=1 must be approximately 0.76 and at k=10 approximately 0.84, matching the abstract and supplementary tables
- HSS single-candidate success should land around 20%, providing a sanity anchor for the database-search baseline
- The output CSV must contain metrics for k = 1, 10, and 50 for both accuracy/similarity and win-rate columns

## Limitations

- Exact structural match is treated as the best outcome; otherwise ties and differences are judged only by Tanimoto similarity of Morgan fingerprints, which cannot resolve subtle isomer differences
- SpecTUS cannot explicitly justify its specific outputs for a given input and lacks intermediate representations such as peak annotations or fragmentation trees, so win rates do not capture explainability
- Model performance is sensitive to input spectra quality — performance is poorer on the Cayman and MONA test sets than on the more rigorously curated NIST or SWGDRUG libraries, so win rates computed on other test sets may differ

## Evidence
