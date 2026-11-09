---
name: reference-pathway-to-organism-pathway-mapping
description: 'Use when reconstructing the functional level of a metabolic network
  from KEGG: you have an organism KEGG code (e.g. hsa, ptr) and need per-reference-pathway
  reaction sets for set/multiset comparison.'
when_to_use_negative:
- The goal is reconstruction from newly annotated genomes or experimental genomic
  data — MetNet is 'based uniquely on the KEGG database' and does not support genomic-data
  input.
- A graphical visualisation of pathway comparison results is the primary deliverable;
  this skill produces reaction-set records, not network plots.
- Metabolic databases other than KEGG (e.g. BioCyc, BioModels) are the data source;
  the KGML parsing here is KEGG-specific.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0780
tools:
- name: MetNet
  role: Java tool that performs the KGML download, parsing and reaction-set construction
    for the functional level
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of KGML files via public APIs; defines reference pathways and organism
    codes
- name: guava-19.0.jar
  role: Java library dependency on MetNet's compile-time classpath used during the
    parsing workflow
  repo: https://github.com/simeoni-biolab/MetNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:095907305efb92bdd4ac9c43492f42fcd87d43811c73b5b36b777a636a243041
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reference-pathway-to-organism-pathway-mapping

## Summary

Downloads the KEGG KGML files for each reference pathway of a selected organism via KEGG's public APIs and parses them to extract the compounds and reactions that constitute the functional-level (reaction-based) content of that pathway, treating physical-mechanism pathways as empty reaction sets.

## When to use

Use when reconstructing the functional level of a metabolic network from KEGG: you have an organism KEGG code (e.g. hsa, ptr) and need per-reference-pathway reaction sets for set/multiset comparison.

## When NOT to use

- The goal is reconstruction from newly annotated genomes or experimental genomic data — MetNet is 'based uniquely on the KEGG database' and does not support genomic-data input.
- A graphical visualisation of pathway comparison results is the primary deliverable; this skill produces reaction-set records, not network plots.
- Metabolic databases other than KEGG (e.g. BioCyc, BioModels) are the data source; the KGML parsing here is KEGG-specific.

## Inputs

- Organism KEGG code (e.g. hsa for Homo sapiens, ptr for Pan troglodytes)
- Pathway catalog file pathwayList.txt (list of KEGG reference pathways to consider)
- Organism catalog file organismList.txt (list of organisms available in KEGG)
- Comparison method parameter: 'set' or 'multiset'
- KGML files of each pathway of the organism, downloaded via KEGG's public APIs

## Outputs

- Per-organism reaction-set records: pathway ID -> set or multiset of reaction IDs (chosen by the 'set'/'multiset' comparison method)
- Tables reaction_sets_<organism>.csv for both organisms and a combined reaction_sets.csv with columns: organism, pathway_id, reaction_id (and multiplicity for multiset mode)
- Empty reaction-set records for physical-mechanism pathways (e.g. membrane pathways) whose KGML contains no reaction entries

## How to apply

Read the organism KEGG codes (e.g. hsa, ptr) and the comparison method ('set' or 'multiset') from the MetNet configuration, using organismList.txt and pathwayList.txt as the pathway catalog. For each pathway listed in pathwayList.txt, download the organism-specific KGML file through KEGG's public APIs. Parse each KGML file to extract the compounds and the reactions of the pathway — these are the 'relevant information for our representation' at the functional level. Treat membrane/physical-mechanism pathways as empty reaction sets, since their KGML files contain no reaction information. Depending on the chosen comparison method, build each pathway's reaction content as a plain set of reaction IDs ('set') or as a multiset preserving reaction multiplicity ('multiset'). Emit per-organism reaction-set records (pathway ID -> reaction set/multiset) that downstream MetNet similarity measures consume for pathway-by-pathway and global comparison.

## Related tools

- **MetNet** (Java tool that performs the KGML download, parsing and reaction-set construction for the functional level) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of KGML files via public APIs; defines reference pathways and organism codes)
- **guava-19.0.jar** (Java library dependency on MetNet's compile-time classpath used during the parsing workflow) — https://github.com/simeoni-biolab/MetNet

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Each pathway present in the organism yields exactly one reaction-set record (empty reaction sets allowed for physical pathways such as membrane/mechanism maps).
- Reaction IDs extracted match the reactions listed in the corresponding pathway map's KGML file.
- The set of pathway IDs in the output equals the set listed in pathwayList.txt for the chosen organism.
- For 'multiset' mode, multiplicity counts match repeated reaction entries in the KGML; for 'set' mode, duplicates collapse to unique reaction IDs.
- Compounds extracted from each KGML file are consistent with the pathway's compound list used to build the structural level.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- Physical-mechanism pathways (e.g. membrane pathways) produce empty reaction sets because their KGML files contain no reaction information.
- MetNet does not currently support input of genomic data; a significant extension would be required for reconstruction from newly annotated genomes.

## Evidence

- [methods] download the KGML files of each pathway of the organism through the public KEGG's APIs: [evidence span withheld — anchor status: not_found]
- [methods] parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [methods] comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""
- [abstract] The two-level representation complies with the KEGG database, which decomposes the metabolism of all the different organisms into “reference” pathways in a standardised way.: "The two-level representation complies with the KEGG database, which decomposes the metabolism of all the different organisms into “reference” pathways"
- [readme] MetNet will start the execution and the user will be guided through the various steps of the comparison by the visual interface.: "MetNet will start the execution and the user will be guided through the various steps of the comparison by the visual interface."
