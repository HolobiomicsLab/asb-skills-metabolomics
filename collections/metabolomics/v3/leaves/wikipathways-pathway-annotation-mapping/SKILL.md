---
name: wikipathways-pathway-annotation-mapping
description: Use when a list of differentially expressed genes (e.g. |log2FC| > 1)
  must be linked to curated pathway annotations by extending a Cytoscape network with
  a WikiPathways linkset via CyTargetLinker.
when_to_use_negative:
- The goal is compound-target or disease-gene extension of a PPI network (Use case
  1) rather than gene-to-pathway annotation with a WikiPathways linkset
- The input network nodes are author/journal entities for the two-step author-article-journal
  extension (Use case 3)
- Only pathways from resources other than curated WikiPathways (e.g. Reactome, OMIM)
  are relevant, since the reported 16/65 result is specific to the curated mouse WikiPathways
  collection
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CyTargetLinker
  role: 'Performs the network extension: its ''cytargetlinker extend'' command parses
    the WikiPathways linkset and adds pathway nodes as targets of the gene nodes;
    version 4.0.0+'
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: Cytoscape
  role: Platform hosting the network, the CyTargetLinker app, and the visual style;
    version 3.6.0+ required for the automation interface
- name: R
  role: External scripting environment used to drive the automated 'cytargetlinker
    extend' workflow
- name: WikiPathways
  role: Source database of the curated mouse pathway annotations distributed as an
    XGMML linkset (Zenodo DOI 10.5281/zenodo.4500957)
  repo: https://www.wikipathways.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:19065407e66c2bcc73897b36a130546da6d3f7269052474c3351eb7e1a0462ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker-automation
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - WikiPathways
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# wikipathways-pathway-annotation-mapping

## Summary

Annotates differentially expressed genes with curated WikiPathways pathway membership by extending a gene network in Cytoscape with the CyTargetLinker 'extend' command and a WikiPathways XGMML linkset, as demonstrated on the Mecp2-/y Purkinje-cell DEG dataset (Use case 2).

## When to use

Use when a list of differentially expressed genes (e.g. |log2FC| > 1) must be linked to curated pathway annotations by extending a Cytoscape network with a WikiPathways linkset via CyTargetLinker.

## When NOT to use

- The goal is compound-target or disease-gene extension of a PPI network (Use case 1) rather than gene-to-pathway annotation with a WikiPathways linkset
- The input network nodes are author/journal entities for the two-step author-article-journal extension (Use case 3)
- Only pathways from resources other than curated WikiPathways (e.g. Reactome, OMIM) are relevant, since the reported 16/65 result is specific to the curated mouse WikiPathways collection

## Inputs

- Differentially expressed gene list (65 Mecp2-/y Purkinje-cell DEGs with |log2 fold change| > 1)
- Curated mouse WikiPathways XGMML linkset (wikipathways-mm-20180410.xgmml)
- Cytoscape network with gene nodes keyed on 'shared name'

## Outputs

- Extended gene-pathway network in Cytoscape (pathways as gray rectangle nodes connected to annotated genes)
- Count of input DEGs present in one or more WikiPathways pathways (expected: 16 of 65)
- Exported network visualization (e.g. figure2_gene_pathway_network.png)

## How to apply

Load the DEGs (65 genes with absolute log2 fold change > 1 in the Mecp2-/y vs. wild-type Purkinje-cell dataset) as nodes in Cytoscape, coloring blue = down-regulated and red = up-regulated. Run the R automation script calling 'cytargetlinker extend' with idAttribute='shared name', linkSetFiles pointing at the mouse WikiPathways linkset (wikipathways-mm-20180410.xgmml), network=current, and direction=SOURCES so pathway nodes are added as targets of gene nodes. Apply the CyTargetLinker visual style so pathways render as gray rectangles and remove genes lacking pathway annotations. Count input genes present in one or more curated mouse WikiPathways pathways; the reported result is 16 of 65. Optionally flag pathways with over-representation Z-Score > 1.96 (PathVisio) with a green border, and validate the linkset beforehand with scripts/qc_xgmml.py to catch failed BridgeDb cross-reference mapping.

## Related tools

- **CyTargetLinker** (Performs the network extension: its 'cytargetlinker extend' command parses the WikiPathways linkset and adds pathway nodes as targets of the gene nodes; version 4.0.0+) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **Cytoscape** (Platform hosting the network, the CyTargetLinker app, and the visual style; version 3.6.0+ required for the automation interface)
- **R** (External scripting environment used to drive the automated 'cytargetlinker extend' workflow)
- **WikiPathways** (Source database of the curated mouse pathway annotations distributed as an XGMML linkset (Zenodo DOI 10.5281/zenodo.4500957)) — https://www.wikipathways.org

## Examples

```
cytargetlinker extend idAttribute="shared name" linkSetFiles="../Link Sets/wikipathways-mm-20180410.xgmml" network=current direction=SOURCES
```

## Evaluation signals

- The count of input DEGs with one or more WikiPathways pathway annotations equals 16 out of the 65 DEGs with |log2FC| > 1
- Pathway nodes appear only as targets (direction=SOURCES) of gene nodes and render as gray rectangles under the CyTargetLinker visual style
- Genes without any pathway annotation are removed from the final network
- The linkset XGMML passes scripts/qc_xgmml.py (well-formed graph, resolvable edge sources/targets, adequate BridgeDb cross-reference coverage)
- Optionally, pathways with over-representation Z-Score > 1.96 (PathVisio) carry a green border

## Limitations

- Only curated mouse WikiPathways content is used, so many DEGs (49 of 65) remain unannotated — the result is resource-specific
- Requires Cytoscape 3.6.0+ and CyTargetLinker 4.0.0+ for the automation interface; older versions lack the 'cytargetlinker extend' command
- A failed BridgeDb mapper leaves linkset nodes with only their input identifier, producing structurally valid but useless linksets — hence the qc_xgmml.py --min-mapped-frac check
- Linkset generation for some resources is still under construction in the linkset-automation repo; ensure the WikiPathways linkset version is current

## Evidence

- [methods] Of 65 DEGs (|log2FC| > 1), only 16 genes are present in curated mouse WikiPathways pathways: "we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways"
- [abstract] Workflow of Use case 2: load DEGs and extend with gene-pathway associations: "we created a workflow to load differentially expressed genes from an experimental dataset and extend it with gene-pathway associations"
- [methods] CyTargetLinker adds pathway information from the WikiPathways database: "the pathway annotations from the Wiki Pathways database14, Cy Target Linker adds the pathway information"
- [methods] Automation tool versions required: "The newest version of Cy Target Linker (4.0.0+) was developed for Cytoscape (3.6.0+)"
- [readme] Linkset QC guards against failed identifier mapping: "This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id"
