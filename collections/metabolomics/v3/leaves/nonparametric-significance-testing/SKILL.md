---
name: nonparametric-significance-testing
description: Use when two MS/MS prediction models yield per-compound cosine-similarity
  scores against the same experimental Q-TOF test set and you must claim one significantly
  outperforms the other per ion mode ([M+H]+, [M-H]-).
when_to_use_negative:
- Only a single model's mean cosine similarity is needed and no baseline superiority
  claim is being made — simple aggregation of eval_results.csv suffices
- The compared models were evaluated on different test compounds or splits — the two
  similarity distributions are then not comparable samples and the U test is meaningless
- Only aggregated summary statistics (means/std) are available rather than per-compound
  scores — the rank-based test requires the full distribution
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SciPy
  role: Implements the Mann-Whitney U test comparing 3DMolMS's per-compound similarity
    distribution against each baseline, per ion mode
- name: molnetpack (3DMolMS)
  role: molnet_engine.evaluate(...) writes the per-compound cosine similarity CSV
    (result_path) that supplies the distributions fed into the U tests
  repo: https://github.com/JosieHong/3DMolMS
- name: matplotlib
  role: Assembles the model-comparison figure annotated with the U-test significance
    outcomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:3c8c045fa99ad0da60549d63f7ee11ed0435cbc92d65b1f77c3f608bc7395417
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - CFM-ID 4.0
  - DGCNN
  - MassFormer
  - NEIMS
  - NumPy
  - Pandas
  - PointNet
  - PyTorch
  - RDKit
  - SchNet
  - SciPy
  - matplotlib
  - molnetpack
  - molnetpack (3DMolMS)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# nonparametric-significance-testing

## Summary

Run Mann-Whitney U (Whitney U) tests on per-compound cosine-similarity distributions, separately per ion mode, to establish whether 3DMolMS significantly outperforms MS/MS prediction baselines; appropriate because the similarity distributions are broad/skewed, so means alone are not decisive evidence.

## When to use

Use when two MS/MS prediction models yield per-compound cosine-similarity scores against the same experimental Q-TOF test set and you must claim one significantly outperforms the other per ion mode ([M+H]+, [M-H]-).

## When NOT to use

- Only a single model's mean cosine similarity is needed and no baseline superiority claim is being made — simple aggregation of eval_results.csv suffices
- The compared models were evaluated on different test compounds or splits — the two similarity distributions are then not comparable samples and the U test is meaningless
- Only aggregated summary statistics (means/std) are available rather than per-compound scores — the rank-based test requires the full distribution

## Inputs

- Per-compound cosine similarity table for 3DMolMS vs experimental Q-TOF spectra (eval_results.csv produced via scripts/eval.py --result_path or molnet_engine.evaluate)
- Per-compound cosine similarity tables for each baseline (CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN) predicted under the same adducts and collision energies
- Ion-mode labels ([M+H]+ / [M-H]-) per test compound from the shared held-out 10% InChIKey-level split

## Outputs

- U statistics and p-values per baseline per ion mode
- Significance annotations for the model-comparison figure and final benchmark table
- Mean ± std cosine similarity per model per ion mode (e.g. 0.691±0.361 for [M+H]+, 0.478±0.379 for [M-H]-)

## How to apply

First compute per-compound cosine similarities for every model against the experimental ground truth on the same held-out 10% compound split — scripts/eval.py with --result_path (or molnet_engine.evaluate(test_pkl=..., pred_mgf=..., result_path='./eval_results.csv')) writes these per-compound results. Because the cosine-similarity distributions are wide and skewed (3DMolMS: 0.691±0.361 for [M+H]+, 0.478±0.379 for [M-H]-), apply the rank-based Mann-Whitney U test (SciPy) rather than a parametric test. Run one U test per baseline per ion mode, comparing 3DMolMS's similarity distribution against each of CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, and DGCNN, never pooling the two ion modes. Report U statistics and p-values alongside mean±std per model and annotate the model-comparison figure; the expected outcome is significance against all baselines in positive mode but only against NEIMS and CFM-ID in negative mode. The comparison is only valid if train/val/test were split at the non-stereochemical InChIKey skeleton level so no structure leaks between models' test sets.

## Related tools

- **SciPy** (Implements the Mann-Whitney U test comparing 3DMolMS's per-compound similarity distribution against each baseline, per ion mode)
- **molnetpack (3DMolMS)** (molnet_engine.evaluate(...) writes the per-compound cosine similarity CSV (result_path) that supplies the distributions fed into the U tests) — https://github.com/JosieHong/3DMolMS
- **matplotlib** (Assembles the model-comparison figure annotated with the U-test significance outcomes)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(sim_3dmolms_pos, sim_neims_pos)
```

## Evaluation signals

- One U test per baseline per ion mode with no pooled across-mode comparison
- Reproduced directionality: 3DMolMS significant vs all baselines (CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN) in [M+H]+ mode, but vs only NEIMS and CFM-ID in [M-H]- mode — any other outcome signals a pipeline error
- 3DMolMS similarity aggregates match the reported 0.691(±0.361) for [M+H]+ and 0.478(±0.379) for [M-H]- on the held-out 10% split
- Each distribution contains one score per test compound, with all models predicted under identical adducts and collision energies
- p-values are reproducible from scipy.stats.mannwhitneyu on the raw per-compound arrays

## Limitations

- In negative ion mode the advantage was significant only against NEIMS and CFM-ID, not against all baselines
- The test compares unpaired distributions rather than exploiting per-compound pairing, and the wide spread (std 0.361 / 0.379) means small mean gaps alone are weak evidence
- Validity depends on the non-stereochemical InChIKey skeleton-level split; structure leakage between train/val/test would inflate the significance comparison

## Evidence

- [results] Whitney U-test confirms 3DMolMS outperforms baselines per ion mode: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: run Mann-Whitney U test per baseline: [evidence span withheld — anchor status: not_found]
- [abstract] Average cosine similarity 0.691 / 0.478 in positive / negative ion modes: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [readme] Per-compound similarity CSV written for downstream testing: "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv""
