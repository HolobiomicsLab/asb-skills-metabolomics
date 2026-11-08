---
name: differentially-expressed-gene-list-curation
description: Use when you have a ranked differential-expression table (e.g. Mecp2-/y
  vs. wild-type Purkinje cells) and need a filtered DEG gene list extended with pathway
  annotations from a curated linkset such as WikiPathways.
when_to_use_negative:
- The gene list is already annotated with pathway memberships and the goal is only
  visualization, not network extension
- The expression data lack log2 fold-change values, so the |log2FC| > 1 cutoff cannot
  be applied to define DEGs
- The target species has no curated WikiPathways linkset available for the gene identifiers
  in the dataset
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: CyTargetLinker
  role: Cytoscape app whose 'cytargetlinker extend' automation function parses the
    WikiPathways linkset and adds pathway nodes as targets of the DEG gene nodes
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Desktop platform (3.6.0+) hosting the DEG network, the visual style application,
    and the CyTargetLinker automation interface
- name: R
  role: External scripting environment used to run the automation script that invokes
    cytargetlinker extend programmatically
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: WikiPathways
  role: Source of the curated mouse gene-pathway associations packaged as the XGMML
    linkset used for extension
  repo: https://cytargetlinker.github.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:eebc37e3c69b6408a58a1eab9a1275eeed2bd9182240ed3cab25c7081c54ea52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - WikiPathways
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differentially-expressed-gene-list-curation

## Summary

Curate a list of differentially expressed genes from an expression dataset using an absolute log2 fold-change cutoff (|log2FC| > 1), then extend the list with gene-pathway associations from curated WikiPathways linksets via CyTargetLinker's 'cytargetlinker extend' automation in Cytoscape.

## When to use

Use when you have a ranked differential-expression table (e.g. Mecp2-/y vs. wild-type Purkinje cells) and need a filtered DEG gene list extended with pathway annotations from a curated linkset such as WikiPathways.

## When NOT to use

- The gene list is already annotated with pathway memberships and the goal is only visualization, not network extension
- The expression data lack log2 fold-change values, so the |log2FC| > 1 cutoff cannot be applied to define DEGs
- The target species has no curated WikiPathways linkset available for the gene identifiers in the dataset

## Inputs

- Differential-expression gene table (gene identifiers, log2 fold change, up/down direction) from Mecp2-/y vs. wild-type Purkinje-cell dataset
- Curated mouse WikiPathways linkset in XGMML format (wikipathways-mm-20180410.xgmml)
- Cytoscape current network containing the 65 DEG nodes

## Outputs

- Filtered DEG gene list (65 genes with |log2FC| > 1) loaded as a Cytoscape network
- Extended gene-pathway network with pathway nodes added as targets
- Count of DEGs present in one or more pathways (16 of 65)
- Exported network visualization (figure2_gene_pathway_network.png)

## How to apply

Load the differentially expressed genes from the experimental dataset as nodes in Cytoscape, coloring them by expression direction (blue = down-regulated, red = up-regulated), and extract genes passing the absolute log2 fold change larger than 1 threshold (yielding 65 genes in the Mecp2-/y Purkinje-cell use case). Run the R automation script calling 'cytargetlinker extend' with idAttribute="shared name", linkSetFiles pointing to the WikiPathways XGMML linkset (e.g. ../Link Sets/wikipathways-mm-20180410.xgmml), network=current, and direction=SOURCES so pathway nodes are added as targets of the gene nodes. Apply the CyTargetLinker visual style so pathways render as gray rectangles, and remove genes without pathway annotations as in the reported workflow. Optionally, flag pathways found significantly altered by over-representation analysis (Z-Score > 1.96 in PathVisio) with a green border. The rationale is that curated mouse WikiPathways coverage is sparse, so counting linked genes (16 of 65) quantifies pathway annotation coverage of the DEG set.

## Related tools

- **CyTargetLinker** (Cytoscape app whose 'cytargetlinker extend' automation function parses the WikiPathways linkset and adds pathway nodes as targets of the DEG gene nodes) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Desktop platform (3.6.0+) hosting the DEG network, the visual style application, and the CyTargetLinker automation interface)
- **R** (External scripting environment used to run the automation script that invokes cytargetlinker extend programmatically) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **WikiPathways** (Source of the curated mouse gene-pathway associations packaged as the XGMML linkset used for extension) — https://cytargetlinker.github.io/

## Examples

```
cytargetlinker extend idAttribute="shared name" linkSetFiles="../Link Sets/wikipathways-mm-20180410.xgmml" network=current direction=SOURCES
```

## Evaluation signals

- The filtered DEG list contains exactly 65 genes when the |log2FC| > 1 cutoff is applied to the Mecp2-/y Purkinje-cell dataset
- The extension result reports that only 16 of the 65 input genes are present in one or more curated mouse WikiPathways pathways
- Pathway nodes render as gray rectangles under the CyTargetLinker visual style and appear as targets (direction=SOURCES) of the gene nodes
- The XGMML linkset passes the qc_xgmml.py structural validation (well-formed graph, nodes with identifiers list + type, edges referencing existing nodes with datasource/interaction)
- The exported network visualization matches the reported gene-pathway extension figure for Use case 2

## Limitations

- Only a minority of DEGs (16 of 65) map to curated mouse WikiPathways pathways, so many filtered genes will be removed as unannotated
- The QC script fails empty graphs by default, and a failed BridgeDb mapper leaves every node carrying only its input id — structurally valid but useless for matching a network
- WikiPathways linkset coverage is limited to 13 species; resources like ChEMBL, Orphanet, and Reactome linksets were not yet deposited at the time
- Requires CyTargetLinker 4.0.0+ on Cytoscape 3.6.0+ for the automation interface

## Evidence

- [methods] Of 65 DEGs (|log2FC| > 1), only 16 genes are present in curated mouse WikiPathways pathways: "we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways"
- [abstract] Load differentially expressed genes from an experimental dataset and extend with gene-pathway associations: "we created a workflow to load differentially expressed genes from an experimental dataset and extend it with gene-pathway associations"
- [methods] The extend function parses the provided linksets and extracts relevant interactions: [evidence span withheld — anchor status: not_found]
- [readme] WikiPathways linkset automated build and QC in the linkset-automation repository: "This repo automatically generates CyTargetLinker linksets from different resources."
