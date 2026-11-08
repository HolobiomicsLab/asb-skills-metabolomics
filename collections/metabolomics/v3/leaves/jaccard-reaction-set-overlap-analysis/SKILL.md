---
name: jaccard-reaction-set-overlap-analysis
description: Use when comparing the reaction content of corresponding KEGG reference
  pathways in two organisms, and you need a normalized per-pathway similarity in [0,1]
  computed over reaction sets or multisets.
when_to_use_negative:
- When the goal is structural/topological comparison of pathway graphs (node connections
  via shared compounds) — use the SimSi index instead
- When only one organism is involved; the Jaccard overlap is undefined without a second
  organism's reaction set
- When metabolic networks are reconstructed from sources other than KEGG (e.g. BioCyc,
  BioModels, genomic data), since MetNet's parsing and pathway identification are
  KEGG-based
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: MetNet
  role: Java tool that parses KEGG KGML files into per-pathway reaction sets and computes
    the functional-level SimPi Jaccard index between two organisms, supporting 'set'
    or 'multiset' comparison modes
  repo: simeoni-biolab/MetNet
- name: guava-19.0.jar
  role: Java library on the compilation classpath required to build MetNet, which
    performs the reaction-set Jaccard computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:d410914403b0df6fbcb16ccab56c89393114cbd6c95d0cec348b8b26e6a6b1c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# jaccard-reaction-set-overlap-analysis

## Summary

Computes the functional-level pathway similarity SimPi as a Jaccard index over reaction sets or multisets reconstructed from KEGG KGML data, so two organisms' metabolic networks can be compared pathway by pathway.

## When to use

Use when comparing the reaction content of corresponding KEGG reference pathways in two organisms, and you need a normalized per-pathway similarity in [0,1] computed over reaction sets or multisets.

## When NOT to use

- When the goal is structural/topological comparison of pathway graphs (node connections via shared compounds) — use the SimSi index instead
- When only one organism is involved; the Jaccard overlap is undefined without a second organism's reaction set
- When metabolic networks are reconstructed from sources other than KEGG (e.g. BioCyc, BioModels, genomic data), since MetNet's parsing and pathway identification are KEGG-based

## Inputs

- KEGG KGML files for each pathway of each organism (downloaded via KEGG's public APIs)
- KEGG organism codes for the two organisms being compared (e.g. hsa, ptr)
- pathwayList.txt configuration file listing the reference pathways to include in the comparison
- Parsed per-pathway reaction sets/multisets for each organism (from MetNet's KGML parsing)
- Comparison method parameter: 'set' or 'multiset'

## Outputs

- Per-pathway SimPi similarity index in [0,1] for each KEGG reference pathway present in either organism
- Reaction-set intersection size |Ri ∩ Ri′| and union size |Ri ∪ Ri′| per pathway (supporting the Jaccard computation)
- A comparison table with one row per reference pathway, carrying SimPi (and SimSi from the paired structural index)

## How to apply

Reconstruct both organisms' networks from KEGG KGML files (parsed with MetNet) and, iterating over the reference pathways listed in pathwayList.txt, apply the functional index SimPi. If the reference pathway is present in only one organism, set SimPi = 0; if present in both but containing no reactions (a physical pathway), set SimPi = 1; otherwise compute the Jaccard index |Ri ∩ Ri′| / |Ri ∪ Ri′| over the two organisms' reaction sets for that pathway. Choose the comparison mode — 'set' or 'multiset' — since in multiset mode repeated reactions are kept as multiple elements (weighted), whereas set mode treats repeats as identical. The set-mode Jaccard requires elements common to both organisms; pathway presence in O or O′ must be established from the KGML-derived reaction lists before the index is computed. Values below 0 or above 1 indicate an implementation error, since both indexes are defined on the interval [0,1].

## Related tools

- **MetNet** (Java tool that parses KEGG KGML files into per-pathway reaction sets and computes the functional-level SimPi Jaccard index between two organisms, supporting 'set' or 'multiset' comparison modes) — simeoni-biolab/MetNet
- **guava-19.0.jar** (Java library on the compilation classpath required to build MetNet, which performs the reaction-set Jaccard computation)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every computed SimPi value must lie in the interval [0, 1]
- Pathways present in only one organism must yield SimPi = 0, not a Jaccard fraction
- Pathways present in both organisms but containing no reactions must yield SimPi = 1 (physical pathway)
- In multiset mode, pathways whose repeated reactions differ in multiplicity must score below 1, whereas the same reaction set without repeats scores 1 in set mode
- The output table must cover all KEGG reference pathways present in either organism, as listed in pathwayList.txt

## Limitations

- Reconstruction is based uniquely on the KEGG database, which constrains the represented information
- MetNet does not currently accept input of genomic data; reaction sets must come from KEGG KGML, not experimental genomic data
- The comparison method choice ('set' vs 'multiset') changes the meaning of the index — set mode treats repeated reactions as identical, multiset mode weighs them individually, so results are not directly comparable across modes

## Evidence

- [methods] MetNet comparison window: set vs multiset for SimPi: "3. comparison method: "set" or "multiset""
- [readme] Jaccard index definition over reaction sets/multisets: "java -jar MetNet.jar hsa ptr set"
- [abstract] KEGG reference pathways as standard decomposition: "The two-level representation complies with the KEGG database, which decomposes the metabolism of all the different organisms into “reference” pathways in a standardised way."
- [abstract] Similarity measures allow local, pathway-by-pathway comparison: "On the basis of this two-level representation, we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [abstract] Automatic reconstruction from KEGG database information: "We propose to automatically reconstruct a metabolic network on the basis of KEGG database information."
