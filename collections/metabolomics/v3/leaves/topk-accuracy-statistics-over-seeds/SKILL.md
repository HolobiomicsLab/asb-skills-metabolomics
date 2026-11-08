---
name: topk-accuracy-statistics-over-seeds
description: Use when benchmarking compound-to-spectrum (C2MS) prediction models on
  a retrieval-based compound identification task where candidate sets are built from
  PubChem neighbors and Top-k hit rates must be reported with seed-level variance.
when_to_use_negative:
- The task is formula annotation or peak-prediction error evaluation rather than retrieval-based
  MS2C ranking
- Candidate sets cannot be constructed from PubChem fingerprints (e.g. the query molecule
  has no PubChem coverage), so Tanimoto-similar negatives are unavailable
- Only a single fixed split with one candidate ranking is needed and no seed-level
  variance reporting is intended
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: C2MS model whose predicted spectra are used to rank candidate compounds in
    the retrieval-based MS2C task
  repo: github.com/FraGNNet/fragnnet
- name: RDKit
  role: Computes chemical fingerprints and Tanimoto similarity used to build the 49-molecule
    PubChem candidate sets
- name: Pytorch
  role: Implementation framework for loading and running the C2MS models to predict
    candidate spectra
- name: Pytorch Lightning
  role: Training/experiment framework used with the FraGNNet codebase
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:e879bb2c33b5e964fa2b97dfc3246d1778616fa7170fb2a8508294db8fc78279
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - Python
  - Pytorch
  - Pytorch Lightning
  - RDKit
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# topk-accuracy-statistics-over-seeds

## Summary

Evaluates retrieval-based compound identification (MS2C) by ranking candidate compounds by spectral similarity to a real MS/MS spectrum and computing Top-1/Top-3/Top-5 accuracy as mean ± standard deviation over 5 random seeds. Used to compare C2MS spectrum-prediction models against spectral-library-only retrieval on the NIST20 held-out test set.

## When to use

Use when benchmarking compound-to-spectrum (C2MS) prediction models on a retrieval-based compound identification task where candidate sets are built from PubChem neighbors and Top-k hit rates must be reported with seed-level variance.

## When NOT to use

- The task is formula annotation or peak-prediction error evaluation rather than retrieval-based MS2C ranking
- Candidate sets cannot be constructed from PubChem fingerprints (e.g. the query molecule has no PubChem coverage), so Tanimoto-similar negatives are unavailable
- Only a single fixed split with one candidate ranking is needed and no seed-level variance reporting is intended

## Inputs

- ~4,000 held-out NIST20 MS/MS test spectrum-molecule pairs
- 50-molecule candidate sets per test molecule (true compound + 49 PubChem molecules selected by Tanimoto similarity of RDKit chemical fingerprints)
- Predicted MS/MS spectra for all candidates from each C2MS model (FraGNNet-D4, ICEBERG, MassFormer, NEIMS)
- Real test spectrum Y_i as the ranking reference
- Spectral-similarity metric configurations (binned cosine, Hungarian cosine at 10 ppm tolerance, with/without square-root intensity transformation)

## Outputs

- Table of Top-1/Top-3/Top-5 accuracy percentages (mean ± std over 5 random seeds) per model per split (InChIKey and Scaffold)
- Per-candidate-set rank of the true compound under each spectral-similarity ranking method

## How to apply

For each of the ~4,000 held-out NIST20 MS/MS test spectrum-molecule pairs X_i, build a 50-compound candidate set C_i containing X_i plus the 49 PubChem molecules with highest Tanimoto similarity to X_i as measured by RDKit chemical fingerprints. Predict an MS/MS spectrum for every candidate with each model (e.g. FraGNNet-D4, ICEBERG, MassFormer, NEIMS), score each predicted spectrum against the real spectrum Y_i using spectral-similarity metrics such as binned cosine and Hungarian cosine at 10 ppm tolerance (with and without square-root intensity transformation), and rank the 50 candidates. Record the rank of the true compound and compute Top-1/Top-3/Top-5 accuracy as percentages. Repeat over 5 random seeds (seed affects candidate sampling) and report mean ± standard deviation, separately for the InChIKey and Scaffold splits.

## Related tools

- **FraGNNet** (C2MS model whose predicted spectra are used to rank candidate compounds in the retrieval-based MS2C task) — github.com/FraGNNet/fragnnet
- **RDKit** (Computes chemical fingerprints and Tanimoto similarity used to build the 49-molecule PubChem candidate sets)
- **Pytorch** (Implementation framework for loading and running the C2MS models to predict candidate spectra)
- **Pytorch Lightning** (Training/experiment framework used with the FraGNNet codebase) — Lightning-AI/pytorch-lightning

## Examples

```
python scripts/... # via the fragnnet repo: conda activate FRAGNNET-GPU && python -m fragnnet.runner --config config/<retrieval_eval>.yaml (see config/README.md)
```

## Evaluation signals

- Top-1 accuracy for FraGNNet-D4 matches Table 2 values: 36.0 ± 0.4 on the InChIKey split and 30.3 ± 0.6 on the Scaffold split
- Every candidate set contains exactly 50 compounds: the true molecule X_i plus 49 Tanimoto-nearest PubChem molecules
- Standard deviations are computed across 5 random seeds and are small (sub-1-percentage-point range in the reference table)
- Ranking metric selection is recorded per model, i.e. the best-performing metric (e.g. Hungarian cosine at 10 ppm) is identified before reporting final accuracies
- FraGNNet-D4 surpasses the baseline C2MS models (ICEBERG, MassFormer, NEIMS) on Top-k accuracy

## Limitations

- Only 50-compound candidate sets were tested; performance on larger candidate libraries is not characterized here
- Retrieval evaluation depends on candidate-set construction quality — candidates come from PubChem and are selected by Tanimoto similarity, so results are sensitive to the random seed used for sampling
- FraGNNet's underlying predictions were evaluated on merged NIST20 Orbitrap HCD [M+H]+ spectra; performance under other instrument platforms and precursor adducts has not been established

## Evidence

- [methods] a candidate set C_i is constructed from X_i and 49 other molecules sampled from PubChem (Kim et al., 2019): [evidence span withheld — anchor status: not_found]
- [methods] Each C_i in C_i - {X_i} is selected to have high chemical similarity with X_i as measured by Tanimoto similarity between chemical fingerprints: [evidence span withheld — anchor status: not_found]
- [methods] We evaluated C2MS performance on a held-out portion of the NIST20 MS/MS dataset (Appendix A.9): [evidence span withheld — anchor status: not_found]
