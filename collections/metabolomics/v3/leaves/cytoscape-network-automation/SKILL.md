---
name: cytoscape-network-automation
description: Use when an existing Cytoscape network (e.g. a stringApp PPI network
  or a DEG gene list) needs to be extended with curated linkset annotations such as
  compound-target, gene-disease, or gene-pathway associations through reproducible
  scripted automation.
when_to_use_negative:
- The network should be extended with interactions not covered by any available linkset
  resource (WikiPathways, TFLink, ChEMBL, Orphanet, Reactome, OMIM, Wikidata) and
  no custom linkset can be built with the LinksetCreator
- The analysis requires de novo interaction prediction rather than annotation from
  curated resources — CyTargetLinker only parses linksets and extracts interactions
  relevant to the selected network
- Working outside Cytoscape 3.6.0+ / CyTargetLinker 4.0.0+, which lack the automation
  interface the extend command relies on
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: CyTargetLinker
  role: Cytoscape app providing the 'cytargetlinker extend' command that parses linksets
    and extracts relevant interactions for the selected network; also filterOverlap,
    applyVisualstyle, applyLayout
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Desktop network analysis platform (3.6.0+) hosting the automation interface
    through which extend commands are executed
- name: stringApp
  role: Creates the seed PPI network via Disease Query (query='Rett syndrome', cutoff=0.4,
    limit=20) before extension
- name: R
  role: External scripting tool driving cytargetlinker extend via the automation API,
    as in the cytargetlinker-automation repository tutorials
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: LinksetCreator
  role: Generates new XGMML linksets from preprocessed resource data (input.csv) when
    existing linksets do not cover a resource
  repo: https://github.com/CyTargetLinker/linkset-automation
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:fbc323edcce95d331ec858694f359cb8c4fbccf374caa6d96acc74a4e5cf5d01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - LinksetCreator
  - R
  - stringApp
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cytoscape-network-automation

## Summary

Programmatically extend Cytoscape networks with external knowledge using CyTargetLinker 4.0.0+ via the 'cytargetlinker extend' command (from R, Jupyter, or Python), applying curated XGMML linksets (WikiPathways, ChEMBL, OMIM, TFLink, etc.) to add typed interactions to existing networks.

## When to use

Use when an existing Cytoscape network (e.g. a stringApp PPI network or a DEG gene list) needs to be extended with curated linkset annotations such as compound-target, gene-disease, or gene-pathway associations through reproducible scripted automation.

## When NOT to use

- The network should be extended with interactions not covered by any available linkset resource (WikiPathways, TFLink, ChEMBL, Orphanet, Reactome, OMIM, Wikidata) and no custom linkset can be built with the LinksetCreator
- The analysis requires de novo interaction prediction rather than annotation from curated resources — CyTargetLinker only parses linksets and extracts interactions relevant to the selected network
- Working outside Cytoscape 3.6.0+ / CyTargetLinker 4.0.0+, which lack the automation interface the extend command relies on

## Inputs

- Seed Cytoscape network (e.g. stringApp PPI network or DEG node list) with an id attribute such as 'display name' or 'shared name'
- XGMML linkset files (e.g. chembl_23_hsa_20180126.xgmml, omim-rare-disease-has-20180411.xgmml, wikipathways-mm-20180410.xgmml) from cytargetlinker.github.io
- Linkset generation pipeline inputs in the linkset-automation repo (config files, BridgeDb mapper, input.csv, resource data)

## Outputs

- Extended Cytoscape network with typed nodes/edges (e.g. compound-target interactions, gene-disease associations, gene-pathway links)
- Session file (e.g. rett_syndrome_extended_network.cys) or exported XGMML network (e.g. author_article_journal_network.xgmml)
- Validated XGMML linkset passing qc_xgmml.py checks

## How to apply

Build the seed network in Cytoscape 3.6.0+ (e.g. stringApp Disease Query for 'Rett syndrome' with cutoff=0.4 and limit=20) or load a DEG list (e.g. 65 Mecp2-/y genes with |log2FC| > 1) as nodes with idAttribute 'shared name' or 'display name'. Download the appropriate XGMML linksets (e.g. chembl_23_hsa_20180126.xgmml, omim-rare-disease-has-20180411.xgmml, wikipathways-mm-20180410.xgmml) from https://cytargetlinker.github.io and run 'cytargetlinker extend' with idAttribute, linkSetFiles (comma-separated), network=current, and direction (SOURCES for pathway nodes added as targets of gene nodes); repeat the call for multi-step extensions such as the author→article→journal Wikidata example. Optionally apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout). Validate by counting added interactions: e.g. 37 ChEMBL compound-target interactions and 18 OMIM gene-disease associations for the Rett syndrome case, or exactly 16 of 65 DEGs linked to curated mouse WikiPathways.

## Related tools

- **CyTargetLinker** (Cytoscape app providing the 'cytargetlinker extend' command that parses linksets and extracts relevant interactions for the selected network; also filterOverlap, applyVisualstyle, applyLayout) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Desktop network analysis platform (3.6.0+) hosting the automation interface through which extend commands are executed)
- **stringApp** (Creates the seed PPI network via Disease Query (query='Rett syndrome', cutoff=0.4, limit=20) before extension)
- **R** (External scripting tool driving cytargetlinker extend via the automation API, as in the cytargetlinker-automation repository tutorials) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **LinksetCreator** (Generates new XGMML linksets from preprocessed resource data (input.csv) when existing linksets do not cover a resource) — https://github.com/CyTargetLinker/linkset-automation

## Examples

```
cytargetlinker extend idAttribute="display name" linkSetFiles="../Link Sets/chembl_23_hsa_20180126.xgmml,../Link Sets/omim-rare-disease-has-20180411.xgmml" network=current
```

## Evaluation signals

- Extended network contains exactly 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM for the Rett syndrome use case
- Gene-pathway extension shows exactly 16 of the 65 Mecp2-/y DEGs (|log2FC| > 1) present in one or more curated mouse WikiPathways pathways
- Two-step author-article-journal network contains 5 gray author nodes connected via yellow article nodes to green journal nodes matching Figure 3
- qc_xgmml.py exits with code 0 on the linkset: <graph> present, node identifiers/type valid, edge source/target reference existing nodes with datasource/interaction, and BridgeDb mapped fraction meets --min-mapped-frac

## Limitations

- Linkset resources without deposited Zenodo DOIs (ChEMBL, Orphanet, Reactome) may change availability or licensing; upstream licenses (e.g. TFLink CC BY-NC 4.0) constrain reuse
- A failed BridgeDb mapper leaves every node carrying only its input id — structurally valid XGMML but useless for matching a network; check --min-mapped-frac coverage
- Empty XGMML graphs fail qc_xgmml.py by default, and the linkset-automation pipeline is marked 'Under construction'

## Evidence

- [abstract] The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools (e.g. R, Jupyter, Python, etc).: "The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools (e.g. R, Jupyter, Python, etc)."
- [methods] The key function is the "extend" function, which parses the provided linksets and extracts relevant interactions for the selected network.: [evidence span withheld — anchor status: not_found]
- [methods] the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM: "the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM"
- [methods] we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways: "we extracted 65 genes with an absolute log2 fold change larger than 1. Only 16 genes are present in one or more pathways"
- [readme] --min-mapped-frac fails the file below that fraction. This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id: "--min-mapped-frac fails the file below that fraction. This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id"
