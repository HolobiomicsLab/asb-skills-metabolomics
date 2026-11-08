---
name: molecular-formula-monoisotopic-mass-computation
description: Use when fragmenting a molecular graph into a fragmentation DAG (depth
  d in {3,4}) and you need per-node formula masses with hydrogen tolerance j=4 to
  measure total peak-intensity coverage of NIST20 MS/MS spectra.
when_to_use_negative:
- Spectra from instruments other than Orbitrap with HCD, or precursor adduct types
  other than [M+H]+, where the paper's coverage statistics were not validated
- Glycan or peptide spectra, which the paper explicitly excluded from formula annotation
  analysis
- Annotations whose formula mass deviates more than 10 ppm from the associated peak
  mass, which the paper filtered out
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: RDKit
  role: Parse molecular graphs and strip hydrogen atoms to obtain heavy atom skeleton
    H(G) for fragment enumeration
- name: Cython
  role: Implementation language for the recursive bond-breaking algorithm (Algorithm
    1) that generates fragment nodes and edges of the DAG
- name: Python
  role: Scripting environment for mass enumeration, peak matching, and writing coverage_summary.csv
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:53c92f8305f13343654239a96e0efa24065e5bddcf9a5a41f89c46259af3c180
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-formula-monoisotopic-mass-computation

## Summary

Enumerate monoisotopic formula masses for each fragment node in an approximate fragmentation DAG, allowing hydrogen counts to vary within tolerance j, to build the coverage mass set M̂(G, d, j) used to match real MS/MS peak masses.

## When to use

Use when fragmenting a molecular graph into a fragmentation DAG (depth d in {3,4}) and you need per-node formula masses with hydrogen tolerance j=4 to measure total peak-intensity coverage of NIST20 MS/MS spectra.

## When NOT to use

- Spectra from instruments other than Orbitrap with HCD, or precursor adduct types other than [M+H]+, where the paper's coverage statistics were not validated
- Glycan or peptide spectra, which the paper explicitly excluded from formula annotation analysis
- Annotations whose formula mass deviates more than 10 ppm from the associated peak mass, which the paper filtered out

## Inputs

- NIST20 MS/MS spectra with precursor molecular structures
- Molecular structures parsed as molecular graphs (RDKit)
- Approximate fragmentation DAG G_F^d with depth d in {3,4}
- Hydrogen tolerance parameter j = 4

## Outputs

- Union mass set M̂(G, d, 4) of monoisotopic formula masses per DAG node
- Fraction of total peak intensity covered for d = 3 and d = 4
- coverage_summary.csv with per-(d,j) coverage fractions

## How to apply

Parse each NIST20 precursor molecule into a molecular graph with RDKit, strip hydrogens to obtain the heavy atom skeleton H(G), then run the recursive Cython bond-breaking algorithm (Algorithm 1) at depth d in {3,4} to produce the approximate fragmentation DAG G_F^d. For every DAG node n, enumerate monoisotopic formula masses m_n^i for hydrogen counts h_n - j through h_n + j with tolerance j = 4, forming the union mass set M̂(G, d, 4) per Definition 4.3. Match each real peak mass from the associated NIST20 spectra against this mass set and compute the fraction of total peak intensity covered, separately for d = 3 and d = 4, writing results to coverage_summary.csv. Rationale: allowing hydrogen drift within tolerance captures rearrangements not modeled by pure bond-breaking, and depth 3-4 balances search space size against coverage of real peak intensity.

## Related tools

- **RDKit** (Parse molecular graphs and strip hydrogen atoms to obtain heavy atom skeleton H(G) for fragment enumeration)
- **Cython** (Implementation language for the recursive bond-breaking algorithm (Algorithm 1) that generates fragment nodes and edges of the DAG)
- **Python** (Scripting environment for mass enumeration, peak matching, and writing coverage_summary.csv)

## Examples

```
python scripts/compute_coverage.py --nist20-spectra nist20_msms.pkl --dag-depth 3 4 --hydrogen-tolerance 4 --output coverage_summary.csv
```

## Evaluation signals

- Coverage fractions for d in {3,4} with j=4 should be high, confirming the setting captures most of the total peak intensity of real NIST20 MS/MS data
- Every mass in M̂(G, d, 4) must correspond to a monoisotopic formula derived from a DAG node with hydrogen count within h_n ± 4
- The heavy atom skeleton H(G) must be the largest connected subgraph of G with all hydrogens removed
- Per-(d,j) coverage fractions in coverage_summary.csv should match the paper's reported Table 4 finding

## Limitations

- The fragmentation algorithm depends on recursive edge-removal operations, which could be parallelized to accelerate runtime
- The algorithm fails to account for some fragments, as evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model

## Evidence

- [methods] Calculating coverage statistic: [evidence span withheld — anchor status: not_found]
- [methods] Heavy atom skeleton: [evidence span withheld — anchor status: not_found]
- [methods] Recursive bond-breaking algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] Fragmentation algorithm limitation: [evidence span withheld — anchor status: not_found]
- [methods] RDKit role: [evidence span withheld — anchor status: not_found]
