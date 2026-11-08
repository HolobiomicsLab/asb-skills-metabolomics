---
name: metabolic-pathway-reconstruction
description: Use when you have the KEGG code of one or more organisms and a pathway
  list (organismList.txt / pathwayList.txt) and need per-pathway reaction sets for
  the functional level of the network.
when_to_use_negative:
- When you need to reconstruct the metabolic network from newly annotated genomes
  or raw genomic data — MetNet does not accept genomic input; use AutoKEGGRec or the
  KBase Predictive Biology platform instead.
- When the source of pathway/reaction information is not the KEGG database — the reconstruction
  is uniquely based on KEGG, which constrains the represented information.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0153
tools:
- name: MetNet
  role: 'Java tool that performs the whole pipeline: selects organisms, retrieves
    KGML files from KEGG, parses them for compounds and reactions, builds the functional-level
    reaction sets and runs the comparison'
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML reference pathways whose compounds and reactions are parsed
    to build the functional-level reaction sets
- name: guava-19.0.jar
  role: Java library on the classpath when compiling and running MetNet to parse KGML
    data and build reaction sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:fa396ed9787f94a8f9bddf7d092b4a51922362b0e10f62f9b5ccf140a7a17a22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolic-pathway-reconstruction

## Summary

Automatic reconstruction of the functional (reactions-based) level of a metabolic network for a KEGG organism: retrieve each reference pathway's KGML file via KEGG's public APIs and parse it to extract the pathway's compounds and reactions.

## When to use

Use when you have the KEGG code of one or more organisms and a pathway list (organismList.txt / pathwayList.txt) and need per-pathway reaction sets for the functional level of the network.

## When NOT to use

- When you need to reconstruct the metabolic network from newly annotated genomes or raw genomic data — MetNet does not accept genomic input; use AutoKEGGRec or the KBase Predictive Biology platform instead.
- When the source of pathway/reaction information is not the KEGG database — the reconstruction is uniquely based on KEGG, which constrains the represented information.

## Inputs

- KEGG organism codes (e.g. 'hsa', 'ptr')
- pathwayList.txt — list of KEGG reference pathways to consider
- organismList.txt — list of KEGG organisms available
- comparison method: 'set' or 'multiset'
- KGML files retrieved from KEGG for each organism pathway

## Outputs

- reaction_sets_<organism>.csv — one row per organism pathway (pathway_id, reaction_id, multiplicity for multiset mode)
- combined reaction_sets.csv — columns organism, pathway_id, reaction_id (and multiplicity for multiset mode)

## How to apply

Read the organism KEGG codes (e.g. 'hsa', 'ptr') and the comparison method ('set' or 'multiset') from the configuration files organismList.txt and pathwayList.txt. For each reference pathway of each organism, download the KGML file through KEGG's public APIs and parse it to extract the compounds and reactions of that pathway — the information needed for the functional-level representation. Treat membrane/physical-mechanism pathways as empty reaction sets since their KGML files contain no reaction information. Build, per organism, a pathway-ID to reaction-set record (set or multiset of reactions according to the chosen comparison method) and write reaction_sets_<organism>.csv plus a combined reaction_sets.csv with columns organism, pathway_id, reaction_id (and multiplicity for multiset mode). Validate that every pathway in the organism list yields exactly one reaction-set row and that reaction IDs match the corresponding KEGG pathway map.

## Related tools

- **MetNet** (Java tool that performs the whole pipeline: selects organisms, retrieves KGML files from KEGG, parses them for compounds and reactions, builds the functional-level reaction sets and runs the comparison) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML reference pathways whose compounds and reactions are parsed to build the functional-level reaction sets)
- **guava-19.0.jar** (Java library on the classpath when compiling and running MetNet to parse KGML data and build reaction sets)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every pathway listed in pathwayList.txt for the organism yields exactly one reaction-set record (empty reaction sets are allowed for physical/membrane pathways).
- Reaction IDs in the output correspond to reactions present in the corresponding KEGG pathway map (KGML-derived).
- The output CSV has columns organism, pathway_id, reaction_id (plus multiplicity when the comparison method is 'multiset').
- Running the same organism and method twice produces identical reaction sets (deterministic retrieval from KEGG).
- The combined reaction_sets.csv contains one row set per organism code supplied to the tool.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- MetNet does not currently accept genomic data as input; extending it to reconstruct networks from newly annotated genomes requires significant extension of the tool.
- Membrane/physical-mechanism pathways yield empty reaction sets because their KGML files contain no reaction information.

## Evidence

- [methods] MetNet retrieves each organism's pathway KGML files via KEGG's public APIs and parses them to extract the compounds and reactions of each pathway: [evidence span withheld — anchor status: not_found]
- [methods] KGML parsing is the data-extraction step for the functional level: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [readme] MetNet makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG"
- [readme] Command-line parameters define organisms and comparison method: "the following three parameters must be specified when starting the execution: 1. KEGG code of the first organism to compare 2. KEGG code of the second organism to compare 3. comparison method: "set""
