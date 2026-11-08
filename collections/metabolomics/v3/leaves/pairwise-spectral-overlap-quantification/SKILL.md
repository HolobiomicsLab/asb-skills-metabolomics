---
name: pairwise-spectral-overlap-quantification
description: Use when you need pairwise similarity/distance between complete MS/MS
  datasets (e.g. for phylogenetics, QC, or species identification) without peptide
  identification or spectral libraries.
when_to_use_negative:
- You have identified peptides and want standard protein quantitation or database-search
  results — compareMS2 compares sets of tandem mass spectra directly rather than to
  predicted spectra of specific peptides
- You only have a single dataset (no comparison target) — D(A,A) is always zero, so
  pairwise spectral overlap cannot be quantified
- You need a distance metric that tolerates loose definitions of spectrum identity
  — the Jaccard similarity is very sensitive to the precise definition of when two
  spectra are considered identical
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: 'Primary tool: runs the pairwise spectral similarity comparisons (eq 1 with
    m/z windows and similarity thresholds) and assembles the symmetric distance matrix'
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Alternative downstream tool in which the correct phylogenetic tree could be
    reconstructed from the compareMS2 distance matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:66a0be7fc479ef8f94f2e1c1bc388679cd998fcdd24d340fb2f5c142a1c81f00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pairwise-spectral-overlap-quantification

## Summary

Computes the distance between sets of tandem mass spectra (e.g. LC-MS/MS serum datasets) using compareMS2 2.0's fully symmetric spectral-similarity metric, assembling unique pairwise comparisons into a distance matrix without redundant recalculation.

## When to use

Use when you need pairwise similarity/distance between complete MS/MS datasets (e.g. for phylogenetics, QC, or species identification) without peptide identification or spectral libraries.

## When NOT to use

- You have identified peptides and want standard protein quantitation or database-search results — compareMS2 compares sets of tandem mass spectra directly rather than to predicted spectra of specific peptides
- You only have a single dataset (no comparison target) — D(A,A) is always zero, so pairwise spectral overlap cannot be quantified
- You need a distance metric that tolerates loose definitions of spectrum identity — the Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical

## Inputs

- Tandem mass spectrometry datasets (MGF files of LC-MS/MS spectra), e.g. the five serum datasets from ProteomeXchange PXD034932
- compareMS2 2.0 execution environment (NodeJS 22, corepack/yarn)
- Optional: sample-to-species mapping file (spectra_to_species.txt, TAB-separated filename/species pairs)

## Outputs

- Pairwise distance values for each unique dataset pair
- Symmetric global distance matrix (e.g. distance_matrix.tsv; 5x5 for five datasets, 10 unique entries)
- Optional UPGMA phylogenetic tree rendered by the compareMS2 GUI

## How to apply

Obtain the tandem mass spectra datasets (e.g. MGF files; the five PXD034932 serum datasets each contain 42,629–47,626 spectra) and set up compareMS2 2.0 (git clone https://github.com/524D/compareMS2.git, corepack enable, yarn). Run compareMS2 on all unique dataset pairs using default parameters, applying the spectral similarity of eq 1 with its m/z windows and spectral similarity thresholds to quantify overlap between each pair of spectrum sets. Exploit the fully symmetric distance metric of compareMS2 2.0: since D(A,B) = D(B,A) and D(A,A) = 0, an n-dataset set requires only n(n-1)/2 unique comparisons — 10 for five datasets — which took about 40 min total on an Intel Xeon W-2135 at 3.70 GHz. Assemble the pairwise distances into a symmetric distance matrix (e.g. distance_matrix.tsv); the GUI can then draw a UPGMA phylogenetic tree as each row completes. Note that Jaccard-based similarity is highly sensitive to the precise definition of when two spectra are considered identical, so keep parameters consistent across all comparisons.

## Related tools

- **compareMS2** (Primary tool: runs the pairwise spectral similarity comparisons (eq 1 with m/z windows and similarity thresholds) and assembles the symmetric distance matrix) — https://github.com/524D/compareMS2.git
- **MEGA11** (Alternative downstream tool in which the correct phylogenetic tree could be reconstructed from the compareMS2 distance matrix)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn
```

## Evaluation signals

- Distance matrix contains exactly n(n-1)/2 unique values (10 for five datasets) with all 10 pairwise comparisons accounted for
- Matrix is perfectly symmetric: D(A,B) = D(B,A) for every pair, and the diagonal is zero
- Datasets with identical spectral content give similarity 1 and distance 0
- Runtime is in the expected order — 10 comparisons of ~42,629–47,626-spectrum datasets took ~40 min on an Intel Xeon W-2135 at 3.70 GHz with default parameters
- Reconstructed phylogenetic tree from the matrix matches that of a reference implementation (compareMS2 and MEGA11 reproduced the correct tree in the paper)

## Limitations

- Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical, making results parameter-dependent
- Datasets with no similar spectra tend to distance +infinity as set sizes grow; extremes of none or all spectra being similar are unlikely in practice
- The metric reflects not only peptide sequences but also protein abundance/coverage, which affects comparisons across tissues or cell lines
- Compute cost scales quadratically: 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours

## Evidence

- [results] 10 pairwise comparisons in compareMS2 took 40 min with default parameters: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [readme] symmetric metric rationale: [evidence span withheld — anchor status: external_doc]
- [results] equation 1 usage: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [results] parameter estimation: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [results] Jaccard sensitivity caveat: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"
- [methods] repository installation: "git clone https://github.com/524D/compareMS2.git
cd compareMS2
corepack enable
yarn"
- [readme] real-time matrix and tree updates: [evidence span withheld — anchor status: external_doc]
