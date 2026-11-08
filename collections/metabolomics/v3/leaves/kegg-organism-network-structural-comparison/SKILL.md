---
name: kegg-organism-network-structural-comparison
description: Use when you have reconstructed the two-level metabolic networks of two
  KEGG organisms and need to compare their structural level — pathway connectivity
  through shared non-ubiquitous compounds — pathway by pathway or globally.
when_to_use_negative:
- If the goal is to compare pathway reaction content rather than pathway connectivity
  — that requires the functional-level index SimPi with 'set' or 'multiset' comparison
  mode.
- If the organism's metabolic network cannot be reconstructed from KEGG alone (e.g.
  input is raw genomic data); MetNet does not currently support input of genomic data
  and the article notes this needs a significant tool extension.
- 'If you need cross-database comparison: the reconstruction is based uniquely on
  the KEGG database, which the discussion flags as a limit with respect to the represented
  information.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: MetNet
  role: Java tool that reconstructs the two-level metabolic networks from KEGG and
    computes the structural similarity index SimSi between the two organisms' pathway
    graphs, both quantitatively and visually
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of KGML files and reference pathway definitions from which the pathway
    graphs (nodes = pathways, edges = shared non-ubiquitous compounds) are reconstructed
- name: guava-19.0.jar
  role: Library dependency in the MetNet compilation classpath used to build and run
    the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:df6902832bd29712ec53a3cfe7d8989180ece27074532a7f1714a8f0dabe7ad4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# kegg-organism-network-structural-comparison

## Summary

Compute the structural-level similarity index SimSi for pairs of KEGG organisms by comparing pathway graphs G and G' built from KGML-derived compound-sharing relations, producing a per-reference-pathway table of local similarities that underpins MetNet's metabolism-wide structural comparison.

## When to use

Use when you have reconstructed the two-level metabolic networks of two KEGG organisms and need to compare their structural level — pathway connectivity through shared non-ubiquitous compounds — pathway by pathway or globally.

## When NOT to use

- If the goal is to compare pathway reaction content rather than pathway connectivity — that requires the functional-level index SimPi with 'set' or 'multiset' comparison mode.
- If the organism's metabolic network cannot be reconstructed from KEGG alone (e.g. input is raw genomic data); MetNet does not currently support input of genomic data and the article notes this needs a significant tool extension.
- If you need cross-database comparison: the reconstruction is based uniquely on the KEGG database, which the discussion flags as a limit with respect to the represented information.

## Inputs

- Reconstructed metabolic networks (graph-based level) of two KEGG organisms, built from parsed KGML files listing compounds and reactions per pathway
- KEGG organism codes for the two organisms (e.g. hsa, ptr)
- pathwayList.txt configuration file listing the KEGG reference pathways to consider

## Outputs

- Per-reference-pathway table of structural similarity values SimSi in [0,1]
- Visual/quantitative comparison of the two network structures highlighting similarities and differences at the structural level
- Aggregate global structural similarity over the entire metabolism (as assembled from local values)

## How to apply

After MetNet reconstructs each organism's network from the KGML files downloaded via KEGG's public APIs (compounds and reactions parsed per pathway), compare the two graphs at the structural level where pathways are nodes and edges are relations between pathways through shared non-ubiquitous compounds. For each KEGG reference pathway Pi (iterating over pathwayList.txt), apply the structural index SimSi on the pathway graphs G and G': set SimSi=0 if the node is present in only one graph, SimSi=1 if the node is isolated in both, SimSi=1/(1+k) if the node is isolated in one graph but connected with degree k in the other, and SimSi=|Ei∩Ei'|/|Ei∪Ei'| if the node is connected in both graphs. Record these per-pathway values so they can be aggregated into global structural indexes or rendered visually on the network. Verify every computed SimSi falls in [0,1] and that all reference pathways present in either organism are covered.

## Related tools

- **MetNet** (Java tool that reconstructs the two-level metabolic networks from KEGG and computes the structural similarity index SimSi between the two organisms' pathway graphs, both quantitatively and visually) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of KGML files and reference pathway definitions from which the pathway graphs (nodes = pathways, edges = shared non-ubiquitous compounds) are reconstructed)
- **guava-19.0.jar** (Library dependency in the MetNet compilation classpath used to build and run the comparison)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every per-pathway SimSi value lies in the interval [0,1] as required by the index definition.
- The output table includes one row for every KEGG reference pathway present in either organism O or O' (as listed in pathwayList.txt).
- SimSi equals exactly 1 for reference pathways that are isolated nodes in both graphs, and exactly 0 for pathways present in only one organism.
- For pathways connected in both graphs, SimSi equals the fraction |Ei∩Ei'|/|Ei∪Ei'|; spot-check the edge sets against the KGML-derived compound-sharing relations.
- For nodes connected in one graph and isolated in the other, SimSi = 1/(1+k) where k is the degree in the connected graph — recompute from the adjacency of the reconstructed graph to confirm.

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information.
- MetNet does not yet support input of genomic data; it would have to be significantly extended to reconstruct the corresponding metabolic network from newly annotated genomes.
- The comparison is defined only at the level of the KEGG reference pathways in pathwayList.txt; pathways outside that list are not considered.

## Evidence

- [methods] structural index definition over pathway graphs: [evidence span withheld — anchor status: not_found]
- [abstract] two-level representation rationale: "the first level is graph-based and depicts pathways as nodes and relations between pathways as edges; the second level"
- [readme] MetNet role in structural comparison: "MetNet is a Java tool that makes it possible to automatically reconstruct the metabolic network of two organisms selected in KEGG and to compare their two networks both quantitatively and visually."
- [abstract] similarity indexes for both levels: "we introduce some similarity measures for both levels. They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [methods] validation range constraint: [evidence span withheld — anchor status: not_found]
