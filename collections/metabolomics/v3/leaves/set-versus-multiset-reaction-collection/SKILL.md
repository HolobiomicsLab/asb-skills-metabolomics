---
name: set-versus-multiset-reaction-collection
description: Use when reconstructing the functional (reactions-based) level of a metabolic
  network from KEGG for an organism pair, and you must decide whether each pathway's
  reactions should be compared as a set or a multiset.
when_to_use_negative:
- The structural (graph-based) level of the metabolic network is the target — this
  skill only builds the functional (reactions-based) level.
- Reconstruction must start from genomic or experimental data rather than KEGG reference
  pathways; MetNet does not support genomic data input and relies solely on KEGG.
- An organism is not in the KEGG organism list (organismList.txt), since retrieval
  is exclusively through KEGG KGML files.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0780
tools:
- name: MetNet
  role: Java tool that drives KGML retrieval via KEGG public APIs, parses reactions/compounds
    per pathway, and builds the set- or multiset-based functional-level records for
    the organism pair
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: source of the KGML pathway files whose reactions define the functional level;
    retrieved via KEGG public APIs
- name: guava-19.0.jar
  role: library on MetNet's compile classpath needed to build and run the tool that
    performs this collection
- name: Java compiler (javac)
  role: compiles the MetNet source before execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:8978e82ca669172f76b9eedf7df7d1a0b86a762fd9317922edac7104fdaa5476
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - Java compiler (javac)
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# set-versus-multiset-reaction-collection

## Summary

MetNet's functional-level collection step: retrieve the KGML files of each KEGG reference pathway for two organisms, parse out the reactions (and compounds), and record them as either sets or multisets of reaction IDs per pathway, as chosen by the 'set' or 'multiset' comparison method. This produces the per-organism reaction-set records that the two-level comparison requires.

## When to use

Use when reconstructing the functional (reactions-based) level of a metabolic network from KEGG for an organism pair, and you must decide whether each pathway's reactions should be compared as a set or a multiset.

## When NOT to use

- The structural (graph-based) level of the metabolic network is the target — this skill only builds the functional (reactions-based) level.
- Reconstruction must start from genomic or experimental data rather than KEGG reference pathways; MetNet does not support genomic data input and relies solely on KEGG.
- An organism is not in the KEGG organism list (organismList.txt), since retrieval is exclusively through KEGG KGML files.

## Inputs

- KEGG code of the first organism to compare (e.g. 'hsa')
- KEGG code of the second organism to compare (e.g. 'ptr')
- comparison method: 'set' or 'multiset'
- organismList.txt configuration file listing KEGG organisms
- pathwayList.txt configuration file listing the reference pathways to consider
- KGML files of each pathway of each organism downloaded via the public KEGG APIs

## Outputs

- reaction_sets_<organism>.csv for each organism (pathway ID -> set or multiset of reactions)
- combined reaction_sets.csv with columns: organism, pathway_id, reaction_id (and multiplicity for multiset mode)

## How to apply

Read the two KEGG organism codes (e.g. 'hsa' for Homo sapiens and 'ptr' for Pan troglodytes) and the comparison method ('set' or 'multiset') from the command line or configuration, using organismList.txt and pathwayList.txt as the organism and pathway catalogs. For each reference pathway listed for each organism, download the pathway's KGML file through the public KEGG APIs. Parse each KGML file to extract the reactions (and compounds) that constitute the functional-level data, yielding one reaction-set record per organism per reference pathway. Represent each pathway's reaction content as a set when the comparison method is 'set', or as a multiset (recording reaction multiplicity) when it is 'multiset'; physical/mechanism pathways whose KGML contains no reactions yield empty reaction sets. Write the results as reaction_sets_<organism>.csv for both organisms plus a combined reaction_sets.csv with columns organism, pathway_id, reaction_id (and multiplicity in multiset mode). The rationale is that MetNet's functional level captures each pathway by its reaction content, and the set/multiset choice governs how reaction counts influence downstream similarity.

## Related tools

- **MetNet** (Java tool that drives KGML retrieval via KEGG public APIs, parses reactions/compounds per pathway, and builds the set- or multiset-based functional-level records for the organism pair) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (source of the KGML pathway files whose reactions define the functional level; retrieved via KEGG public APIs)
- **guava-19.0.jar** (library on MetNet's compile classpath needed to build and run the tool that performs this collection)
- **Java compiler (javac)** (compiles the MetNet source before execution)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Each pathway listed for the organism in pathwayList.txt yields exactly one reaction-set record in reaction_sets_<organism>.csv.
- Every reaction_id in the output matches a reaction that actually appears in that pathway's KGML file.
- In 'multiset' mode, multiplicity values correctly reflect repeated reaction occurrences in the KGML; in 'set' mode, each reaction appears at most once per pathway.
- Physical/mechanism pathways (whose KGML contains no reaction information) produce empty reaction sets rather than missing rows.
- The combined reaction_sets.csv contains rows for both organisms with consistent columns (organism, pathway_id, reaction_id, multiplicity when multiset).

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- Pathways of physical mechanisms contribute empty reaction sets because their KGML files contain no reaction information.
- The choice between 'set' and 'multiset' must be fixed at invocation; switching methods requires re-running the collection.
- MetNet does not currently accept genomic data input, so reaction sets can only be derived from KEGG reference pathways, not from newly annotated genomes.

## Evidence

- [abstract] Automatic reconstruction of a metabolic network from KEGG database information: "We propose to automatically reconstruct a metabolic network on the basis of KEGG database information."
- [methods] Download and parse KGML files for pathway reactions: [evidence span withheld — anchor status: not_found]
- [methods] Parse each KGML file for compounds and reactions: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [methods] Comparison method parameter: "3. comparison method: "set" or "multiset""
- [readme] Command-line parameters and example invocation: "java -jar MetNet.jar hsa ptr set"
- [readme] Configuration files required: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
