---
name: gene-pathway-network-extension
description: Use when you have a list of differentially expressed genes (e.g., selected
  by an absolute log2 fold change cutoff such as > 1) and want to annotate/link them
  to curated pathway resources (WikiPathways) as a network extension in Cytoscape.
when_to_use_negative:
- The goal is to extend a protein-protein interaction network with compound-target
  or disease-gene annotations (Use case 1) rather than gene-pathway associations
- The input is not a gene list from an experimental dataset but, e.g., an author-article-journal
  bibliographic network requiring the two-step extension mechanism
- No curated linkset exists for the organism or identifier type of the gene list (linksets
  cover specific species, e.g., WikiPathways covers 13 species)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0121
tools:
- name: CyTargetLinker
  role: Cytoscape app whose 'cytargetlinker extend' automation function parses the
    WikiPathways linkset and adds pathway nodes as targets of gene nodes
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Desktop platform (3.6.0+) hosting the network, gene loading, visual style
    application, and visualization export
- name: R
  role: External tool used to run the CyTargetLinker automation script programmatically
    for repeatable workflows
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: WikiPathways
  role: Source database of curated pathway annotations used to build the gene-pathway
    linkset (wikipathways-mm-20180410.xgmml)
  repo: https://cytargetlinker.github.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:d6389b8f7838048958c142f05a36809f80e35da96113ed1e7bf5b57b94fb513b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - WikiPathways
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-pathway-network-extension

## Summary

Extends a network of differentially expressed genes with gene-pathway associations from curated WikiPathways linksets using the CyTargetLinker 4.0.0+ automation feature ('cytargetlinker extend') in Cytoscape 3.6.0+, as demonstrated in Use case 2 where 65 Mecp2-/y DEGs (|log2FC| > 1) were linked to curated mouse WikiPathways pathways.

## When to use

Use when you have a list of differentially expressed genes (e.g., selected by an absolute log2 fold change cutoff such as > 1) and want to annotate/link them to curated pathway resources (WikiPathways) as a network extension in Cytoscape.

## When NOT to use

- The goal is to extend a protein-protein interaction network with compound-target or disease-gene annotations (Use case 1) rather than gene-pathway associations
- The input is not a gene list from an experimental dataset but, e.g., an author-article-journal bibliographic network requiring the two-step extension mechanism
- No curated linkset exists for the organism or identifier type of the gene list (linksets cover specific species, e.g., WikiPathways covers 13 species)

## Inputs

- List of differentially expressed genes (e.g., 65 Mecp2-/y vs. wild-type Purkinje-cell DEGs with absolute log2 fold change > 1)
- Curated mouse WikiPathways linkset in XGMML format (e.g., wikipathways-mm-20180410.xgmml)
- Cytoscape 3.6.0+ with CyTargetLinker 4.0.0+ installed

## Outputs

- Gene-pathway network in Cytoscape with pathway nodes (gray rectangles) added as targets of gene nodes
- Count of input genes present in one or more pathways (expected: 16 of 65 DEGs)
- Exported network visualization (e.g., figure2_gene_pathway_network.png)

## How to apply

Load the differentially expressed genes (e.g., 65 Mecp2-/y Purkinje-cell DEGs with absolute log2 fold change larger than 1) as nodes in Cytoscape, coloring them by expression change (blue = down-regulated, red = up-regulated). Run the R automation script calling 'cytargetlinker extend' with idAttribute="shared name", linkSetFiles pointing to the curated mouse WikiPathways XGMML linkset (e.g., wikipathways-mm-20180410.xgmml), network=current, and direction=SOURCES so pathway nodes are added as targets of the gene nodes. Apply the CyTargetLinker visual style so pathways render as gray rectangles, and remove genes without pathway annotations as in the reported workflow. Count how many input genes occur in one or more pathways — the reported result is that only 16 of the 65 DEGs are present in one or more curated mouse WikiPathways pathways. Optionally flag pathways found significantly altered by over-representation analysis (Z-Score > 1.96 in PathVisio) with a green border color and export the network visualization.

## Related tools

- **CyTargetLinker** (Cytoscape app whose 'cytargetlinker extend' automation function parses the WikiPathways linkset and adds pathway nodes as targets of gene nodes) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Desktop platform (3.6.0+) hosting the network, gene loading, visual style application, and visualization export)
- **R** (External tool used to run the CyTargetLinker automation script programmatically for repeatable workflows) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **WikiPathways** (Source database of curated pathway annotations used to build the gene-pathway linkset (wikipathways-mm-20180410.xgmml)) — https://cytargetlinker.github.io/

## Examples

```
cytargetlinker extend idAttribute="shared name" linkSetFiles="../Link Sets/wikipathways-mm-20180410.xgmml" network=current direction=SOURCES
```

## Evaluation signals

- The gene-pathway extension must reproduce the reported result: from 65 DEGs (absolute log2 fold change > 1), exactly 16 genes are present in one or more curated mouse WikiPathways pathways
- Genes are colored by expression change (blue = down-regulated, red = up-regulated) and pathway nodes render as gray rectangles under the CyTargetLinker visual style
- Genes without pathway annotations are removed, and the remaining network's gene nodes all link to at least one pathway node
- Optionally, pathways significant in over-representation analysis (Z-Score > 1.96 in PathVisio) carry a green border color
- The exported figure matches figure2_gene_pathway_network.png from the reported workflow

## Limitations

- Only a minority of DEGs map to curated pathways — 16 of 65 — so many input genes will lack pathway annotations and be removed, leaving sparse coverage
- Results depend on linkset version and BridgeDb identifier mapping; a failed BridgeDb mapper leaves every node carrying only its input id, which is structurally valid but useless for matching a network (qc_xgmml --min-mapped-frac catches this)
- Linksets for some resources (ChEMBL, Orphanet, Reactome) are not yet deposited with persistent DOIs, limiting reproducibility for those extensions

## Evidence

- [methods] Of 65 DEGs (|log2FC| > 1), only 16 genes are present in curated mouse WikiPathways pathways: "we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways"
- [abstract] Workflow loads DEGs and extends with gene-pathway associations: "we created a workflow to load differentially expressed genes from an experimental dataset and extend it with gene-pathway associations"
- [methods] The key function is the extend function: [evidence span withheld — anchor status: not_found]
- [readme] WikiPathways linkset DOI and species coverage: "[WikiPathways](https://www.wikipathways.org) (13 species) | [`wikipathways/`](wikipathways/readme.md) | [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4500957.svg)"
- [readme] BridgeDb mapping failure caveat: "This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id — structurally valid, but useless for matching a network."
