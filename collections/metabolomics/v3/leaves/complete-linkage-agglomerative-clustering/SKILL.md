---
name: complete-linkage-agglomerative-clustering
description: Use when you have pairwise similarity indexes (Ssim, Psim, PsimW, Csim)
  for >2 KEGG organisms and want to test whether hierarchical clustering of the resulting
  distance matrix recovers known phylogenetic groupings.
when_to_use_negative:
- When only a two-organism comparison is needed — MetNet natively reconstructs and
  compares a single pair, and no clustering step is required
- When the organisms of interest are not in the KEGG database — MetNet's reconstruction
  'is based uniquely on the KEGG database, which can be a limit wrt. the represented
  information'
- When only local, pathway-by-pathway comparison is of interest rather than global
  organism grouping
edam_operation: http://edamontology.org/operation_3613
tools:
- name: MetNet
  role: Reconstructs each organism's metabolic network from KEGG KGML files and computes
    the pairwise Ssim/Psim/PsimW/Csim similarity indexes feeding the distance matrix
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of organism KGML pathway files retrieved via KEGG's public APIs for
    network reconstruction
- name: Java
  role: Compiler/runtime used to build and execute the MetNet jar (javac with gs-*,
    guava, poi, sax2r2 library jars)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:6feee6bdea01309121f9f4c1e206e14215259a54b8040439aacf3918c8faedd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: http://edamontology.org/operation_3613
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# complete-linkage-agglomerative-clustering

## Summary

Builds a distance matrix from pairwise metabolic-network similarity indexes computed by MetNet (distance = 1 - Csim) and applies complete-linkage agglomerative hierarchical clustering to a set of KEGG organisms, then reads the top-level dendrogram split to check whether groupings agree with known evolutionary relationships.

## When to use

Use when you have pairwise similarity indexes (Ssim, Psim, PsimW, Csim) for >2 KEGG organisms and want to test whether hierarchical clustering of the resulting distance matrix recovers known phylogenetic groupings.

## When NOT to use

- When only a two-organism comparison is needed — MetNet natively reconstructs and compares a single pair, and no clustering step is required
- When the organisms of interest are not in the KEGG database — MetNet's reconstruction 'is based uniquely on the KEGG database, which can be a limit wrt. the represented information'
- When only local, pathway-by-pathway comparison is of interest rather than global organism grouping

## Inputs

- Metabolic networks of N KEGG organisms reconstructed by MetNet from KGML files (compounds and reactions per pathway)
- Pairwise similarity-index matrix (Ssim, Psim, PsimW, Csim) computed with the 'set' comparison method for all N(N-1)/2 organism pairs
- List of KEGG organism codes (e.g. sce, zro, cal, afm, abp, fgr, tpf, tre)

## Outputs

- Symmetric distance matrix (distance = 1 - Csim) over the organism set
- Complete-linkage hierarchical clustering dendrogram
- Top-level organism groupings to compare against known evolutionary relationships (e.g. Yeasts vs. Molds)

## How to apply

First reconstruct each organism's metabolic network with MetNet by downloading the KGML files of each pathway through KEGG's public APIs and parsing each KGML file to extract the compounds and reactions of each pathway, using pathways represented as sets of reactions (comparison method 'set'). Compute all pairwise similarity indexes — structural (Ssim), pathway (Psim), weighted pathway (PsimW) and combined (Csim) — so that N organisms yield N(N-1)/2 comparisons (28 for the paper's eight Fungi: sce, zro, cal, afm, abp, fgr, tpf, tre). Convert the combined index into a distance matrix via distance = 1 - Csim and run complete-linkage agglomerative hierarchical clustering on the organisms. Render the dendrogram and check the top-level split against expected evolutionary relationships; in the paper's validation, the first split must place the two Saccharomyces yeasts (sce, zro) in a branch disjoint from the six mold fungi (cal, afm, abp, fgr, tpf, tre). Repeating the clustering with each index separately (including Ssim, which can differentiate the Saccharomycetes class) verifies the grouping is robust across the defined indexes.

## Related tools

- **MetNet** (Reconstructs each organism's metabolic network from KEGG KGML files and computes the pairwise Ssim/Psim/PsimW/Csim similarity indexes feeding the distance matrix) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of organism KGML pathway files retrieved via KEGG's public APIs for network reconstruction)
- **Java** (Compiler/runtime used to build and execute the MetNet jar (javac with gs-*, guava, poi, sax2r2 library jars))

## Examples

```
java -jar MetNet.jar sce cal set
```

## Evaluation signals

- Number of pairwise comparisons equals N(N-1)/2 (28 for the paper's eight Fungi organisms)
- Distance matrix is symmetric with zero diagonal and entries in [0,1] since distance = 1 - Csim
- The complete-linkage dendrogram's first (top-level) split must separate the two Saccharomyces yeasts (sce, zro) from the six mold fungi (cal, afm, abp, fgr, tpf, tre)
- The same clustering must be obtained with all defined indexes, including the structure similarity index Ssim, which should additionally differentiate the Saccharomycetes class
- Configuration files organismList.txt and pathwayList.txt must be present in the tool's directory before execution

## Limitations

- Reconstruction is limited to the KEGG database, which constrains the represented information
- MetNet compares organisms pairwise, so a set-level clustering requires an external loop over all organism pairs
- MetNet does not currently support input of genomic data; extending it would require a reconstruction approach compliant with the modular organization of KEGG pathways
- Classpath separators for the jar libraries differ between Windows (semicolon) and Linux (colon), a common compile-time pitfall

## Evidence

- [methods] KGML download step: [evidence span withheld — anchor status: not_found]
- [methods] comparison method option: "3. comparison method: "set" or "multiset""
- [intro] evolutionary grouping motivation: "to explore whether the groups suggested by our similarity indexes agree with those established by well known evolutionary relationships."
- [abstract] global vs local comparison: "They allow for both a local comparison, pathway by pathway, and a global comparison of the entire metabolism."
- [readme] CLI invocation pattern: "will start the execution for the organisms "hsa" (Homo Sapiens) and "prt" (Pan troglodytes) with the comparison method "set"."
- [results] KEGG organism list download: "the initial view allows the user to download the latest list of KEGG organisms: if the download is requested, the local list is substituted with the new one."
