---
name: kegg-kgml-pathway-data-retrieval
description: Use when the goal is to automatically reconstruct a metabolic network
  for one or two KEGG organisms from KEGG database information, requiring organism-specific
  KGML downloads and parsing of compounds and reactions.
when_to_use_negative:
- 'When the starting data are raw genomic or experimental gene data: MetNet reconstructs
  solely from the KEGG database and ''MetNet does not currently support input of genomic
  data'''
- 'When a pathway comparison visualization from non-KEGG sources is needed: the reconstruction
  ''is based uniquely on the KEGG database, which can be a limit wrt. the represented
  information'''
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
tools:
- name: MetNet
  role: Java tool that automatically downloads KGML files of each pathway of the selected
    organism(s) via the public KEGG APIs, parses them, and builds the two-level metabolic
    network
  repo: https://github.com/simeoni-biolab/MetNet
- name: sax2r2.jar
  role: Java library used by MetNet to parse each KGML file and extract compounds
    and reactions
- name: poi-3.9.jar
  role: Apache POI library included in MetNet's classpath used during KGML parsing
- name: gs-core-1.3.jar
  role: GraphStream library used to hold the reconstructed structural graph of pathways
    after KGML parsing
- name: guava-19.0.jar
  role: Google Guava library in MetNet's classpath supporting the retrieval and parsing
    pipeline
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:942f43318e5828de956f64c3c8ec88bbf328d71e2b4e7b709974d576a4b01150
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
  - poi-3.9.jar
  - sax2r2.jar
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# kegg-kgml-pathway-data-retrieval

## Summary

Automatically retrieves the KGML files of every pathway of a selected KEGG organism via the public KEGG APIs and parses them to extract the compounds and reactions needed to reconstruct a two-level metabolic network, using the configuration files organismList.txt and pathwayList.txt.

## When to use

Use when the goal is to automatically reconstruct a metabolic network for one or two KEGG organisms from KEGG database information, requiring organism-specific KGML downloads and parsing of compounds and reactions.

## When NOT to use

- When the starting data are raw genomic or experimental gene data: MetNet reconstructs solely from the KEGG database and 'MetNet does not currently support input of genomic data'
- When a pathway comparison visualization from non-KEGG sources is needed: the reconstruction 'is based uniquely on the KEGG database, which can be a limit wrt. the represented information'

## Inputs

- KEGG organism code (e.g. 'hsa', 'api', 'bap', 'ptr') of the organism(s) to reconstruct
- Configuration file pathwayList.txt listing the KEGG reference pathways to be considered
- Configuration file organismList.txt containing the list of organisms available in KEGG
- Comparison method parameter ('set' or 'multiset') when reconstructing for pairwise comparison

## Outputs

- Parsed per-pathway compound sets and reaction sets extracted from KGML files
- Two-level metabolic network (structural adjacency matrix over KEGG reference pathways; functional reaction sets per pathway) for each organism

## How to apply

Confirm that the configuration files organismList.txt and pathwayList.txt are in the tool's directory; pathwayList.txt lists the reference pathways to be considered for the comparison and organismList.txt contains the KEGG organism codes. Invoke the tool with the KEGG organism code of the organism of interest (and a second organism plus the 'set' or 'multiset' comparison method if running a pairwise comparison), so that MetNet automatically downloads the KGML files of each pathway of the organism through the public KEGG's APIs. Parse each KGML file (MetNet uses the Java sax2r2 library and Apache POI) to extract the relevant information for the two-level representation, which are the compounds and the reactions of each pathway. On application start, MetNet also allows the user to download the latest list of KEGG organisms, substituting the local list with the new one. The parsed compound and reaction sets feed the structural graph (pathways as nodes, edges for shared non-ubiquitous compounds) and the functional reaction-set level, and, for pairwise comparisons, the similarity measures Psim, PsimW, Ssim and Csim.

## Related tools

- **MetNet** (Java tool that automatically downloads KGML files of each pathway of the selected organism(s) via the public KEGG APIs, parses them, and builds the two-level metabolic network) — https://github.com/simeoni-biolab/MetNet
- **sax2r2.jar** (Java library used by MetNet to parse each KGML file and extract compounds and reactions)
- **poi-3.9.jar** (Apache POI library included in MetNet's classpath used during KGML parsing)
- **gs-core-1.3.jar** (GraphStream library used to hold the reconstructed structural graph of pathways after KGML parsing)
- **guava-19.0.jar** (Google Guava library in MetNet's classpath supporting the retrieval and parsing pipeline)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- The set of downloaded pathways matches exactly the pathways listed in pathwayList.txt that the organism actually possesses
- Every parsed KGML file yields both a compound set and a reaction set for the corresponding pathway
- The structural adjacency matrix built from the parsed data contains the full set of considered reference pathways as rows and columns
- Ubiquitous compounds (H2O, phosphate, ATP, ADP) contribute no edges in the structural level, confirming the parser distinguished them from informative compounds
- Running with 'hsa ptr set' completes without error and produces both structural and functional levels for Homo sapiens and Pan troglodytes

## Limitations

- Reconstruction is limited to the KEGG database, 'which can be a limit wrt. the represented information'
- MetNet has no mechanism to reconstruct from genomic data: 'the tool MetNet has to be significantly extended to allow the input of genomic data'
- Obtaining the metabolism generally requires human intervention in general pipelines, but MetNet automates only KEGG-based reconstruction, so organisms outside KEGG cannot be processed

## Evidence

- [methods] KGML download step: [evidence span withheld — anchor status: not_found]
- [methods] KGML parsing step: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [results] organism list refresh: "the initial view allows the user to download the latest list of KEGG organisms: if the download is requested, the local list is substituted with the new one."
- [readme] README command-line parameters: "the following three parameters must be specified when starting the execution: 1. KEGG code of the first organism to compare 2. KEGG code of the second organism to compare 3. comparison method: "set""
- [readme] README configuration files: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
