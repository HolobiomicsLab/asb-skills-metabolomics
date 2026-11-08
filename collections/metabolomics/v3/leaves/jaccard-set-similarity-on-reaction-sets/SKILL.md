---
name: jaccard-set-similarity-on-reaction-sets
description: Use when two organisms' metabolisms have been reconstructed from KEGG
  KGML into per-pathway reaction sets and you need quantitative local and global functional
  similarity between them.
when_to_use_negative:
- The organism data is not in KEGG and cannot be retrieved as KGML — MetNet's reconstruction
  is based uniquely on the KEGG database, which the authors note 'can be a limit wrt.
  the represented information'
- You have raw annotated genomic data instead of KEGG pathway entries — MetNet does
  not currently support input of genomic data
- You need structural (topological graph) similarity rather than functional reaction-set
  similarity — use the Ssim/Csim structural-level indexes instead of Psim/PsimW
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetNet
  role: Reconstructs the two organisms' KEGG pathways into reaction sets and computes
    the per-pathway SimPi and global Psim/PsimW indexes in 'set' or 'multiset' comparison
    mode
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML files and the standardized decomposition of each organism's
    metabolism into reference pathways used as the units of comparison
- name: Java
  role: Runtime and compiler used to build and run the MetNet jar
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:5290d28302401295cc47f01bea59f0b1787693dbbe5c077bc1016c3e3d0e17a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# Jaccard set similarity on reaction sets

## Summary

Computes the local pathway similarity index SimPi as the Jaccard ratio |Ri ∩ Ri′| / |Ri ∪ Ri′| over reaction sets of KEGG reference pathways, then aggregates per-pathway values into the global indexes Psim (arithmetic mean) and PsimW (mean weighted by |Ri ∪ Ri′|) to compare two organisms' metabolisms.

## When to use

Use when two organisms' metabolisms have been reconstructed from KEGG KGML into per-pathway reaction sets and you need quantitative local and global functional similarity between them.

## When NOT to use

- The organism data is not in KEGG and cannot be retrieved as KGML — MetNet's reconstruction is based uniquely on the KEGG database, which the authors note 'can be a limit wrt. the represented information'
- You have raw annotated genomic data instead of KEGG pathway entries — MetNet does not currently support input of genomic data
- You need structural (topological graph) similarity rather than functional reaction-set similarity — use the Ssim/Csim structural-level indexes instead of Psim/PsimW

## Inputs

- KEGG KGML files of each pathway of two organisms, downloaded via the public KEGG APIs and parsed into compounds and reactions
- KEGG organism codes for the two organisms (e.g. api and bap)
- pathwayList.txt configuration file listing the KEGG reference pathways considered for the comparison

## Outputs

- Per-pathway SimPi values (in [0,1]) for every reference pathway in pathwayList.txt
- Global pathway similarity index Psim and weighted pathway similarity index PsimW (in [0,1])
- Complementarity inventory of pathways whose reactions occur in only one organism (e.g. complementarity_analysis.csv)

## How to apply

For each KEGG reference pathway listed in pathwayList.txt, extract the reaction set Ri from each organism's KGML-derived reconstruction and compute SimPi according to three cases: pathways present in one organism only get SimPi = 0; pathways in both that are reaction pathways get the Jaccard index |Ri ∩ Ri′| / |Ri ∪ Ri′|; pathways based on physical mechanisms present in both get SimPi = 1. Aggregate the per-pathway values into Psim, the arithmetic mean over all pathways present in either organism, and into PsimW, the weighted mean where each pathway is weighted by |Ri ∪ Ri′| (noting PsimW ignores pathways dealing with physical transformations). Run MetNet in 'set' mode (java -jar MetNet.jar <org1> <org2> set) so reactions are treated as sets; rerun in 'multiset' mode if repeated reactions must be accounted for. Low Psim/PsimW values across all reference pathways indicate largely different metabolisms, and pathways with SimPi = 0 in one organism feed the complementarity inventory.

## Related tools

- **MetNet** (Reconstructs the two organisms' KEGG pathways into reaction sets and computes the per-pathway SimPi and global Psim/PsimW indexes in 'set' or 'multiset' comparison mode) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML files and the standardized decomposition of each organism's metabolism into reference pathways used as the units of comparison)
- **Java** (Runtime and compiler used to build and run the MetNet jar)

## Examples

```
java -jar MetNet.jar api bap set
```

## Evaluation signals

- Every KEGG reference pathway listed in pathwayList.txt receives a SimPi value
- All computed SimPi values, and the aggregated Psim and PsimW, fall within [0,1]
- The Jaccard ratio is computed only for pathways present in both organisms (SimPi = 0 flags one-organism-only pathways; SimPi = 1 flags physical-mechanism pathways present in both)
- PsimW equals the |Ri ∪ Ri′|-weighted mean and excludes pathways dealing with physical transformations, while Psim averages over all pathways present in either organism
- For api vs bap, the global values fall in the reported low range 5.02%–14.73% across indexes, reproducing the 'largely different metabolisms' qualitative finding

## Limitations

- Reconstruction relies uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; reconstructing from experimental genomic data would require extending the tool
- The configuration files organismList.txt and pathwayList.txt must be present in the tool's directory or the comparison cannot enumerate the pathways

## Evidence

- [abstract] Similarity measures are introduced for both levels, enabling local (pathway-by-pathway) and global (entire metabolism) comparison.: "On the basis of this two-level representation, we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global"
- [methods] Comparison method set or multiset: "comparison method: "set" or "multiset""
- [results] Pathway rows/columns correspond to reference pathways from the configuration file: "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [readme] README command-line execution with comparison method: "will start the execution for the organisms "hsa" (Homo Sapiens) and "prt" (Pan troglodytes) with the comparison method "set"."
