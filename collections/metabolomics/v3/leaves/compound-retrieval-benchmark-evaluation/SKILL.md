---
name: compound-retrieval-benchmark-evaluation
description: Use when comparing C2MS spectrum-prediction models (e.g. FraGNNet-D4,
  ICEBERG, MassFormer, NEIMS) on a held-out MS/MS test set and you need retrieval-based
  compound identification accuracy rather than raw prediction error.
when_to_use_negative:
- You are evaluating formula annotation performance as a classification problem rather
  than full-spectrum retrieval ranking
- Your unknown spectra come from instruments/adducts outside the paper's scope (the
  paper flags that performance is not yet established for unmerged spectra, other
  instrument platforms, or precursor adducts beyond [M+H]+)
- You only need C2MS prediction error on real spectra without the compound-identification
  retrieval framing
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: C2MS model whose predicted spectra are ranked in the retrieval benchmark;
    primary model under evaluation
  repo: github.com/FraGNNet/fragnnet
- name: RDKit
  role: Computes chemical fingerprints and Tanimoto similarity used to select the
    49 PubChem candidate compounds for each test molecule
- name: Pytorch
  role: Framework in which FraGNNet and baseline models are implemented and used for
    spectrum prediction
- name: Pytorch Lightning
  role: Training/experiment orchestration framework used to produce and run the models
    evaluated in the retrieval task
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:c9d3d9b47977fb3fed30f5dede3557024c06bcb3f8e37b4daf6860682b3f9ec4
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
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# compound-retrieval-benchmark-evaluation

## Summary

Evaluates compound-to-spectrum (C2MS) models on a retrieval-based MS2C task by ranking candidate compounds against an unknown MS/MS spectrum and reporting Top-k accuracy. It quantifies whether predicted spectra improve compound identification over spectral libraries alone.

## When to use

Use when comparing C2MS spectrum-prediction models (e.g. FraGNNet-D4, ICEBERG, MassFormer, NEIMS) on a held-out MS/MS test set and you need retrieval-based compound identification accuracy rather than raw prediction error.

## When NOT to use

- You are evaluating formula annotation performance as a classification problem rather than full-spectrum retrieval ranking
- Your unknown spectra come from instruments/adducts outside the paper's scope (the paper flags that performance is not yet established for unmerged spectra, other instrument platforms, or precursor adducts beyond [M+H]+)
- You only need C2MS prediction error on real spectra without the compound-identification retrieval framing

## Inputs

- ~4,000 held-out NIST20 MS/MS test spectrum-molecule pairs (merged Orbitrap HCD, [M+H]+ adduct spectra)
- Molecular structures/fingerprints for each test compound and for PubChem candidate molecules
- Trained C2MS model checkpoints (FraGNNet-D4, ICEBERG, MassFormer, NEIMS) from the fragnnet framework
- Candidate sets of 50 compounds per test molecule (true compound + 49 Tanimoto-similar PubChem molecules)

## Outputs

- Per-model Top-1/Top-3/Top-5 accuracy percentages on the InChIKey and Scaffold splits, reported as mean ± standard deviation over 5 random seeds
- Rank of the true compound within each 50-compound candidate set under each spectral-similarity ranking metric (e.g. binned cosine, Hungarian cosine at 10 ppm, ± square-root intensity transform)
- Comparison table matching the paper's Table 2 retrieval accuracy values

## How to apply

For each of the ~4,000 held-out NIST20 MS/MS test spectrum-molecule pairs, construct a 50-compound candidate set consisting of the true compound X_i plus the 49 PubChem molecules with highest Tanimoto similarity to X_i, measured with RDKit chemical fingerprints. Predict an MS/MS spectrum for every candidate compound with each model under test, then score each predicted spectrum against the real spectrum Y_i using spectral-similarity ranking metrics such as binned cosine and Hungarian cosine at 10 ppm tolerance, with and without square-root intensity transformation; select the ranking metric that yields the best result per model. Rank the 50 candidates by spectral similarity, record the rank of the true compound, and compute Top-1/Top-3/Top-5 accuracy as percentages on both the InChIKey and Scaffold splits, reporting mean ± standard deviation over 5 random seeds. The workflow relies on the fragnnet codebase (Python/PyTorch, PyTorch Lightning), and validation is judged by matching reported values such as FraGNNet-D4 Top-1 = 36.0 ± 0.4 (InChIKey split) and 30.3 ± 0.6 (Scaffold split).

## Related tools

- **FraGNNet** (C2MS model whose predicted spectra are ranked in the retrieval benchmark; primary model under evaluation) — github.com/FraGNNet/fragnnet
- **RDKit** (Computes chemical fingerprints and Tanimoto similarity used to select the 49 PubChem candidate compounds for each test molecule)
- **Pytorch** (Framework in which FraGNNet and baseline models are implemented and used for spectrum prediction)
- **Pytorch Lightning** (Training/experiment orchestration framework used to produce and run the models evaluated in the retrieval task) — Lightning-AI/pytorch-lightning

## Examples

```
git clone https://github.com/FraGNNet/fragnnet && conda activate FRAGNNET-GPU && python scripts/run_retrieval_eval.py --n_candidates 50 --seed 0
```

## Evaluation signals

- FraGNNet-D4 Top-1 accuracy reproduces Table 2 values: 36.0 ± 0.4 on the InChIKey split and 30.3 ± 0.6 on the Scaffold split
- Each candidate set contains exactly 50 compounds: the true molecule X_i plus 49 PubChem molecules ranked by Tanimoto similarity of RDKit fingerprints
- Top-1/Top-3/Top-5 accuracies are reported as percentages with mean ± standard deviation over 5 random seeds on both InChIKey and Scaffold splits
- The spectral-similarity ranking metric is chosen per model as the best across binned cosine, Hungarian cosine at 10 ppm tolerance, with and without square-root intensity transformation
- FraGNNet surpasses ICEBERG, MassFormer, and NEIMS on the retrieval-based MS2C task, consistent with the state-of-the-art claim

## Limitations

- The recursive fragmentation algorithm depends on non-parallelized recursive edge-removal operations, limiting runtime for generating predictions over large candidate sets
- The fragmentation algorithm fails to account for some fragments, evidenced by unexplained OS peaks; these may arise from complex chemistry such as cyclizations
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)
- Retrieval evaluation used glycan- and peptide-excluded small-molecule spectra with annotations restricted to formula masses within 10 ppm of the peak mass

## Evidence

- [methods] Retrieval candidate construction by chemical similarity: [evidence span withheld — anchor status: not_found]
- [methods] Tanimoto-similarity-based candidate filtering: [evidence span withheld — anchor status: not_found]
- [methods] Retrieval-based MS2C evaluation task: "Each model was also evaluated in a retrieval-based MS2C task"
- [methods] Evaluation dataset provenance: [evidence span withheld — anchor status: not_found]
- [readme] Tooling grounding from repository README: [evidence span withheld — anchor status: external_doc]
- [abstract] Headline finding supporting the benchmark: "FraGNNet achieves state-of-the-art performance in terms of prediction error and surpasses existing C2MS models as a tool for retrieval-based MS2C."
