---
name: kegg-pathway-similarity-index-computation
description: Use when two KEGG organisms' metabolic networks have been reconstructed
  from KGML files and you need quantitative pairwise comparison scores (SimPi, SimSi,
  Psim, PsimW, Csim) in [0,1].
when_to_use_negative:
- When no organism pair has been reconstructed yet from KEGG — the indexes require
  KGML-derived reaction sets and pathway graphs as input
- When you need reconstruction from raw genomic/experimental data rather than comparison
  of KEGG-listed organisms (MetNet does not currently support input of genomic data)
- When the organisms are not both available in the KEGG database, since the reconstruction
  is based uniquely on KEGG
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0605
tools:
- name: MetNet
  role: Implements the reconstruction and computes all local and global similarity
    indexes (SimPi, SimSi, Psim, PsimW, Csim) via its comparison window
  repo: github.com/simeoni-biolab/MetNet
- name: Java
  role: Compiler/runtime for MetNet; the tool is compiled with javac and run via java
    -jar MetNet.jar
- name: guava-19.0.jar
  role: Library on the MetNet classpath required at compile time
- name: KEGG database
  role: Source of KGML pathway data and organism codes from which the reaction sets
    and pathway graphs are derived before index computation
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:06a0b2428cbc884ded7f8639fa3814c75568cb20b7193ebec823326c84e5edfe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kegg-pathway-similarity-index-computation

## Summary

Compute MetNet's local (SimPi, SimSi) and global (Psim, PsimW, Csim) similarity indexes that quantify structural and functional similarity between the metabolic networks of two KEGG organisms. The indexes are Jaccard-based at the pathway level and aggregate pathway-by-pathway results into organism-level comparison scores.

## When to use

Use when two KEGG organisms' metabolic networks have been reconstructed from KGML files and you need quantitative pairwise comparison scores (SimPi, SimSi, Psim, PsimW, Csim) in [0,1].

## When NOT to use

- When no organism pair has been reconstructed yet from KEGG — the indexes require KGML-derived reaction sets and pathway graphs as input
- When you need reconstruction from raw genomic/experimental data rather than comparison of KEGG-listed organisms (MetNet does not currently support input of genomic data)
- When the organisms are not both available in the KEGG database, since the reconstruction is based uniquely on KEGG

## Inputs

- KGML files for each pathway of the two organisms, retrieved through the public KEGG's APIs
- pathwayList.txt configuration file listing the KEGG reference pathways to consider for the comparison
- Comparison mode selection: 'set' or 'multiset' for reaction content
- KEGG organism codes for the two organisms (e.g. hsa, ptr)

## Outputs

- Per-pathway table with SimPi and SimSi for each KEGG reference pathway present in either organism (with |Ri∪Ri′| supporting the reaction count used by weighted aggregation)
- Global similarity index Psim (arithmetic mean of local SimPi values)
- Weighted global similarity index PsimW (weighted mean with weights |Ri∪Ri′|)
- Combined similarity index Csim = Σ SimSi·SimPi/n
- Structure similarity index Ssim (arithmetic mean of SimSi over |V∪V′| nodes)

## How to apply

Load the KGML-derived pathway data (reactions and compound-sharing relations) for both organisms and iterate over the reference pathways listed in pathwayList.txt. For each reference pathway Pi compute the functional index SimPi using the chosen comparison mode ('set' or 'multiset'): set SimPi=0 if the pathway is present in only one organism, SimPi=1 if present in both but containing no reactions (physical pathway), otherwise the Jaccard index |Ri∩Ri′|/|Ri∪Ri′| over the reaction sets/multisets. For the structural level compute SimSi on the reconstructed pathway graphs: 0 if the node is in only one graph, 1 if isolated in both, 1/(1+k) if isolated in one graph and connected with degree k in the other, and |Ei∩Ei′|/|Ei∪Ei′| if connected in both. Aggregate the local indexes: Psim as the arithmetic mean of SimPi over the n KEGG pathways present in either organism; PsimW as the weighted mean of SimPi with weights |Ri∪Ri′| over reaction pathways only; Csim as Σ SimSi·SimPi/n, which amplifies differences because both factors lie in [0,1]. Record all indexes in a summary table with one row per reference pathway.

## Related tools

- **MetNet** (Implements the reconstruction and computes all local and global similarity indexes (SimPi, SimSi, Psim, PsimW, Csim) via its comparison window) — github.com/simeoni-biolab/MetNet
- **Java** (Compiler/runtime for MetNet; the tool is compiled with javac and run via java -jar MetNet.jar)
- **guava-19.0.jar** (Library on the MetNet classpath required at compile time)
- **KEGG database** (Source of KGML pathway data and organism codes from which the reaction sets and pathway graphs are derived before index computation)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every computed SimPi and SimSi lies in the interval [0, 1] as required by the index definitions
- The per-pathway table covers all KEGG reference pathways present in organism O or O′ (not only those in both)
- SimPi=0 for pathways present in only one organism and SimPi=1 for pathways present in both but with no reactions (physical pathways)
- PsimW differs from Psim because aggregation weights are |Ri∪Ri′| and only reaction pathways are considered
- Csim amplifies differences relative to Psim since it is a product Σ SimSi·SimPi/n of two factors in [0,1]

## Limitations

- The reconstruction and comparison are based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; it has to be significantly extended to allow genomic input and reconstruction of the corresponding metabolic network
- The choice of 'set' vs 'multiset' comparison mode affects SimPi: repeated reactions are either treated as identical or individually weighted

## Evidence

- [results] set SimPi=0 if the pathway is present in only one organism, SimPi=1 if the pathway is present in both but contains no reactions (physical pathway), otherwise compute the Jaccard index |Ri∩Ri′|/|Ri∪Ri′|: [evidence span withheld — anchor status: not_found]
- [results] MetNet computes the global index Psim as the mean of the local structural similarity values, applies a weighting by pathway degree for the weighted variant PsimW: [evidence span withheld — anchor status: not_found]
- [intro] The comparison method based on the two-level representation makes it possible to visualise the network structures and explore their similarities and differences, and to compute similarity indexes: "The comparison method based on the two-level representation makes it possible to visualise the network structures and explore their similarities and differences, and to compute similarity indexes"
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."
- [readme] java -jar MetNet.jar hsa ptr set: "will start the execution for the organisms "hsa" (Homo Sapiens) and "prt" (Pan troglodytes) with the comparison method "set"."
