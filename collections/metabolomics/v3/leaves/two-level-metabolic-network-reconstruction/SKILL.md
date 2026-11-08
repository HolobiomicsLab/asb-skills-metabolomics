---
name: two-level-metabolic-network-reconstruction
description: Use when the goal is to reconstruct the metabolisms of one or two KEGG
  organisms from KGML pathway files and compare them at structural (graph) and functional
  (reaction-set) levels using MetNet's similarity indexes.
when_to_use_negative:
- When the input is raw genomic data rather than KEGG organism codes — MetNet does
  not currently support input of genomic data and would need significant extension
- When reconstruction from a non-KEGG source is required — the method is based uniquely
  on the KEGG database, which limits the represented information
- When pathway comparison results must be integrated across databases like BioCyc
  or BioModels rather than KEGG reference pathways
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0780
tools:
- name: MetNet
  role: Java tool that automatically reconstructs the two-level metabolic networks
    of two KEGG organisms and compares them quantitatively and visually
  repo: https://github.com/simeoni-biolab/MetNet
- name: Java
  role: Compiler/runtime used to build and execute MetNet from source with the bundled
    library jars
- name: gs-algo-1.3.jar
  role: Classpath library needed to compile MetNet's graph-handling source code
- name: gs-core-1.3.jar
  role: Classpath library needed to compile MetNet's graph-handling source code
- name: gs-ui-1.3.jar
  role: Classpath library needed to compile MetNet's source code
- name: guava-19.0.jar
  role: Classpath library needed to compile MetNet's source code
- name: KEGG database
  role: Source of KGML pathway files and organism list from which compounds and reactions
    are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:4f1a15ec095fedd712374949caacde73456c359052d2fab25d0b829bcb61547a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  - gs-algo-1.3.jar
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# two-level-metabolic-network-reconstruction

## Summary

Automatically reconstruct the metabolic networks of two KEGG organisms using MetNet's two-level representation — a structural graph of pathways and a functional set of reactions per pathway — so that the metabolisms can be compared quantitatively and visually with local and global similarity indexes.

## When to use

Use when the goal is to reconstruct the metabolisms of one or two KEGG organisms from KGML pathway files and compare them at structural (graph) and functional (reaction-set) levels using MetNet's similarity indexes.

## When NOT to use

- When the input is raw genomic data rather than KEGG organism codes — MetNet does not currently support input of genomic data and would need significant extension
- When reconstruction from a non-KEGG source is required — the method is based uniquely on the KEGG database, which limits the represented information
- When pathway comparison results must be integrated across databases like BioCyc or BioModels rather than KEGG reference pathways

## Inputs

- KEGG organism codes for two organisms (e.g. 'api' Acyrthosiphon pisum, 'bap' Buchnera aphidicola 5A)
- KGML pathway files automatically downloaded from the KEGG database via its public APIs
- Configuration files organismList.txt (KEGG organism list) and pathwayList.txt (pathways to consider)
- Choice of comparison method: 'set' or 'multiset'

## Outputs

- Two-level metabolic networks (structural graph + functional reaction sets) for both organisms
- Local similarity indexes per pathway (SimPi, SimSi)
- Global similarity indexes Psim, PsimW, Ssim, Csim in [0, 1]
- Quantitative and visual comparison of the two networks

## How to apply

Launch MetNet with the two KEGG organism codes and the comparison method, e.g. 'java -jar MetNet.jar api bap set' (use 'multiset' to account for repeated reactions). MetNet downloads the KGML files of each pathway of both organisms through the public KEGG APIs and parses them to extract compounds and reactions, building the two-level network: pathways as nodes with relations as edges (structural level) and each pathway's reaction content (functional level). Ensure the configuration files organismList.txt and pathwayList.txt are in the tool's directory; pathwayList.txt controls which reference pathways enter the comparison. MetNet then computes local indexes (SimPi Jaccard on reaction sets with Case 1/2/3 handling of missing, physical-only and reaction pathways; SimSi edge overlap at the structural level) and global indexes Psim, PsimW, Ssim and Csim. Compile from source if needed with javac against the bundled gs-algo-1.3.jar, gs-core-1.3.jar, gs-ui-1.3.jar and guava-19.0.jar libraries.

## Related tools

- **MetNet** (Java tool that automatically reconstructs the two-level metabolic networks of two KEGG organisms and compares them quantitatively and visually) — https://github.com/simeoni-biolab/MetNet
- **Java** (Compiler/runtime used to build and execute MetNet from source with the bundled library jars)
- **gs-algo-1.3.jar** (Classpath library needed to compile MetNet's graph-handling source code)
- **gs-core-1.3.jar** (Classpath library needed to compile MetNet's graph-handling source code)
- **gs-ui-1.3.jar** (Classpath library needed to compile MetNet's source code)
- **guava-19.0.jar** (Classpath library needed to compile MetNet's source code)
- **KEGG database** (Source of KGML pathway files and organism list from which compounds and reactions are extracted)

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- All four global indexes (Psim, PsimW, Ssim, Csim) fall within [0, 1]
- For a biologically distinct pair such as api vs bap, all global indexes are low (paper reports 5.02%–14.73%), confirming largely different metabolisms
- 'set' and 'multiset' modes both run to completion and yield comparable index ranges for the same organism pair
- KGML download succeeds for every pathway listed in pathwayList.txt for both organisms
- organismList.txt and pathwayList.txt are present in the tool's directory before execution

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; it would have to be significantly extended to implement reconstruction from genomes
- Obtaining a metabolism generally requires human intervention because data repositories are incomplete, heterogeneous and incoherent — hence the choice to rely on KEGG's standardized reference pathways

## Evidence

- [abstract] MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually.: "MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [methods] download the KGML files of each pathway of the organism through the public KEGG's APIs: [evidence span withheld — anchor status: not_found]
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
- [discussion] Our metabolic reconstruction technique is totally automatic and is based uniquely on the KEGG database: "Our metabolic reconstruction technique is totally automatic and is based uniquely on the KEGG database"
- [discussion] the tool MetNet has to be significantly extended to allow the input of genomic data and implement the reconstruction of the corresponding metabolic network: "the tool MetNet has to be significantly extended to allow the input of genomic data and implement the reconstruction of the corresponding metabolic network"
