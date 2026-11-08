---
name: gene-disease-association-annotation
description: Use when a Cytoscape network of genes/proteins (e.g. built with a stringApp
  Disease Query such as query='Rett syndrome', cutoff=0.4, limit=20) needs disease-gene
  and compound-target annotations added from CyTargetLinker XGMML linksets.
when_to_use_negative:
- The input is a differentially expressed gene list for pathway extension — use the
  gene-pathway (WikiPathways) linkset use case instead
- You need author-article-journal relationships from Wikidata rather than compound-target
  or disease-gene annotations
- The network nodes cannot be matched by the 'display name' attribute the linksets
  join on
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CyTargetLinker
  role: Runs the 'cytargetlinker extend' automation function that parses the linksets
    and extracts relevant interactions for the selected network
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: stringApp
  role: Creates the base Rett syndrome PPI network via Disease Query (query='Rett
    syndrome', cutoff=0.4, limit=20)
- name: Cytoscape
  role: Host application (3.6.0+) providing the automation interface through which
    the extend, applyVisualstyle, and applyLayout commands run
- name: R
  role: External tool optionally used to drive the CyTargetLinker automation commands
    for repeatable workflows
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:e6444077edfdbe5f1b0c910f779af649d20e5bd82b9b5d33b822e9f2bc158bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - stringApp
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-disease-association-annotation

## Summary

Annotates a Cytoscape protein-protein interaction network (e.g. a stringApp Disease Query network) with gene-disease associations from a curated OMIM rare-disease linkset, and optionally compound-target interactions from ChEMBL, using the CyTargetLinker 'extend' automation command.

## When to use

Use when a Cytoscape network of genes/proteins (e.g. built with a stringApp Disease Query such as query='Rett syndrome', cutoff=0.4, limit=20) needs disease-gene and compound-target annotations added from CyTargetLinker XGMML linksets.

## When NOT to use

- The input is a differentially expressed gene list for pathway extension — use the gene-pathway (WikiPathways) linkset use case instead
- You need author-article-journal relationships from Wikidata rather than compound-target or disease-gene annotations
- The network nodes cannot be matched by the 'display name' attribute the linksets join on

## Inputs

- Protein-protein interaction network in Cytoscape created by stringApp Disease Query (query='Rett syndrome', cutoff=0.4, limit=20)
- CyTargetLinker XGMML linksets: omim-rare-disease-has-20180411.xgmml (curated rare-disease OMIM subset) and chembl_23_hsa_20180126.xgmml
- Network node identifier attribute 'display name' for ID matching

## Outputs

- Extended Cytoscape network containing the original PPI plus 18 OMIM gene-disease associations and 37 ChEMBL compound-target interactions
- Saved Cytoscape session file rett_syndrome_extended_network.cys

## How to apply

Build the base network in Cytoscape 3.6.0+ with stringApp using a Disease Query (query='Rett syndrome', cutoff=0.4, limit=20). Download the date-stamped linksets (chembl_23_hsa_20180126.xgmml, omim-rare-disease-has-20180411.xgmml) from https://cytargetlinker.github.io and run the CyTargetLinker automation command 'cytargetlinker extend' with idAttribute="display name", linkSetFiles pointing at both XGMML linksets, and network=current, optionally via R scripts from the cytargetlinker-automation repository. The extend function parses each linkset and extracts only interactions relevant to nodes in the selected network, joining on the display-name identifier. Optionally apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout), then save the session (rett_syndrome_extended_network.cys). Validate by confirming the extension added exactly 18 gene-disease associations from OMIM and 37 compound-target interactions from ChEMBL.

## Related tools

- **CyTargetLinker** (Runs the 'cytargetlinker extend' automation function that parses the linksets and extracts relevant interactions for the selected network) — https://github.com/CyTargetLinker/cytargetlinker
- **stringApp** (Creates the base Rett syndrome PPI network via Disease Query (query='Rett syndrome', cutoff=0.4, limit=20))
- **Cytoscape** (Host application (3.6.0+) providing the automation interface through which the extend, applyVisualstyle, and applyLayout commands run)
- **R** (External tool optionally used to drive the CyTargetLinker automation commands for repeatable workflows) — https://github.com/CyTargetLinker/cytargetlinker-automation

## Examples

```
cytargetlinker extend idAttribute="display name" linkSetFiles="../Link Sets/chembl_23_hsa_20180126.xgmml,../Link Sets/omim-rare-disease-has-20180411.xgmml" network=current
```

## Evaluation signals

- The extended network contains exactly 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM
- idAttribute is set to 'display name' and network=current is used so edges attach to existing PPI nodes rather than unmatched identifiers
- Session is saved as rett_syndrome_extended_network.cys with the CyTargetLinker visual style and force-directed layout applied
- Each XGMML linkset passed the qc_xgmml.py quality-control procedure before use
- Re-running the workflow reproduces the same edge counts, demonstrating the documented repeatability of the automation

## Limitations

- Counts depend on the date-stamped linkset versions used (chembl_23_hsa_20180126, omim-rare-disease-has-20180411); newer linksets may change the 18/37 edge counts
- The ChEMBL linkset has no deposited persistent DOI yet ('not yet deposited'), complicating version pinning
- A BridgeDb mapper failure during linkset creation leaves every node carrying only its input id, which makes the linkset structurally valid but useless for matching a network

## Evidence

- [methods] Rett syndrome network extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM: "the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM"
- [methods] stringApp Disease Query parameters: [evidence span withheld — anchor status: not_found]
- [intro] CyTargetLinker purpose: [evidence span withheld — anchor status: not_found]
- [readme] Linkset QC requirement: "Do the quality control procedure with the obtained xgmml file (see **QC** below)."
- [discussion] Automation repeatability: [evidence span withheld — anchor status: not_found]
