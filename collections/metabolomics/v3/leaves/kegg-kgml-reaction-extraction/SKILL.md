---
name: kegg-kgml-reaction-extraction
description: 'Use when building the functional level of a KEGG-based metabolic network
  for an organism: you need, per reference pathway, the set or multiset of reactions
  (and compounds) that MetNet compares between two organisms.'
when_to_use_negative:
- When the goal is the structural (graph-based) level of the network, which is built
  from pathway relations rather than reaction content.
- When reconstructing metabolism from raw genomic/experimental data — MetNet is based
  uniquely on the KEGG database and does not currently support input of genomic data.
- When the target pathways are physical-mechanism (membrane) pathway maps, since their
  KGML files contain no reaction information and yield only empty reaction sets.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: MetNet
  role: Java tool that downloads KGML files via KEGG's public APIs and parses them
    to extract the compounds and reactions of each pathway for the functional level
    of the two-level representation
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML Markup Language files containing each pathway map's reactions
    and compounds
- name: Java
  role: Compiler/runtime used to build and run MetNet from the source files
- name: guava-19.0.jar
  role: Library on MetNet's compile classpath required when building the tool that
    performs the KGML parsing
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:4cd7d0908ef76f919ab09881e3b616e3147a43e37d4bf7e70fa8f33bb0ea069e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# kegg-kgml-reaction-extraction

## Summary

Retrieve and parse KEGG KGML files via KEGG's public APIs to extract the compounds and reactions of each reference pathway, building the functional-level (reactions-based) metabolic data for an organism as used by MetNet's two-level metabolic network representation.

## When to use

Use when building the functional level of a KEGG-based metabolic network for an organism: you need, per reference pathway, the set or multiset of reactions (and compounds) that MetNet compares between two organisms.

## When NOT to use

- When the goal is the structural (graph-based) level of the network, which is built from pathway relations rather than reaction content.
- When reconstructing metabolism from raw genomic/experimental data — MetNet is based uniquely on the KEGG database and does not currently support input of genomic data.
- When the target pathways are physical-mechanism (membrane) pathway maps, since their KGML files contain no reaction information and yield only empty reaction sets.

## Inputs

- KEGG organism code(s) (e.g. hsa, ptr, apis, bap) from organismList.txt
- pathwayList.txt — configuration file listing the KEGG reference pathways to consider
- KGML files of each pathway of the organism, downloaded through KEGG's public APIs
- Comparison method setting: 'set' or 'multiset'

## Outputs

- Per-reference-pathway reaction-set records (pathway ID -> set or multiset of reactions)
- reaction_sets_<organism>.csv tables (columns: organism, pathway_id, reaction_id; multiplicity in multiset mode)
- Combined reaction_sets.csv across organisms

## How to apply

Start from the organism's KEGG organism code (e.g. hsa, ptr, apis, bap) and the pathway catalog in pathwayList.txt, then download the KGML files of each pathway of the organism through KEGG's public APIs — this automatic data retrieval is the first step in reconstructing the metabolic network. Parse each KGML file to extract the relevant information for the functional-level representation, which is the compounds and the reactions of each pathway, and build a per-reference-pathway reaction-set record (pathway ID -> reactions). Apply the comparison method from MetNet's configuration — 'set' or 'multiset' — so pathway maps with repeated reactions become multisets when requested. Treat membrane/physical-mechanism pathways as empty reaction sets, since their KGML files contain no reaction information. Emitters write the per-organism tables (reaction_sets_<organism>.csv) with columns organism, pathway_id, reaction_id, including multiplicity when multiset mode is selected.

## Related tools

- **MetNet** (Java tool that downloads KGML files via KEGG's public APIs and parses them to extract the compounds and reactions of each pathway for the functional level of the two-level representation) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML Markup Language files containing each pathway map's reactions and compounds)
- **Java** (Compiler/runtime used to build and run MetNet from the source files)
- **guava-19.0.jar** (Library on MetNet's compile classpath required when building the tool that performs the KGML parsing)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Each pathway listed for the organism in pathwayList.txt yields exactly one reaction-set record; empty reaction sets are allowed for physical-mechanism pathways.
- Extracted reaction IDs match the reactions present in the corresponding pathway map's KGML file.
- Reaction-set records respect the configured comparison method: 'set' removes multiplicity, 'multiset' preserves it.
- Output CSVs (reaction_sets_<organism>.csv) contain the columns organism, pathway_id, reaction_id (plus multiplicity for multiset mode) and cover both compared organisms.
- Compile/run validation: MetNet builds with the documented classpath (gs-algo, gs-core, gs-ui, guava-19.0, poi-3.9, sax2r2 jars) and executes with configuration files organismList.txt and pathwayList.txt present in the tool's directory.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which constrains the represented information.
- KGML files for membrane/physical-mechanism pathways contain no reaction information, so those pathways contribute empty reaction sets.
- MetNet does not currently support input of genomic data; it cannot reconstruct networks from experimental genomic data.
- KGML files must be downloaded through KEGG's public APIs; the list of KEGG organisms must be present as organismList.txt in the tool's directory.

## Evidence

- [methods] it is necessary to download the KGML files of each pathway of the organism through the public KEGG's APIs: [evidence span withheld — anchor status: not_found]
- [methods] to parse each KGML file to extract the relevant information: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [methods] comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""
- [methods] The first step in automatically reconstructing the metabolic network of a specific organism is automatic data retrieval.: "The first step in automatically reconstructing the metabolic network of a specific organism is automatic data retrieval."
- [methods] Our metabolism reconstruction method is based on the KEGG database.: "Our metabolism reconstruction method is based on the KEGG database."
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
