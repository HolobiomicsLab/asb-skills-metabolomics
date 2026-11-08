---
name: functional-level-network-representation
description: Use when reconstructing the functional (reactions-based) level of an
  organism's metabolism from KEGG KGML pathway files, for local pathway-by-pathway
  or global similarity comparison.
when_to_use_negative:
- Input is already a reconstructed structural-level metabolic network (graph of pathways
  and relations) and no reactions-based content is needed
- The organism's data is not available in the KEGG database, since MetNet's reconstruction
  is based uniquely on KEGG information
- The goal is to reconstruct a metabolic network from raw genomic or experimental
  data, which MetNet does not currently support
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0797
tools:
- name: MetNet
  role: Java tool that automatically retrieves the KGML files of each pathway of the
    organism through the public KEGG APIs, parses them to extract reactions and compounds,
    and builds the functional-level reaction sets for each organism
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML Markup Language files that contain the reactions and compounds
    of each pathway map, used to construct the functional level representation
- name: guava-19.0.jar
  role: Software library listed in MetNet's compilation classpath for building the
    executable that performs the KGML parsing and reaction-set construction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:ec4479a9c976bb571451c24b08849e9994995d688feacf804e55ac86e30926f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# functional-level-network-representation

## Summary

Builds the second level of MetNet's two-level metabolic network representation: for each KEGG reference pathway of an organism, a set (or multiset) of reactions extracted from the pathway's KGML file.

## When to use

Use when reconstructing the functional (reactions-based) level of an organism's metabolism from KEGG KGML pathway files, for local pathway-by-pathway or global similarity comparison.

## When NOT to use

- Input is already a reconstructed structural-level metabolic network (graph of pathways and relations) and no reactions-based content is needed
- The organism's data is not available in the KEGG database, since MetNet's reconstruction is based uniquely on KEGG information
- The goal is to reconstruct a metabolic network from raw genomic or experimental data, which MetNet does not currently support

## Inputs

- KEGG organism codes for the two organisms to compare (e.g. hsa, ptr)
- KEGG KGML pathway files for each organism, retrieved via KEGG's public APIs
- pathwayList.txt — list of KEGG reference pathways to consider for the comparison
- organismList.txt — list of organisms available in KEGG
- comparison method parameter: "set" or "multiset"

## Outputs

- reaction_sets_<organism>.csv — per-organism pathway-to-reaction mapping
- reaction_sets.csv — combined table with columns organism, pathway_id, reaction_id (and multiplicity in multiset mode)
- per-reference-pathway reaction-set records (pathway ID -> set or multiset of reactions)

## How to apply

Select the two KEGG organism codes (e.g. hsa, ptr) and the comparison method ('set' or 'multiset') as MetNet input, using organismList.txt and pathwayList.txt as catalogs. MetNet automatically retrieves, via KEGG's public APIs, the KGML files of each pathway of each organism. Parse each KGML file to extract the chemical reactions (and compounds) of each pathway map — these constitute the functional-level data; membrane or physical-mechanism pathways yield empty reaction sets because their KGML files contain no reaction information. Per pathway, record the reactions as a set (or multiset, preserving multiplicity) according to the chosen comparison method. Write per-organism tables reaction_sets_<organism>.csv and a combined reaction_sets.csv with columns organism, pathway_id, reaction_id (plus multiplicity in multiset mode). Validate that each pathway in the catalog yields exactly one reaction-set row and that reaction IDs match the reactions present in the corresponding pathway map's KGML file.

## Related tools

- **MetNet** (Java tool that automatically retrieves the KGML files of each pathway of the organism through the public KEGG APIs, parses them to extract reactions and compounds, and builds the functional-level reaction sets for each organism) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML Markup Language files that contain the reactions and compounds of each pathway map, used to construct the functional level representation)
- **guava-19.0.jar** (Software library listed in MetNet's compilation classpath for building the executable that performs the KGML parsing and reaction-set construction)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Each pathway listed in the organism's catalog yields exactly one reaction-set row in reaction_sets_<organism>.csv
- Empty reaction sets are permitted only for membrane or physical-mechanism pathways, since their KGML files contain no reaction information
- Every reaction_id in the output table matches a reaction present in the corresponding pathway map's KGML file
- In multiset mode, multiplicity values are recorded alongside reaction IDs; in set mode each reaction appears at most once per pathway
- The combined reaction_sets.csv contains rows for both organisms with the schema organism, pathway_id, reaction_id

## Limitations

- Reconstruction is based uniquely on the KEGG database, which constrains the represented information
- Membrane and physical-mechanism pathways produce empty reaction sets because their KGML files contain no reaction information
- MetNet has to be significantly extended to allow the input of genomic data and reconstruct networks from experimental genomic data

## Evidence

- [methods] KGML parsing to extract functional-level reactions: [evidence span withheld — anchor status: not_found]
- [methods] what is extracted from KGML: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [readme] comparison method parameter: "3. comparison method: "set" or "multiset""
- [discussion] KEGG-only data source: "is based uniquely on the KEGG database, which can be a limit wrt. the represented information"
- [discussion] no genomic-data input: "the tool MetNet has to be significantly extended to allow the input of genomic data and implement the reconstruction of the corresponding metabolic network"
- [intro] two-level representation: "representing a metabolic network in two distinct levels: the structural level and the functional level."
