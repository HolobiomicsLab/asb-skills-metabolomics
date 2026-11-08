---
name: weighted-pathway-similarity-scoring
description: Use when two KEGG organisms' metabolic networks have been reconstructed
  at the structural and functional levels and you need quantitative global similarity
  scores (Psim, PsimW, Csim) per organism pair.
when_to_use_negative:
- Input is raw genomic or experimental data rather than KEGG-annotated organisms —
  MetNet's reconstruction is based uniquely on the KEGG database and does not yet
  accept genomic data input
- Only a local, pathway-by-pathway comparison is needed (SimPi) without global aggregation
  into Psim/PsimW/Csim
- The organisms of interest are not represented in the KEGG organism list downloaded
  into organismList.txt
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: MetNet
  role: Java tool that reconstructs the two organisms' KEGG metabolic networks and
    computes the similarity indexes Psim, PsimW and Csim for their comparison
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML pathway files and reference pathways from which reaction
    sets for the Jaccard-based SimPi are extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:e377571ed0cae0c1ddc10464f28096eb2737cae89d976b80af0dfe5e53111717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# weighted-pathway-similarity-scoring

## Summary

Computes the global metabolic similarity indexes Psim, PsimW (weighted by pathway reaction-set size) and the combined index Csim for a pair of KEGG organisms, aggregating the local Jaccard-based pathway similarity SimPi produced by MetNet's two-level network comparison.

## When to use

Use when two KEGG organisms' metabolic networks have been reconstructed at the structural and functional levels and you need quantitative global similarity scores (Psim, PsimW, Csim) per organism pair.

## When NOT to use

- Input is raw genomic or experimental data rather than KEGG-annotated organisms — MetNet's reconstruction is based uniquely on the KEGG database and does not yet accept genomic data input
- Only a local, pathway-by-pathway comparison is needed (SimPi) without global aggregation into Psim/PsimW/Csim
- The organisms of interest are not represented in the KEGG organism list downloaded into organismList.txt

## Inputs

- KGML pathway files of two organisms downloaded via KEGG's public APIs
- Two-level metabolic network reconstructions (structural graph plus per-pathway reaction sets) in 'set' or 'multiset' representation
- Configuration file pathwayList.txt listing the KEGG reference pathways to consider
- Configuration file organismList.txt with the list of available KEGG organisms

## Outputs

- Global similarity indexes Psim, PsimW and Csim per organism pair
- Summary table / matrix whose rows and columns are the compared KEGG reference pathways' similarity values

## How to apply

After MetNet reconstructs both organisms' networks from their KGML files (parsed for compounds and reactions per pathway), compute the local similarity SimPi for each reference pathway as the Jaccard index over the pathways' reaction sets, assigning the boundary values 0 or 1 for physical-pathway cases. Aggregate into Psim as the arithmetic mean of SimPi over the n KEGG pathways present in either organism, and into PsimW as a weighted mean of SimPi with weights |Ri ∪ Ri′| over reaction pathways only, so larger pathways dominate. Compute the structure similarity Ssim as the arithmetic mean of the structural indexes SimSi over the |V ∪ V′| nodes, then form the combined index Csim = Σ SimSi·SimPi/n; because both factors lie in [0,1], their product amplifies differences between the organisms. Write the three global indexes (Psim, PsimW, Csim) to a summary table, typically as rows/columns of a matrix over all KEGG reference pathways listed in pathwayList.txt.

## Related tools

- **MetNet** (Java tool that reconstructs the two organisms' KEGG metabolic networks and computes the similarity indexes Psim, PsimW and Csim for their comparison) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML pathway files and reference pathways from which reaction sets for the Jaccard-based SimPi are extracted)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Psim equals the arithmetic mean of the per-pathway SimPi values over the n KEGG pathways present in either organism
- PsimW equals the weighted mean of SimPi with weights |Ri ∪ Ri′|, computed over reaction pathways only (physical-pathway cases excluded)
- All three indexes Psim, PsimW and Csim fall in [0,1]; Csim should be smaller than or comparable to the component means since the product of two [0,1] factors amplifies differences
- Boundary cases where SimPi is exactly 0 or 1 (physical-pathway cases) are handled without distorting the aggregates
- The summary table contains one entry per pair of organisms for each of the three indexes

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- MetNet does not currently support input of genomic data; reconstruction from newly annotated genomes requires tool extension
- The comparison is pairwise — one invocation handles a single pair of KEGG organisms; set-level comparisons require multiple runs

## Evidence

- [abstract] They allow for both a local comparison, pathway by pathway, and a global comparison of the entire: "They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [methods] to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [results] whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file): "whose rows and columns represent all the KEGG reference pathways considered for the comparison (i.e. the ones listed in the configuration file)"
- [readme] MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks"
