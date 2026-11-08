---
name: xgmml-linkset-interaction-mapping
description: Use when a Cytoscape network (gene list, PPI, author-article graph) must
  be extended with curated external interactions stored as XGMML linksets, and the
  goal is to attach e.g. pathway, TF-target, or compound-target links to current network
  nodes.
when_to_use_negative:
- The input data is raw experimental output (e.g. a differential-expression table)
  that has not yet been filtered and loaded as network nodes — the DEGs must first
  be extracted with the |log2FC| > 1 cutoff
- You only need to visualise interactions from a single non-linkset source embedded
  directly in Cytoscape rather than merging curated XGMML linksets
- The linkset's identifier types cannot be mapped to the network's node identifiers
  (BridgeDb mapped fraction near zero), since 'extend' would extract no matching interactions
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CyTargetLinker
  role: Cytoscape app whose 'extend' function parses XGMML linksets and extracts relevant
    interactions for the selected network; version 4.0.0+ provides the automation
    command line
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Host application (3.6.0+) providing the network model, visual style application,
    and automation interface used by CyTargetLinker
- name: R
  role: External tool used to drive CyTargetLinker's automation feature programmatically
    (cytargetlinker extend calls) for repeatable workflows
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: qc_xgmml.py
  role: Quality-control script validating that an XGMML linkset is well-formed and
    reports BridgeDb cross-reference coverage before extension
  repo: https://github.com/CyTargetLinker/linkset-automation
- name: WikiPathways
  role: Curated pathway resource (13 species) supplying the gene-pathway XGMML linkset
    used in the DEG extension use case
  repo: https://github.com/CyTargetLinker/linkset-automation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:da20d07290912b4096c38bdc708c41829bd590816167743a0d967e9c9ef468c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - WikiPathways
  - qc_xgmml.py
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# xgmml-linkset-interaction-mapping

## Summary

Extends a Cytoscape network with prior-knowledge interactions by parsing XGMML linksets (e.g. WikiPathways, TFLink, ChEMBL) through CyTargetLinker's 'extend' function, so external relationships are mapped onto existing nodes and edges.

## When to use

Use when a Cytoscape network (gene list, PPI, author-article graph) must be extended with curated external interactions stored as XGMML linksets, and the goal is to attach e.g. pathway, TF-target, or compound-target links to current network nodes.

## When NOT to use

- The input data is raw experimental output (e.g. a differential-expression table) that has not yet been filtered and loaded as network nodes — the DEGs must first be extracted with the |log2FC| > 1 cutoff
- You only need to visualise interactions from a single non-linkset source embedded directly in Cytoscape rather than merging curated XGMML linksets
- The linkset's identifier types cannot be mapped to the network's node identifiers (BridgeDb mapped fraction near zero), since 'extend' would extract no matching interactions

## Inputs

- Cytoscape network (gene list, stringApp PPI, or author-article network)
- XGMML linkset file (e.g. wikipathways-mm-20180410.xgmml, chembl_23_hsa_20180126.xgmml)
- Gene identifier attribute in the network (e.g. 'shared name' or 'display name')
- BridgeDb mapping cache matching the linkset's identifier types

## Outputs

- Extended Cytoscape network containing original nodes plus linkset-derived nodes and interactions
- Gene-pathway network visualization (e.g. figure2_gene_pathway_network.png)
- QC report from qc_xgmml.py including BridgeDb mapped-type coverage fraction

## How to apply

Build or obtain an XGMML linkset from the resource of interest (e.g. wikipathways-mm-20180410.xgmml for mouse WikiPathways) and validate it with the qc_xgmml.py script so the graph structure, node 'identifiers' lists, and edge datasource/interaction attributes are well-formed. Load the input network in Cytoscape 3.6.0+ with CyTargetLinker 4.0.0+ and call the 'extend' function, which parses the provided linksets and extracts relevant interactions for the selected network, setting idAttribute (e.g. 'shared name'), network=current, and direction (e.g. SOURCES to add pathway nodes as targets of gene nodes). Identifier matching relies on BridgeDb cross-reference mapping, so check the mapped fraction (e.g. --min-mapped-frac 0.9 for gene nodes) before extending. Apply the CyTargetLinker visual style so new node types render distinctly (e.g. pathways as gray rectangles), then use filterOverlap if only multi-resource-supported interactions should be shown. For the Mecp2-/y Purkinje-cell use case, the expected outcome is that 16 of the 65 DEGs (|log2 fold change| > 1) map to one or more curated mouse WikiPathways pathways.

## Related tools

- **CyTargetLinker** (Cytoscape app whose 'extend' function parses XGMML linksets and extracts relevant interactions for the selected network; version 4.0.0+ provides the automation command line) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Host application (3.6.0+) providing the network model, visual style application, and automation interface used by CyTargetLinker)
- **R** (External tool used to drive CyTargetLinker's automation feature programmatically (cytargetlinker extend calls) for repeatable workflows) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **qc_xgmml.py** (Quality-control script validating that an XGMML linkset is well-formed and reports BridgeDb cross-reference coverage before extension) — https://github.com/CyTargetLinker/linkset-automation
- **WikiPathways** (Curated pathway resource (13 species) supplying the gene-pathway XGMML linkset used in the DEG extension use case) — https://github.com/CyTargetLinker/linkset-automation

## Examples

```
cytargetlinker extend idAttribute="shared name" linkSetFiles="../Link Sets/wikipathways-mm-20180410.xgmml" network=current direction=SOURCES
```

## Evaluation signals

- qc_xgmml.py exits non-zero unless the XGMML file contains a <graph>, nodes with 'identifiers' lists + 'type', and edges whose source/target reference existing nodes with 'datasource'/'interaction' attributes; empty graphs fail by default
- BridgeDb coverage check: --mapped-type gene --min-mapped-frac 0.9 must pass, since a failed mapper leaves every node with only its input id
- Use-case 2 invariant: from 65 DEGs with absolute log2 fold change larger than 1, exactly 16 genes must be present in one or more curated mouse WikiPathways pathways
- Visual style check: pathway nodes render as gray rectangles and genes are colored by expression (blue = down-regulated, red = up-regulated)
- filterOverlap verification: restricting to interactions supported by multiple resources reduces the network to only multi-resource-supported edges

## Limitations

- Linkset quality depends on the BridgeDb cache: if the mapper fails to load, every node carries only its input id, which is structurally valid but useless for matching a network
- Resource coverage is incomplete — some linksets (ChEMBL, Orphanet, Reactome) were not yet deposited on Zenodo at the time of writing
- Extension only matches identifiers present in the linkset; as Use case 2 shows, most DEGs (49 of 65) may map to no curated pathway at all

## Evidence

- [methods] The key function is the "extend" function, which parses the provided linksets and extracts relevant interactions for the selected network.: [evidence span withheld — anchor status: not_found]
- [methods] Only 16 genes are present in one or more pathways: "we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways"
- [readme] qc_xgmml.py validates XGMML linkset structure: "`scripts/qc_xgmml.py` validates that an XGMML linkset is well-formed and has the structure every CyTargetLinker linkset carries"
- [readme] BridgeDb mapped-fraction check catches failed mappers: "This catches a BridgeDb mapper that failed to load, which leaves every node carrying
only its input id — structurally valid, but useless for matching a network."
- [intro] linksets renamed from RegINs to broaden applicability: "we renamed the previously used Regulatory Interaction Networks (RegINs) to linksets to make the broader applicability more explicit."
