---
name: graph-structure-similarity-index-calculation
description: Use when two KEGG-organism metabolic networks have been reconstructed
  as a two-level representation (structural graph + functional reaction sets) and
  a quantitative pathway-by-pathway and whole-metabolism similarity comparison is
  required.
when_to_use_negative:
- The input is raw genomic/experimental data rather than KEGG organisms — MetNet does
  not support genomic data input and reconstructs only from KEGG
- The networks are not decomposable into KEGG reference pathways, or come from non-KEGG
  sources such as BioCyc or BioModels, since the reconstruction is based uniquely
  on the KEGG database
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetNet
  role: Reconstructs two-level metabolic networks from KEGG KGML data and computes
    the local (SimPi, SimSi) and global (Psim, PsimW, Ssim, Csim) similarity indexes
    in 'set' or 'multiset' mode
  repo: https://github.com/simeoni-biolab/MetNet
- name: Java
  role: Compiles and runs the MetNet executable jar from a terminal window
- name: gs-algo-1.3.jar
  role: Graph library used on the classpath when compiling MetNet's structural graph
    functionality
- name: guava-19.0.jar
  role: Dependency on the javac classpath for building MetNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:dc5d76d8b8ebb4344b3ecfa87344296a7de6798d61edf161f8883dd29305a56c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - MetNet
  - gs-algo-1.3.jar
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# graph-structure-similarity-index-calculation

## Summary

Compute similarity indexes between two reconstructed metabolic networks at the structural level (pathway graphs, edge overlap) and functional level (reaction sets), yielding local indexes (SimPi, SimSi) and global indexes (Psim, PsimW, Ssim, Csim) as implemented in MetNet.

## When to use

Use when two KEGG-organism metabolic networks have been reconstructed as a two-level representation (structural graph + functional reaction sets) and a quantitative pathway-by-pathway and whole-metabolism similarity comparison is required.

## When NOT to use

- The input is raw genomic/experimental data rather than KEGG organisms — MetNet does not support genomic data input and reconstructs only from KEGG
- The networks are not decomposable into KEGG reference pathways, or come from non-KEGG sources such as BioCyc or BioModels, since the reconstruction is based uniquely on the KEGG database

## Inputs

- Two KEGG organism codes (three-letter codes such as api, bap, hsa, ptr)
- KEGG KGML pathway files (automatically downloaded by MetNet via KEGG public APIs)
- Configuration files organismList.txt and pathwayList.txt present in the tool directory
- Comparison method choice: 'set' or 'multiset'

## Outputs

- Local similarity values SimPi (functional, per pathway pair) and SimSi (structural, per pathway pair)
- Global similarity indexes: Psim (arithmetic mean), PsimW (weighted mean by |Ri ∪ Ri'|), Ssim (structure), Csim (combined SimSi*SimPi products)
- Results table of global index values (e.g. similarity_results.csv)
- Visual comparison of the two networks (weighted graphs)

## How to apply

Run MetNet with the two KEGG organism codes and a comparison method ('set' or 'multiset'); MetNet downloads each organism's KGML pathway files via the public KEGG APIs and parses them to extract compounds and reactions. At the functional level compute the local pathway similarity SimPi as the Jaccard index on reaction sets (handling missing, physical-only and reaction pathways via Case 1/2/3), then derive the global pathway similarity Psim as the arithmetic mean and PsimW as the weighted mean by |Ri ∪ Ri'|. At the structural level compute the local structure similarity SimSi from edge overlap between pathway graphs (with cases), the global structure similarity Ssim as the mean over SimSi cases, and the combined similarity Csim as the mean of the SimSi*SimPi products. Repeat with 'multiset' mode when repeated reactions within a pathway must be accounted for. All indexes are in [0, 1]; values near 0 indicate largely different metabolisms (e.g. api vs bap: 5.02%–14.73% across indexes).

## Related tools

- **MetNet** (Reconstructs two-level metabolic networks from KEGG KGML data and computes the local (SimPi, SimSi) and global (Psim, PsimW, Ssim, Csim) similarity indexes in 'set' or 'multiset' mode) — https://github.com/simeoni-biolab/MetNet
- **Java** (Compiles and runs the MetNet executable jar from a terminal window)
- **gs-algo-1.3.jar** (Graph library used on the classpath when compiling MetNet's structural graph functionality)
- **guava-19.0.jar** (Dependency on the javac classpath for building MetNet)

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- All four global indexes (Psim, PsimW, Ssim, Csim) fall within [0, 1]
- For a known dissimilar pair such as api vs bap, all global indexes stay low (paper reports 5.02%–14.73%) in both 'set' and 'multiset' modes, confirming largely different metabolisms
- 'set' and 'multiset' runs both complete and the two modes' index values are recorded for comparison
- All KGML pathway files listed in pathwayList.txt were successfully downloaded and parsed for both organisms before index computation
- Configuration files organismList.txt and pathwayList.txt were present in the tool directory at execution time

## Limitations

- The method is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; it would have to be significantly extended to reconstruct networks from experimental genomic data
- Data repositories are incomplete, heterogeneous and incoherent, so reconstruction generally requires care; the tool is constrained to pathways listed in pathwayList.txt

## Evidence

- [abstract] On the basis of this two-level representation, we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire: "On the basis of this two-level representation, we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [methods] download KGML files through KEGG APIs: [evidence span withheld — anchor status: not_found]
- [methods] parse each KGML file to extract the relevant information: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [methods] comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""
- [readme] README command-line parameters: "To run MetNet as a command line tool, the following three parameters must be specified when starting the execution: 1. KEGG code of the first organism to compare"
