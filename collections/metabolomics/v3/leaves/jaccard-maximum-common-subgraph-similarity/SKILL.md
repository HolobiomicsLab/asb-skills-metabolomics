---
name: jaccard-maximum-common-subgraph-similarity
description: Use when lipid species names must be compared by molecular structure
  rather than exact name matching, e.g. to build structural distance matrices across
  heterogeneous lipidomes from csv/xlsx or mzTab-M lipidomics tables.
when_to_use_negative:
- The analysis only requires comparing lipidomes by exact matches of lipid species
  names/identifiers rather than structural similarity
- Quantitative abundance effects dominate the question — this structural distance
  alone is heavily influenced by low abundant lipid species unless abundances are
  later added and normalized to the variance of the first PC
- Input lipid names cannot be parsed/standardized by Goslin (unsupported lipid name
  dialects)
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: LipidSpace
  role: Standalone C++/Qt application that executes the MCS+Jaccard pairwise lipid
    structural comparison to build the distance matrix
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names into structures before distance computation
  repo: https://github.com/lifs-tools/cppgoslin
- name: OpenBLAS
  role: Fast matrix operations on the resulting n x n distance matrix during downstream
    PCA
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:fefe13d0d16df3ac2a79ce9575259cde8425464c5046e301a267948449dbb9de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LipidSpace
  - OpenBLAS
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# jaccard-maximum-common-subgraph-similarity

## Summary

Computes pairwise structural similarity between lipid species by combining the maximum common subgraph (LaWeCSE) approach with the Jaccard index on parsed molecular structures, producing the n x n distance matrix that underpins LipidSpace's structural space model.

## When to use

Use when lipid species names must be compared by molecular structure rather than exact name matching, e.g. to build structural distance matrices across heterogeneous lipidomes from csv/xlsx or mzTab-M lipidomics tables.

## When NOT to use

- The analysis only requires comparing lipidomes by exact matches of lipid species names/identifiers rather than structural similarity
- Quantitative abundance effects dominate the question — this structural distance alone is heavily influenced by low abundant lipid species unless abundances are later added and normalized to the variance of the first PC
- Input lipid names cannot be parsed/standardized by Goslin (unsupported lipid name dialects)

## Inputs

- Standardized lipid species structures parsed from lipid names via Goslin (csv/xlsx flat or pivot tables, or mzTab-M files)
- Optional mode flags: ignore lipid sn-positions (all-combination FA comparison) and unbound distance metric

## Outputs

- Pairwise structural similarity/distance values for all pairs of distinct lipid species
- n x n distance matrix used as input for PCA of the global lipid space model

## How to apply

First parse and standardize lipid names with the Goslin C++ library so that every species resolves to a structural graph. Then, for each pair of distinct lipid species, apply the maximum common subgraph approach (LaWeCSE) to identify the largest shared substructure and combine it with the Jaccard index to yield a bounded pairwise similarity. By default, fatty acyl chains are compared positionally (first FA of one lipid vs first FA of the other); when sn-positions are unspecified (e.g. PC 18:0_16:1), optionally compare all FA combinations and take the lowest distance for higher accuracy at reduced performance. Collect all pairwise values into an n x n distance matrix, which is then fed to PCA (via the Lanczos method with OpenBLAS) to build the global lipid space model. Distances range from 0 (identical lipids) to 1 with the default bound metric; an unbound mode (0 to infinity) can be activated but may compress the visibility of lipid spaces.

## Related tools

- **LipidSpace** (Standalone C++/Qt application that executes the MCS+Jaccard pairwise lipid structural comparison to build the distance matrix) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid names into structures before distance computation) — https://github.com/lifs-tools/cppgoslin
- **OpenBLAS** (Fast matrix operations on the resulting n x n distance matrix during downstream PCA) — https://www.openblas.net/

## Examples

```
LipidSpace.exe  # File → Import example dataset, then compute pairwise MCS+Jaccard lipid distances for the structural space model
```

## Evaluation signals

- The resulting distance matrix is square and symmetric of size n x n for n distinct lipid species
- With the default bound metric, every pairwise distance falls in [0, 1], where 0 indicates two identical lipids
- Lipid pairs differing only by unsaturation detail (e.g. double bond positions) receive near-identical scores consistent with the 'Correctness of Double Bond Comparisons' checks
- Activating 'Ignore lipid sn-positions' on sn-unspecified species (e.g. PC 18:0_16:1) yields the lowest distance over all FA comparison combinations and decreases performance
- The downstream PCA on the matrix resolves the default seven principal components and projects lipidomes onto PC1 (x) and PC2 (y)

## Limitations

- All-combination fatty acyl comparison (for sn-unspecified names) is more accurate but decreases performance
- The unbound distance metric gives more accurate results but reduces visibility of the lipid spaces because distances may become very large
- LipidSpace supports only a bound metric by default; unbound mode must be explicitly activated via Analysis → Unbound distance metric
- Analyses cannot currently be saved or reloaded, so the distance computation must be rerun per session

## Evidence

- [intro] MCS+Jaccard pairwise similarity: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] LaWeCSE distance computation: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [methods] PCA on distance matrix: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [readme] Sn-position comparison mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
- [readme] Bound distance metric range: "the distance is a value that ranges between 0 (both lipids are identical) and 1. However, other distance measures suggest an unbound distance ranging from 0 to infinity"
