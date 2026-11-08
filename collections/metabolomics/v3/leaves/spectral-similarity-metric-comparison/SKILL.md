---
name: spectral-similarity-metric-comparison
description: 'Decision support for retrieval-based MS2C compound identification: use
  when scoring candidate spectra against a reference MS/MS spectrum with spectral
  similarity metrics (COS, SPC, entropy) to rank candidates and measure retrieval
  rates, especially when augmenting incomplete libraries with.'
when_to_use_negative:
- When the unknown spectrum already matches an entry in the spectral library — direct
  library search (MS2C against NIST20) is sufficient and this skill adds unnecessary
  C2MS modeling overhead
- When the goal is de novo fragment annotation or formula annotation (classification
  task) rather than ranking candidates by spectral similarity
- When spectra come from unmerged acquisitions, different instrument platforms (not
  Orbitrap/HCD), or different precursor adducts than [M+H]+ — the paper notes performance
  has not been validated under these broader conditions
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: Deep probabilistic C2MS model that predicts MS/MS spectra for candidate molecules,
    whose predicted spectra are then scored against a reference spectrum using the
    spectral similarity metrics; the model was implemented in Python with PyTorch
  repo: github.com/FraGNNet/fragnnet
- name: PyTorch
  role: Framework used to implement FraGNNet and the similarity scoring pipeline
  repo: https://pytorch.org/
- name: PyTorch Lightning
  role: Training and experiment management framework for the FraGNNet model whose
    spectra feed the similarity comparison
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: RDKit
  role: Molecule featurization and Tanimoto-similarity fingerprint calculation used
    to construct the candidate set C_i for retrieval
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:2a7abd4924fe31a3f2ddc8a20d78a394a77f02433d53d9d0ac0f47f1f0ea2c6b
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - NumPy
  - Pandas
  - PyTorch
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectral-similarity-metric-comparison

## Summary

Compares tandem mass spectrometry (MS/MS) spectra from candidate spectra using spectral similarity metrics (COS, SPC, entropy-based metrics) to score candidate spectra against a reference spectrum, supporting retrieval-based compound identification (MS2C) workflows where library coverage is incomplete.

## When to use

Decision support for retrieval-based MS2C compound identification: use when scoring candidate spectra against a reference MS/MS spectrum with spectral similarity metrics (COS, SPC, entropy) to rank candidates and measure retrieval rates, especially when augmenting incomplete libraries with predicted C2MS spectra.

## When NOT to use

- When the unknown spectrum already matches an entry in the spectral library — direct library search (MS2C against NIST20) is sufficient and this skill adds unnecessary C2MS modeling overhead
- When the goal is de novo fragment annotation or formula annotation (classification task) rather than ranking candidates by spectral similarity
- When spectra come from unmerged acquisitions, different instrument platforms (not Orbitrap/HCD), or different precursor adducts than [M+H]+ — the paper notes performance has not been validated under these broader conditions

## Inputs

- Reference MS/MS spectrum for the unknown compound (m/z, intensity pairs, e.g. from NIST20 merged spectra)
- Candidate spectra predicted by a C2MS model (e.g. FraGNNet) or exported from NIST20 library
- Candidate set C_i (input molecule X_i + 49 Tanimoto-similar PubChem molecules)
- Held-out split specification (InChIKey or Scaffold) and 5 random seeds

## Outputs

- Top-k rank of the true compound per candidate set (retrieval accuracy)
- Table 1-style CSV: metric (C_BIN, C_HUN), split (InChIKey, Scaffold), mean ± standard deviation, and delta between models
- Retrieval rate curves or Top-10 accuracy values over 5 seeds

## How to apply

For each candidate spectrum (predicted by a C2MS model such as FraGNNet, or retrieved from a spectral library like NIST20), compute spectral similarity against the reference MS/MS spectrum using three metrics reported in the paper: cosine similarity (COS), Stein–Scott dot product (SPC), and entropy-based similarity (ENT). Score all candidates in the retrieval candidate set C_i, which is constructed from the input molecule X_i plus 49 other molecules sampled from PubChem with high Tanimoto similarity to X_i, then rank candidates by similarity and record the top-k rank of the true compound. Evaluate retrieval on two held-out splits of NIST20 (InChIKey and Scaffold), reporting mean and standard deviation over 5 random seeds; the retrieval task is scored with Top-10 accuracy (e.g. FraGNNet-D4 achieves C_BIN 0.736±0.002 on InChIKey) and Top-10% recall. When comparing two models, compute deltas (e.g. D4 minus D3 gives +0.015 to +0.021 across metric/split combinations) and confirm the deeper fragmentation depth (d=4) outperforms d=3 on all metric–split combinations. Use the results as a Table-1-style CSV of metric, split, mean, standard deviation, and delta to communicate performance differences.

## Related tools

- **FraGNNet** (Deep probabilistic C2MS model that predicts MS/MS spectra for candidate molecules, whose predicted spectra are then scored against a reference spectrum using the spectral similarity metrics; the model was implemented in Python with PyTorch) — github.com/FraGNNet/fragnnet
- **PyTorch** (Framework used to implement FraGNNet and the similarity scoring pipeline) — https://pytorch.org/
- **PyTorch Lightning** (Training and experiment management framework for the FraGNNet model whose spectra feed the similarity comparison) — https://github.com/Lightning-AI/pytorch-lightning
- **RDKit** (Molecule featurization and Tanimoto-similarity fingerprint calculation used to construct the candidate set C_i for retrieval) — https://www.rdkit.org/

## Examples

```
python scripts/run_retrieval.py --model fragnnet_d4 --splits inchikey,scaffold --seeds 5 --metrics cos,spc,ent --output table1.csv
```

## Evaluation signals

- Deltas between models must be computed consistently: verify D4 − D3 for C_BIN and C_HUN on InChIKey and Scaffold splits matches reported values (+0.015, +0.016, +0.019, +0.021) and that D4 > D3 holds for all four metric–split combinations
- Reported values must match the published Table 1 means and standard deviations over 5 random seeds exactly (e.g. FraGNNet-D4 C_BIN 0.736±0.002 on InChIKey)
- Candidate set must contain exactly 50 molecules (X_i plus 49 sampled PubChem molecules) and be ranked by the similarity metric; the true compound's top-k rank should be recoverable from the output
- Retrieval accuracy must be reported on both held-out splits (InChIKey and Scaffold) to confirm generalization beyond exact structural matches

## Limitations

- The fragmentation algorithm relies on recursive edge-removal operations that are not parallelized, limiting runtime when generating candidate spectra at scale
- The algorithm fails to account for some fragments, as evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry (e.g. cyclizations) that dramatically expands the fragment search space and is challenging to model
- Performance has not yet been evaluated on unmerged spectra or under broader experimental conditions, including different instrument platforms and precursor adducts

## Evidence

- [methods] Candidate set construction via Tanimoto-similar PubChem sampling: [evidence span withheld — anchor status: not_found]
- [methods] Evaluation was performed on held-out NIST20: [evidence span withheld — anchor status: not_found]
- [methods] Recursive bond-breaking generates the fragment candidates: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime bottleneck from non-parallelized recursion: [evidence span withheld — anchor status: not_found]
- [readme] Code availability for reproducing retrieval experiments: [evidence span withheld — anchor status: not_found]
