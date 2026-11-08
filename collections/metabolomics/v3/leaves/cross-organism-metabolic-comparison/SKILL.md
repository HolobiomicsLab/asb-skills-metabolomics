---
name: cross-organism-metabolic-comparison
description: Use when the goal is to compare metabolisms of two or more KEGG organisms
  via similarity indexes (set/multiset reaction sets) or to check whether index-based
  groupings reproduce known evolutionary relationships.
when_to_use_negative:
- When the input is raw genomic/experimental data rather than KEGG organisms — MetNet
  does not yet accept genomic data and reconstructs metabolism only from the KEGG
  database
- When a single organism must be reconstructed without any comparison — MetNet is
  built around pairwise (and derived multi-pair) comparison of two or more KEGG organisms
- When metabolism must be reconstructed from non-KEGG sources (e.g. BioCyc, BioModels)
  or from newly annotated genomes, since reconstruction is based uniquely on KEGG's
  reference pathways
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MetNet
  role: Reconstructs KEGG organism metabolic networks, computes the Ssim/Psim/PsimW/Csim
    similarity indexes, and visualizes/renders the comparison results
  repo: github.com/simeoni-biolab/MetNet
- name: Java
  role: Compiler/runtime used to build and launch the MetNet jar from the command
    line
- name: KEGG database
  role: Source of KGML pathway files and organism codes from which the networks are
    reconstructed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:3fb5d834029c9a69101be1d14a18f40d9105cd335dfc7274357f4ad2f0eb426b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-organism-metabolic-comparison

## Summary

Reconstruct the metabolic networks of KEGG organisms with MetNet and compare them quantitatively and visually using two-level similarity indexes (structural Ssim, pathway Psim/PsimW, combined Csim), optionally extending pairwise comparisons to a whole organism set via distance-matrix clustering.

## When to use

Use when the goal is to compare metabolisms of two or more KEGG organisms via similarity indexes (set/multiset reaction sets) or to check whether index-based groupings reproduce known evolutionary relationships.

## When NOT to use

- When the input is raw genomic/experimental data rather than KEGG organisms — MetNet does not yet accept genomic data and reconstructs metabolism only from the KEGG database
- When a single organism must be reconstructed without any comparison — MetNet is built around pairwise (and derived multi-pair) comparison of two or more KEGG organisms
- When metabolism must be reconstructed from non-KEGG sources (e.g. BioCyc, BioModels) or from newly annotated genomes, since reconstruction is based uniquely on KEGG's reference pathways

## Inputs

- KEGG organism codes (e.g. sce, zro, cal, afm, abp, fgr, tpf, tre) selected from the KEGG organism list
- KGML pathway files of each organism retrieved via KEGG public APIs
- MetNet configuration files organismList.txt (KEGG organisms) and pathwayList.txt (reference pathways to compare)
- Comparison method parameter: 'set' or 'multiset'

## Outputs

- Two-level metabolic network reconstructions (structural graph plus reaction content per pathway) for each organism
- Per-pair similarity index values: Ssim, Psim, PsimW, and Csim
- For multi-organism studies: a symmetric similarity-index matrix, a distance matrix (1 - Csim), and a complete-linkage dendrogram grouping the organisms

## How to apply

Select the organisms of interest by KEGG code, ensuring the configuration files organismList.txt and pathwayList.txt are present in the tool directory (pathwayList.txt defines which KEGG reference pathways enter the comparison). MetNet downloads the KGML files of each pathway through KEGG's public APIs, parses compounds and reactions, and builds the two-level representation: pathways as nodes/edges (structural level) and pathway reaction content (functional level). Choose the comparison method 'set' or 'multiset' — pathways are treated as sets of reactions when 'set' is chosen — then compute the structural (Ssim), pathway (Psim), weighted pathway (PsimW), and combined (Csim) indexes for each organism pair. For an N-organism study, run all N(N-1)/2 pairwise comparisons, build a distance matrix as distance = 1 - Csim, and apply complete-linkage hierarchical clustering; render the dendrogram and read the top-level split against known phylogeny. In the published validation, the eight Fungi (sce, zro, cal, afm, abp, fgr, tpf, tre) separated at the top level into Yeasts vs Molds with every index, including Ssim.

## Related tools

- **MetNet** (Reconstructs KEGG organism metabolic networks, computes the Ssim/Psim/PsimW/Csim similarity indexes, and visualizes/renders the comparison results) — github.com/simeoni-biolab/MetNet
- **Java** (Compiler/runtime used to build and launch the MetNet jar from the command line)
- **KEGG database** (Source of KGML pathway files and organism codes from which the networks are reconstructed)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- The distance matrix is symmetric with N(N-1)/2 distinct pairwise Csim values (28 pairs for eight organisms)
- Complete-linkage clustering of distance = 1 - Csim reproduces the expected top-level split; in the Fungi validation the dendrogram's first split isolates the two Saccharomyces yeasts (sce, zro) from the six molds
- The same evolutionary grouping is obtained with all defined indexes, including the structure similarity index Ssim, which additionally separates the Saccharomycetes class
- Both configuration files organismList.txt and pathwayList.txt are present and the requested organism codes resolve against the KEGG organism list
- Compiled jar executes with the documented classpath (gs-algo-1.3.jar, gs-core-1.3.jar, gs-ui-1.3.jar, guava-19.0.jar, poi-3.9.jar, sax2r2.jar)

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; significant extension is required to reconstruct networks from experimental genomic data
- The comparison result depends on the chosen comparison method ('set' vs 'multiset') and on the pathway subset listed in pathwayList.txt

## Evidence

- [abstract] MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually.: "MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [methods] it is necessary to download the KGML files of each pathway of the organism through the public KEGG's APIs: [evidence span withheld — anchor status: not_found]
- [methods] to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [readme] To run MetNet as a command line tool, the following three parameters must be specified when starting the execution: "To run MetNet as a command line tool, the following three parameters must be specified when starting the execution"
- [discussion] is based uniquely on the KEGG database, which can be a limit wrt. the represented information: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
