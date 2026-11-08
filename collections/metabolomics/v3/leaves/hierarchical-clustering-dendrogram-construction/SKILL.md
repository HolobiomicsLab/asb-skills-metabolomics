---
name: hierarchical-clustering-dendrogram-construction
description: Use when you have a full pairwise similarity-index matrix (Ssim, Psim,
  PsimW, Csim) for a set of organisms and want to test whether index-based groupings
  reproduce known evolutionary or phylogenetic structure.
when_to_use_negative:
- When you need to reconstruct a metabolic network from raw genomic/experimental data
  — MetNet reconstruction is based uniquely on the KEGG database and does not currently
  accept genomic data as input
- When comparing only a single pair of organisms — a two-organism MetNet comparison
  yields no distance matrix and no dendrogram to interpret
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetNet
  role: Reconstructs KEGG organism metabolic networks and computes the pairwise Ssim/Psim/PsimW/Csim
    similarity indexes feeding the distance matrix
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of KGML pathway files and organism codes used for network reconstruction
- name: Java
  role: Compilation and execution environment for the MetNet jar used to generate
    similarity indexes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:a53c9ca662448c7639917b993a77d59e6da21f7e67f0249ff161e225dae71cb9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# hierarchical-clustering-dendrogram-construction

## Summary

Build a distance matrix from pairwise MetNet similarity indexes (distance = 1 - Csim) and apply complete-linkage hierarchical clustering to a set of KEGG organisms, rendering a dendrogram whose top-level splits can be checked against known evolutionary relationships.

## When to use

Use when you have a full pairwise similarity-index matrix (Ssim, Psim, PsimW, Csim) for a set of organisms and want to test whether index-based groupings reproduce known evolutionary or phylogenetic structure.

## When NOT to use

- When you need to reconstruct a metabolic network from raw genomic/experimental data — MetNet reconstruction is based uniquely on the KEGG database and does not currently accept genomic data as input
- When comparing only a single pair of organisms — a two-organism MetNet comparison yields no distance matrix and no dendrogram to interpret

## Inputs

- List of KEGG organism codes (e.g. sce, zro, cal, afm, abp, fgr, tpf, tre)
- KGML pathway files for each organism retrieved from KEGG
- MetNet configuration files organismList.txt and pathwayList.txt
- Comparison method choice: 'set' or 'multiset'

## Outputs

- Pairwise similarity-index matrix for each index (Ssim, Psim, PsimW, Csim)
- Distance matrix (1 - Csim) over the organism set
- Complete-linkage hierarchical clustering dendrogram of the organisms
- Validation check: top-level branch assignment (e.g. Yeasts branch disjoint from Molds branch)

## How to apply

First reconstruct the metabolic networks of all N KEGG organisms with MetNet by downloading each organism's KGML files via KEGG's public APIs and parsing compounds and reactions per pathway, using the 'set' comparison method so pathways are represented as sets of reactions. Compute all N(N-1)/2 pairwise indexes — structural (Ssim), pathway (Psim), weighted pathway (PsimW) and combined (Csim) — and build the distance matrix as distance = 1 - Csim (repeat with the other indexes to check robustness). Apply complete-linkage hierarchical clustering on the distance matrix, where complete linkage is chosen because it separates very similar organisms (e.g. Yeasts vs Molds) at the top level of the tree. Render the dendrogram and validate its top-level split against the expected evolutionary grouping; in the paper's Fungi experiment (sce, zro, cal, afm, abp, fgr, tpf, tre), the first split must isolate the two Saccharomyces yeasts from the six mold fungi, and the same clustering should be obtained with all defined indexes, including Ssim.

## Related tools

- **MetNet** (Reconstructs KEGG organism metabolic networks and computes the pairwise Ssim/Psim/PsimW/Csim similarity indexes feeding the distance matrix) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of KGML pathway files and organism codes used for network reconstruction)
- **Java** (Compilation and execution environment for the MetNet jar used to generate similarity indexes)

## Examples

```
java -jar MetNet.jar sce cal set
```

## Evaluation signals

- The distance matrix must be complete for all 28 pairwise comparisons among the eight Fungi organisms
- The complete-linkage dendrogram's first (top-level) split must place the two Saccharomyces yeasts (sce, zro) in a branch disjoint from the six mold fungi (cal, afm, abp, fgr, tpf, tre)
- The same top-level grouping must be reproduced when clustering on each individual index (Psim, PsimW, Ssim, Csim), not only Csim
- The structure similarity index should additionally separate the Saccharomycetes class within the yeast branch
- Pathway similarity must be computed with the same comparison method ('set') across all pairs for the matrix to be consistent

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; it would need to be significantly extended to reconstruct networks from newly annotated genomes
- Results depend on the pathway scope set in pathwayList.txt — the comparison covers only the KEGG reference pathways listed in the configuration file

## Evidence

- [abstract] Similarity measures are introduced for both levels, enabling local and global comparison: "we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [intro] Objective: check whether similarity-index groupings agree with known evolutionary relationships: "to explore whether the groups suggested by our similarity indexes agree with those established by well known evolutionary relationships."
- [methods] KGML retrieval step feeding network reconstruction: [evidence span withheld — anchor status: not_found]
- [methods] Comparison method must be specified as set or multiset: "3. comparison method: "set" or "multiset""
- [readme] CLI invocation form with two KEGG codes and comparison method: "will start the execution for the organisms "hsa" (Homo Sapiens) and "prt" (Pan troglodytes) with the comparison method "set"."
- [results] Clustering relies on KEGG reference pathways listed in the configuration file: "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
