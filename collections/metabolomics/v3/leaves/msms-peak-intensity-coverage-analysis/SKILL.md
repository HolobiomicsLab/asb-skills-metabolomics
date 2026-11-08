---
name: msms-peak-intensity-coverage-analysis
description: Use when you need to quantify how much of the total peak intensity of
  real MS/MS spectra (e.g., NIST20, [M+H]+, HCD Orbitrap) is explained by an approximate
  fragmentation DAG with given depth d and hydrogen tolerance j.
when_to_use_negative:
- The goal is spectrum prediction or retrieval (MS2C/C2MS) rather than validating
  the DAG mass support — use the full FraGNNet/GNN pipeline instead of the coverage
  statistic
- Spectra come from non-merged data, other instrument platforms, or other precursor
  adducts — the paper only evaluated this setting on merged Orbitrap HCD [M+H]+ spectra
- The analysis targets glycan or peptide spectra — the paper excluded glycan and peptide
  spectra from its small-molecule formula analyses
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Parses precursor molecules into molecular graphs and strips hydrogen atoms
    to build the heavy atom skeleton H(G) before fragmentation
- name: Cython
  role: Hosts the recursive bond-breaking algorithm (Algorithm 1) that generates the
    approximate fragmentation DAG G_F^d from H(G)
- name: Python
  role: Implements the mass-enumeration, peak-matching, and coverage computation steps
    that produce coverage_summary.csv
- name: FraGNNet (fragnnet repo)
  role: Source codebase providing the data preprocessing, fragmentation algorithm,
    and analysis notebooks for reproducing the coverage result
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:d508225c1a8cebb7253e0ddfdf03b30298b8e9a34baa0ea223b769c3d37b57e8
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - FraGNNet (fragnnet repo)
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msms-peak-intensity-coverage-analysis

## Summary

Computes the coverage statistic M̂(G, d, j) (Definition 4.3), the fraction of total peak intensity in real NIST20 MS/MS spectra explained by the union mass set of an approximate fragmentation DAG, to validate that depth d in {3, 4} with hydrogen tolerance j = 4 captures most peak intensity.

## When to use

Use when you need to quantify how much of the total peak intensity of real MS/MS spectra (e.g., NIST20, [M+H]+, HCD Orbitrap) is explained by an approximate fragmentation DAG with given depth d and hydrogen tolerance j.

## When NOT to use

- The goal is spectrum prediction or retrieval (MS2C/C2MS) rather than validating the DAG mass support — use the full FraGNNet/GNN pipeline instead of the coverage statistic
- Spectra come from non-merged data, other instrument platforms, or other precursor adducts — the paper only evaluated this setting on merged Orbitrap HCD [M+H]+ spectra
- The analysis targets glycan or peptide spectra — the paper excluded glycan and peptide spectra from its small-molecule formula analyses

## Inputs

- Exported NIST20 MS/MS spectra with precursor molecular structures (merged spectra, Orbitrap HCD, [M+H]+ adduct)
- Molecular structures parseable by RDKit into molecular graphs
- DAG depth parameters d in {3, 4} and hydrogen tolerance j = 4

## Outputs

- Per-(d, j) coverage fractions (fraction of total peak intensity matched against M̂(G, d, 4)) written to coverage_summary.csv
- Approximate fragmentation DAG G_F^d per molecule for d = 3 and d = 4
- Union mass set M̂(G, d, 4) of enumerated monoisotopic formula masses per DAG node

## How to apply

Parse each NIST20 precursor molecule into a molecular graph with RDKit and strip hydrogens to obtain the heavy atom skeleton H(G). Run the recursive bond-breaking algorithm (Algorithm 1, Cython implementation) on H(G) for each depth d in {3, 4} to build the approximate fragmentation DAG G_F^d. For every DAG node n, enumerate monoisotopic formula masses over hydrogen counts h_n - j to h_n + j with j = 4, and take the union as the mass set M̂(G, d, 4). Match each real peak mass from the associated NIST20 spectra against this union set and compute the intensity-weighted fraction covered, separately for d = 3 and d = 4, writing per-(d, j) coverage fractions to coverage_summary.csv. The rationale is that the paper's Definition 4.3 statistic empirically justifies the fragment/mass parameterization of FraGNNet: if high coverage holds at d in {3, 4}, j = 4, the DAG's mass support suffices for real spectra.

## Related tools

- **RDKit** (Parses precursor molecules into molecular graphs and strips hydrogen atoms to build the heavy atom skeleton H(G) before fragmentation)
- **Cython** (Hosts the recursive bond-breaking algorithm (Algorithm 1) that generates the approximate fragmentation DAG G_F^d from H(G))
- **Python** (Implements the mass-enumeration, peak-matching, and coverage computation steps that produce coverage_summary.csv)
- **FraGNNet (fragnnet repo)** (Source codebase providing the data preprocessing, fragmentation algorithm, and analysis notebooks for reproducing the coverage result) — github.com/FraGNNet/fragnnet

## Evaluation signals

- Computed coverage fractions for d in {3, 4} and j = 4 are high, consistent with the paper's Table 4 finding that this setting 'captures most of the total peak intensity of real MS/MS data'
- Coverage at d = 4 is at least as high as at d = 3, since a deeper DAG can only add mass-set elements
- Every per-molecule coverage fraction lies in [0, 1], and unmatched intensity corresponds to Outside of the Support (OS) peaks
- coverage_summary.csv contains one coverage entry per (d, j) pair, with d in {3, 4} and j = 4
- All enumerated node masses are monoisotopic formula masses with hydrogen counts within the tolerance window [h_n - j, h_n + j]

## Limitations

- The recursive edge-removal fragmentation algorithm is not parallelized, limiting runtime on large molecules
- The algorithm fails to account for some fragments, evidenced by unexplained OS (Outside of the Support) peaks in spectra
- Complex chemistry such as cyclizations can dramatically expand the fragment search space and is challenging to model, leaving residual peaks uncovered
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts)

## Evidence

- [methods] calculating M_hat(G,d,j) (Definition 4.3) with d in {3,4} and j=4 can effectively capture most of the total peak intensity in real MS/MS data: [evidence span withheld — anchor status: not_found]
- [methods] heavy atom skeleton definition: [evidence span withheld — anchor status: not_found]
- [methods] Cython fragmentation algorithm: [evidence span withheld — anchor status: not_found]
