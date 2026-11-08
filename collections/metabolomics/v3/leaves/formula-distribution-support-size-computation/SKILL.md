---
name: formula-distribution-support-size-computation
description: Use when comparing formula-support breadth between C2MS models (e.g.
  FraGNNet's ~679 formulae median vs GrAFF-MS's ~10,000) on held-out NIST20 MS/MS
  spectra, or when validating that a predicted formula distribution is derived from
  fragmentation rather than a fixed vocabulary.
when_to_use_negative:
- Input spectra are already formula-annotated and only peak-level error is needed
  — use the article's IS/OS peak decomposition or formula annotation classification
  instead
- The candidate set is a retrieval/MS2C Top-k ranking task rather than a formula-support-size
  analysis
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: Produces the fragment-induced formula distribution P_theta(f) from the approximate
    fragmentation DAG whose support size is counted
  repo: github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: Baseline whose fixed top-10,000 formula vocabulary defines the comparison
    support size
- name: RDKit
  role: Data preprocessing and molecule featurization for NIST20 spectra before formula
    distribution computation
- name: PyTorch
  role: Inference backend for FraGNNet-D4 on the held-out NIST20 test set
  repo: Lightning-AI/pytorch-lightning
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:8fd155bbad098afb5d5cbd7d97775c3b8cef0535499aadd6216eaa0f9bd27aa2
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - GrAFF-MS
  - PyTorch
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# formula-distribution-support-size-computation

## Summary

Computes and compares the number of molecular formulae with nonzero probability per test molecule for FraGNNet versus GrAFF-MS, quantifying how formula support size differs between a learned fragment-induced formula distribution and a fixed formula vocabulary. This supports the paper's claim that FraGNNet's formula distribution is derived from an approximate fragmentation DAG rather than a large fixed vocabulary.

## When to use

Use when comparing formula-support breadth between C2MS models (e.g. FraGNNet's ~679 formulae median vs GrAFF-MS's ~10,000) on held-out NIST20 MS/MS spectra, or when validating that a predicted formula distribution is derived from fragmentation rather than a fixed vocabulary.

## When NOT to use

- Input spectra are already formula-annotated and only peak-level error is needed — use the article's IS/OS peak decomposition or formula annotation classification instead
- The candidate set is a retrieval/MS2C Top-k ranking task rather than a formula-support-size analysis

## Inputs

- Trained FraGNNet-D4 model checkpoint (github.com/FraGNNet/fragnnet)
- Held-out portion of NIST20 MS/MS dataset with precursor formulae
- GrAFF-MS static library of product and neutral-loss formulae (top 10,000 by total intensity)

## Outputs

- Per-molecule formula support sizes for FraGNNet and GrAFF-MS
- Summary statistics table (median, quartiles, support-size distribution) written to formula_support_stats.csv

## How to apply

Run trained FraGNNet-D4 inference on the held-out NIST20 MS/MS test set to obtain the marginalized formula distribution P_theta(f), derived from the approximate fragmentation DAG M_hat(G,d,j) with depth d=4 and hydrogen tolerance j=4. Separately construct the GrAFF-MS formula support per test molecule as the union of the top 10,000 product formulae and neutral-loss complements of the precursor formula, selected by total intensity across spectra. For each model, count formulae with nonzero probability (support size) per molecule, excluding formulae whose masses are more than 10 ppm from any peak mass and excluding glycan/peptide spectra. Aggregate median and quartile support-size statistics over the test set with pandas/numpy and compare distributions; the article reports FraGNNet at approximately 679 formulae and GrAFF-MS at approximately 10,000 formulae per compound.

## Related tools

- **FraGNNet** (Produces the fragment-induced formula distribution P_theta(f) from the approximate fragmentation DAG whose support size is counted) — github.com/FraGNNet/fragnnet
- **GrAFF-MS** (Baseline whose fixed top-10,000 formula vocabulary defines the comparison support size)
- **RDKit** (Data preprocessing and molecule featurization for NIST20 spectra before formula distribution computation)
- **PyTorch** (Inference backend for FraGNNet-D4 on the held-out NIST20 test set) — Lightning-AI/pytorch-lightning

## Examples

```
python -c "import pandas as pd; d=pd.read_csv('formula_support_stats.csv'); print(d.groupby('model')['support_size'].describe())"
```

## Evaluation signals

- FraGNNet median support size should be approximately 679 formulae per compound
- GrAFF-MS support size should be approximately 10,000 formulae per compound
- No formula in the counted support exceeds 10 ppm mass error from an associated peak
- Glycan and peptide spectra must be excluded before aggregation
- Support-size distribution quartiles computed with pandas/numpy over the full test set

## Limitations

- Cases where only a single fragment is possible are excluded from the agreement analysis, biasing support-size counts toward molecules with richer fragmentation DAGs
- Unexplained Outside-of-Support (OS) peaks indicate the fragmentation algorithm fails to account for some fragments, so the learned formula distribution may be too narrow
- The recursive bond-breaking algorithm depends on sequential edge-removal operations, limiting runtime on large molecules
- Formula-support breadth has not been evaluated on unmerged spectra or under broader instrument platforms and precursor adducts

## Evidence

- [results] FraGNNet does not draw from a fixed formula vocabulary; instead it learns a distribution over molecule fragments, from which the formula distribution underlying the predicted spectrum is derived.: [evidence span withheld — anchor status: not_found]
- [methods] calculating M_hat(G,d,j) with d in {3,4} and j=4 can effectively capture most of the total peak intensity: [evidence span withheld — anchor status: not_found]
