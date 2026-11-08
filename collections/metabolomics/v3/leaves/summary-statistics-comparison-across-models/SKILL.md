---
name: summary-statistics-comparison-across-models
description: Use when you need to quantify and contrast the breadth of predicted formula
  distributions from two spectrum-prediction models on a shared held-out test set,
  e.
when_to_use_negative:
- When the goal is spectrum prediction quality (peak error, MS2C retrieval Top-k)
  rather than characterizing the size/breadth of the predicted formula support
- When comparing models that both draw from identical fixed formula vocabularies —
  support-size contrast is uninformative and will reflect vocabulary construction,
  not model behavior
- When the input spectra are glycan or peptide spectra, which the article excluded
  from formula-annotation analysis to focus on small molecules
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FraGNNet
  role: Source of the learned fragment-induced formula distribution P_theta(f); inference
    on held-out NIST20 data yields the per-molecule support to be summarized
  repo: github.com/FraGNNet/fragnnet
- name: GrAFF-MS
  role: Reference C2MS model whose fixed formula support (top 10,000 product and neutral-loss
    formulae) provides the comparison baseline
- name: Python
  role: Implementation language for the support-size counting and summary-statistics
    pipeline (pandas/numpy aggregation, CSV export)
- name: RDKit
  role: Chemical-validity filtering of counted formulae before support-size statistics
    are computed
- name: Pytorch
  role: Framework in which FraGNNet inference over the fragmentation DAG is executed
    to obtain the marginalized formula distribution
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:459879292af0ab25fbfa1015ca1139454cce3ab5fbba6d72aceec371de2dc3ce
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - GrAFF-MS
  - Python
  - Pytorch
  - RDKit
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# summary-statistics-comparison-across-models

## Summary

Computes and compares per-molecule summary statistics (median, quartiles, full support-size distribution) for formula-support sizes between two C2MS models — the fragment-induced FraGNNet distribution and the fixed-vocabulary GrAFF-MS support — quantifying how each model parameterizes its predicted formula distribution.

## When to use

Use when you need to quantify and contrast the breadth of predicted formula distributions from two spectrum-prediction models on a shared held-out test set, e.g. checking whether a model's support is a learned fragment-induced distribution (~679 formulae) versus a fixed library vocabulary (~10,000 formulae).

## When NOT to use

- When the goal is spectrum prediction quality (peak error, MS2C retrieval Top-k) rather than characterizing the size/breadth of the predicted formula support
- When comparing models that both draw from identical fixed formula vocabularies — support-size contrast is uninformative and will reflect vocabulary construction, not model behavior
- When the input spectra are glycan or peptide spectra, which the article excluded from formula-annotation analysis to focus on small molecules

## Inputs

- Trained FraGNNet-D4 model checkpoint (github.com/FraGNNet/fragnnet) with fragmentation DAG parameters d=4, j=4
- Held-out portion of the NIST20 MS/MS dataset (spectra exported from the NIST20 MS/MS spectral library, [M+H]+ adduct, Orbitrap HCD)
- GrAFF-MS static formula library: top 10,000 product and neutral-loss formulae ranked by total intensity across spectra
- NIST20 expert-curated formula annotations for chemical-validity filtering
- Precursor formulae of test molecules for neutral-loss complement construction

## Outputs

- Per-molecule support sizes for FraGNNet and GrAFF-MS (counts of nonzero-probability formulae, valid formulae only)
- formula_support_stats.csv — summary statistics table (median, quartiles, support-size distribution per model)

## How to apply

Run trained FraGNNet-D4 inference on the held-out portion of the NIST20 MS/MS dataset to obtain the marginalized formula distribution P_theta(f) per test molecule, derived from the approximate fragmentation DAG M_hat(G,d,j) with depth d in {3,4} and hydrogen tolerance j=4. Independently construct the GrAFF-MS formula support per molecule as the union of the static library's product formulae and the complements of neutral-loss formulae with the precursor formula, taking the top 10,000 product and neutral-loss formulae by total intensity. For each molecule and each model, count formulae with nonzero probability (support size), excluding chemically invalid formulae using RDKit-based chemical-validity filtering and excluding annotations more than 10 ppm from their associated peak mass. Compute per-model summary statistics — median, quartiles, and the full support-size distribution — over the test set with pandas/numpy, and compare the two distributions. The FraGNNet median support size should be approximately 679 formulae and the GrAFF-MS support approximately 10,000 formulae per compound, reflecting that FraGNNet draws from a learned distribution over molecule fragments rather than a fixed formula vocabulary.

## Related tools

- **FraGNNet** (Source of the learned fragment-induced formula distribution P_theta(f); inference on held-out NIST20 data yields the per-molecule support to be summarized) — github.com/FraGNNet/fragnnet
- **GrAFF-MS** (Reference C2MS model whose fixed formula support (top 10,000 product and neutral-loss formulae) provides the comparison baseline)
- **Python** (Implementation language for the support-size counting and summary-statistics pipeline (pandas/numpy aggregation, CSV export))
- **RDKit** (Chemical-validity filtering of counted formulae before support-size statistics are computed)
- **Pytorch** (Framework in which FraGNNet inference over the fragmentation DAG is executed to obtain the marginalized formula distribution) — github.com/FraGNNet/fragnnet

## Examples

```
import pandas as pd, numpy as np; df = pd.read_parquet('support_sizes.parquet'); stats = df[['fragnnet_support','graffms_support']].describe(percentiles=[.25,.5,.75]); stats.to_csv('formula_support_stats.csv')
```

## Evaluation signals

- FraGNNet median support size per compound is approximately 679 formulae and GrAFF-MS approximately 10,000 formulae, matching Section 5.3 / Table 4 of the article
- Per-molecule support counts contain only chemically valid formulae (RDKit filter applied) with formula masses within 10 ppm of the associated peak mass
- GrAFF-MS support sizes cluster at the 10,000 upper bound set by the top-10,000 formulae selection, while FraGNNet support sizes show a distribution reflecting fragment-DAG structure (d=4, j=4)
- Summary table written to formula_support_stats.csv contains median and quartile columns for both models over the full held-out test set
- Support sizes are computed over the same set of test molecules for both models (paired comparison, no molecule dropped on one side)

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, limiting inference runtime over large test sets
- The algorithm fails to account for some fragments, evidenced by unexplained Outside-of-the-Support (OS) peaks; complex chemistry such as cyclizations dramatically expands the fragment search space and is challenging to model
- Performance was evaluated on merged spectra under a single experimental condition (Orbitrap HCD, [M+H]+ adduct); broader instrument platforms, adducts, and unmerged spectra are untested

## Evidence

- [methods] DAG parameters capture peak intensity: [evidence span withheld — anchor status: not_found]
- [readme] README: fragment-based spectrum prediction: [evidence span withheld — anchor status: external_doc]
