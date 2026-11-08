---
name: ubiquitous-compound-filtering
description: 'Use when building pathway adjacency from KEGG KGML compound sets: compounds
  shared by many pathways (water, ATP, phosphate) would otherwise connect nearly every
  node and must be excluded before edge creation.'
when_to_use_negative:
- When building the functional level of MetNet (reaction-content representation per
  pathway), which relies on reactions rather than compound-sharing edges.
- When the downstream analysis depends on cofactor-mediated connectivity itself (e.g.
  energy-transfer analysis) rather than discriminative pathway topology.
- When working with compound sets from sources other than KEGG reference pathways
  whose ubiquitous-compound semantics (H2O, phosphate, ATP, ADP) may differ.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3407
tools:
- name: MetNet
  role: Implements the structural-level reconstruction that applies the ubiquitous-compound
    filter when building pathway–pathway edges
  repo: https://github.com/simeoni-biolab/MetNet
- name: sax2r2
  role: Java library used by MetNet to parse KGML files and extract per-pathway compounds
    prior to filtering
- name: Apache POI
  role: Java library (poi-3.9.jar) on MetNet's classpath used for KGML/parsing support
    during reconstruction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:e2f78cc0d5c5ba72d60f9c4a2ef321c54eae93be6a755757543bf993bb6e7ccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Apache POI
  - Java
  - MetNet
  - gs-algo-1.3.jar
  - gs-core-1.3.jar
  - guava-19.0.jar
  - poi-3.9.jar
  - sax2r2
  - sax2r2.jar
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# ubiquitous-compound-filtering

## Summary

Removes ubiquitous metabolites (e.g. H2O, phosphate, ATP, ADP) when deriving pathway–pathway relations from KEGG KGML compound sets, so that shared-compound edges in MetNet's structural level reflect meaningful metabolic overlap rather than universal cofactors.

## When to use

Use when building pathway adjacency from KEGG KGML compound sets: compounds shared by many pathways (water, ATP, phosphate) would otherwise connect nearly every node and must be excluded before edge creation.

## When NOT to use

- When building the functional level of MetNet (reaction-content representation per pathway), which relies on reactions rather than compound-sharing edges.
- When the downstream analysis depends on cofactor-mediated connectivity itself (e.g. energy-transfer analysis) rather than discriminative pathway topology.
- When working with compound sets from sources other than KEGG reference pathways whose ubiquitous-compound semantics (H2O, phosphate, ATP, ADP) may differ.

## Inputs

- KGML files of the organism's pathways, parsed into per-pathway compound sets (Java KGML parsing via sax2r2 / Apache POI)
- List of ubiquitous compounds to discard: H2O (water), phosphate, ATP, ADP

## Outputs

- Filtered per-pathway compound sets with ubiquitous compounds removed
- Pathway–pathway edges based on shared non-ubiquitous compounds (feed into the symmetric adjacency matrix / structural_graph.csv, cells 0/1)

## How to apply

Parse the organism's KGML files (retrieved via the public KEGG APIs) to extract the compound set of each pathway. Before pairwise comparison, remove the ubiquitous compounds — MetNet treats water (H2O), phosphate, ATP and ADP as ubiquitous and discards them from the compound sets. Then compare the filtered compound sets of each pair of pathways: an edge between two pathway nodes is created only if they share at least one remaining (non-ubiquitous) compound. The rationale is that universal cofactors and solvents co-occur in essentially all pathways, so leaving them in would produce a near-complete graph and destroy the discriminative power of the structural adjacency matrix (structural_graph.csv, cells 0/1 for absence/presence of a shared compound edge).

## Related tools

- **MetNet** (Implements the structural-level reconstruction that applies the ubiquitous-compound filter when building pathway–pathway edges) — https://github.com/simeoni-biolab/MetNet
- **sax2r2** (Java library used by MetNet to parse KGML files and extract per-pathway compounds prior to filtering)
- **Apache POI** (Java library (poi-3.9.jar) on MetNet's classpath used for KGML/parsing support during reconstruction)

## Examples

```
java -cp ./lib/gs-core-1.3.jar:./lib/gs-algo-1.3.jar:./lib/guava-19.0.jar:./lib/poi-3.9.jar:./lib/sax2r2.jar -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every edge in the resulting adjacency matrix corresponds to at least one shared non-ubiquitous compound between the two pathway compound sets.
- No edge in structural_graph.csv can be traced back to H2O, phosphate, ATP or ADP alone — removing these compounds from both pathway sets should drop the edge.
- The filtered compound set of each pathway excludes the four ubiquitous compounds before pairwise comparison.
- The structural graph contains edges only for pathways actually present in the organism (paths listed in pathwayList.txt / organism's KGML files).

## Limitations

- The list of ubiquitous compounds (H2O, phosphate, ATP, ADP) is fixed in MetNet; other broadly-shared metabolites (e.g. CO2, NADH) are not filtered and may still create near-universal edges.
- The filter is tied to MetNet's KEGG-based reconstruction; applying it to other data sources requires re-deriving which compounds are ubiquitous.
- The structural level ignores KEGG maplinks, which are visual-only and unreliable, so edge semantics depend entirely on the filtered shared-compound rule.

## Evidence

- [other] Ubiquitous compounds are removed before pathway-pairwise comparison: [evidence span withheld — anchor status: not_found]
- [other] Structural level encoded as adjacency over pathways with shared non-ubiquitous compounds: [evidence span withheld — anchor status: not_found]
- [methods] MetNet relies on KEGG KGML files retrieved via public APIs: [evidence span withheld — anchor status: not_found]
- [methods] KGML parsing extracts compounds and reactions for the representation: "to parse each KGML file to extract the relevant information for our representation, which are the compounds and the reactions of each pathway"
- [readme] Tool repository and compile-time dependencies confirmed in README: "MetNet can be compiled using the Java compiler from a terminal window."
