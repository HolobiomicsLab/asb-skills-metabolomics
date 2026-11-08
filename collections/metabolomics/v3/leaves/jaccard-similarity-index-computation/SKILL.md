---
name: jaccard-similarity-index-computation
description: Use when the goal is to quantify how similar the metabolisms of two (or
  more) KEGG organisms are, either locally (per reference pathway) or globally (whole
  metabolism), e.
when_to_use_negative:
- 'The organism is not available in KEGG or you need a reconstruction from non-KEGG
  sources: MetNet''s reconstruction is based uniquely on the KEGG database, which
  limits the represented information.'
- 'You want to reconstruct networks from raw or newly annotated genomic data: MetNet
  does not currently support input of genomic data and would need significant extension.'
- You need per-reaction multiplicity-aware similarity but the pathway reaction content
  is only meaningfully compared as presence/absence sets — in that case reconsider
  the 'set' vs 'multiset' choice, but do not apply this skill to non-pathway data
  (e.g. plain gene lists).
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MetNet
  role: Reconstructs the metabolic networks of two KEGG organisms from KGML data and
    computes the structural (Ssim), pathway (Psim), weighted pathway (PsimW) and combined
    (Csim) similarity indexes with the 'set' or 'multiset' comparison method.
  repo: github.com/simeoni-biolab/MetNet
- name: Java
  role: Runtime/compiler required to build and execute MetNet from the source jar
    files (gs-algo-1.3.jar, gs-core-1.3.jar, guava-19.0.jar, etc.).
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:f3ce9b8a16380f258759742a99133c4c1f3703b6c9549fc2629a2d0dea9ad89d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MetNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# jaccard-similarity-index-computation

## Summary

Computes Jaccard-based similarity indexes (Ssim, Psim, PsimW, Csim) between the metabolic networks of two KEGG organisms, as implemented by the MetNet tool. Pathways are represented as sets of reactions and compared pathway-by-pathway, then aggregated into global whole-metabolism scores used for pairwise or multi-organism clustering analyses.

## When to use

Use when the goal is to quantify how similar the metabolisms of two (or more) KEGG organisms are, either locally (per reference pathway) or globally (whole metabolism), e.g. to build similarity matrices for hierarchical clustering of organisms or to detect metabolic complementation between a host and its symbiont.

## When NOT to use

- The organism is not available in KEGG or you need a reconstruction from non-KEGG sources: MetNet's reconstruction is based uniquely on the KEGG database, which limits the represented information.
- You want to reconstruct networks from raw or newly annotated genomic data: MetNet does not currently support input of genomic data and would need significant extension.
- You need per-reaction multiplicity-aware similarity but the pathway reaction content is only meaningfully compared as presence/absence sets — in that case reconsider the 'set' vs 'multiset' choice, but do not apply this skill to non-pathway data (e.g. plain gene lists).

## Inputs

- KEGG organism codes of the two organisms to compare (e.g. 'hsa', 'ptr'; 'apis' and 'bap' in the complementation experiment)
- KEGG KGML pathway files retrieved via KEGG's public APIs
- Configuration file pathwayList.txt listing the KEGG reference pathways to consider for the comparison
- Configuration file organismList.txt with the list of KEGG organisms
- Comparison method choice: 'set' or 'multiset'

## Outputs

- Per-pathway similarity values SimPi for every reference pathway in pathwayList.txt
- Global functional-level similarity indexes Psim and PsimW for the organism pair
- Structural-level similarity index Ssim and combined index Csim
- Similarity/distance matrix (rows and columns = KEGG reference pathways or organisms) for multi-organism clustering
- Comparison results presented visually and quantitatively by MetNet

## How to apply

First reconstruct each organism's metabolism with MetNet: download the KGML files of each pathway via KEGG's public APIs and parse them to extract compounds and reactions. Run MetNet in command-line mode passing the two KEGG organism codes and the comparison method 'set', so each pathway is represented as a set of reactions (use 'multiset' if reaction multiplicity matters). For every KEGG reference pathway listed in pathwayList.txt compute the per-pathway similarity SimPi: Jaccard ratio |Ri ∩ Ri′| / |Ri ∪ Ri′| when the pathway is present in both organisms, 0 when present in only one, and 1 for shared physical-mechanism pathways. Aggregate into the global functional-level indexes Psim (arithmetic mean of SimPi over all pathways present in either organism) and PsimW (weighted mean by |Ri ∪ Ri′|, which ignores physical-transformation pathways); combine with the structural-level index Ssim (based on shared compounds) into the combined index Csim. For multi-organism studies (e.g. the eight Fungi comparison), compute all pairwise indexes into a similarity matrix, convert to distance = 1 − Csim, and apply complete-linkage hierarchical clustering; a successful application reproduces known evolutionary groupings, e.g. the top-level split separating Yeasts (sce, zro) from Molds (cal, afm, abp, fgr, tpf, tre).

## Related tools

- **MetNet** (Reconstructs the metabolic networks of two KEGG organisms from KGML data and computes the structural (Ssim), pathway (Psim), weighted pathway (PsimW) and combined (Csim) similarity indexes with the 'set' or 'multiset' comparison method.) — github.com/simeoni-biolab/MetNet
- **Java** (Runtime/compiler required to build and execute MetNet from the source jar files (gs-algo-1.3.jar, gs-core-1.3.jar, guava-19.0.jar, etc.).)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every KEGG reference pathway listed in pathwayList.txt must receive a SimPi value (0 if present in only one organism, Jaccard ratio if in both, 1 for shared physical-mechanism pathways).
- The computed global indexes Psim and PsimW must fall within [0,1]; a Jaccard ratio of exactly 1 means identical reaction sets, 0 means disjoint sets.
- For the aphid–Buchnera (apis/bap) case, all indexes should yield very low global similarity (5.02%–14.73%), consistent with complementary rather than similar metabolisms.
- For the eight Fungi experiment, the complete-linkage dendrogram built from distance = 1 − Csim must separate Yeasts (sce, zro) from Molds at the top level, and this grouping should hold for all defined indexes including Ssim.
- Running with 'multiset' instead of 'set' must be reflected in whether reaction multiplicities affect the Jaccard ratios.

## Limitations

- Reconstruction and comparison are based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- MetNet does not currently support input of genomic data; reconstructing from newly sequenced genomes requires a significant extension of the tool.
- The weighted index PsimW ignores pathways dealing with physical transformations, so those pathways contribute only to Psim.
- Data repositories are incomplete, heterogeneous and incoherent, making automatic reconstruction difficult and error-prone.

## Evidence

- [abstract] Two-level similarity indexes: "They allow for both a local comparison, pathway by pathway, and a global comparison of the entire metabolism."
- [readme] MetNet tool scope: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG"
- [methods] Comparison method parameter: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [results] Similarity matrix structure: "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [intro] Definition of similarity measures: "In order to perform the comparison, some similarity measures have been defined."
