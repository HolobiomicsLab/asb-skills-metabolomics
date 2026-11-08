---
name: network-extension-with-linksets
description: Use when a Cytoscape network (e.g. a stringApp Disease Query PPI) needs
  to be enriched with compound-target, gene-disease, gene-pathway, or transcription-factor
  interactions from pre-built XGMML linksets, reproducibly via command line or R/Python.
when_to_use_negative:
- You only need to visualize interactions supported by multiple resources rather than
  extend the network — use the 'filterOverlap' function instead of a fresh extension.
- The seed network's node identifiers cannot be matched by any linkset identifier
  mapping (e.g. a failed BridgeDb mapper leaves nodes with only their input id, making
  extension useless).
- The task is linkset creation itself rather than network extension; use the linkset-automation
  LinksetCreator workflow instead.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
tools:
- name: CyTargetLinker
  role: Cytoscape app providing the 'extend' command, linksets, visual style, and
    layout applied to the network
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Platform (3.6.0+) whose automation interface executes cytargetlinker extend
    via command line
- name: stringApp
  role: Creates the seed protein-protein interaction network via Disease Query (query,
    cutoff, limit parameters)
- name: R
  role: External scripting tool driving the Cytoscape automation API through cytargetlinker-automation
    example scripts
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
- name: qc_xgmml.py
  role: Validates that XGMML linksets are well-formed and structurally valid before
    extension
  repo: https://github.com/CyTargetLinker/linkset-automation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:91b48642a623fd5a34e137be4e8299e0913258715962d70ef834dab0bac7b611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - qc_xgmml.py
  - stringApp
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# network-extension-with-linksets

## Summary

Extend a Cytoscape network (e.g. a stringApp PPI network) with prior-knowledge edges from XGMML linksets (ChEMBL compound-target, OMIM gene-disease, WikiPathways, TFLink, etc.) via the CyTargetLinker 'extend' automation command, enabling repeatable, scriptable workflows.

## When to use

Use when a Cytoscape network (e.g. a stringApp Disease Query PPI) needs to be enriched with compound-target, gene-disease, gene-pathway, or transcription-factor interactions from pre-built XGMML linksets, reproducibly via command line or R/Python.

## When NOT to use

- You only need to visualize interactions supported by multiple resources rather than extend the network — use the 'filterOverlap' function instead of a fresh extension.
- The seed network's node identifiers cannot be matched by any linkset identifier mapping (e.g. a failed BridgeDb mapper leaves nodes with only their input id, making extension useless).
- The task is linkset creation itself rather than network extension; use the linkset-automation LinksetCreator workflow instead.

## Inputs

- Cytoscape network (e.g. stringApp PPI from Disease Query 'Rett syndrome', cutoff=0.4, limit=20)
- XGMML linkset files (e.g. chembl_23_hsa_20180126.xgmml, omim-rare-disease-has-20180411.xgmml)
- idAttribute setting for node identifier mapping (e.g. "display name")

## Outputs

- Extended Cytoscape network with added compound-target and gene-disease edges (37 ChEMBL + 18 OMIM for the Rett syndrome case)
- Session file rett_syndrome_extended_network.cys
- Optionally styled/laid-out network (CyTargetLinker visual style, force-directed layout)

## How to apply

Build the seed network in Cytoscape 3.6.0+ with stringApp (e.g. Disease Query 'Rett syndrome', cutoff=0.4, limit=20) and download XGMML linksets from https://cytargetlinker.github.io (e.g. chembl_23_hsa_20180126.xgmml, omim-rare-disease-has-20180411.xgmml). Run 'cytargetlinker extend' with idAttribute="display name", a comma-separated linkSetFiles list, and network=current via the Cytoscape automation API, optionally driven from R using scripts in the cytargetlinker-automation repo. Optionally apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout). Validate the extension by counting added edges — for the Rett syndrome use case, exactly 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM must appear — then save the session (.cys). Linksets can be regenerated with the linkset-automation pipeline (BridgeDb update, preprocessing to input.csv, LinksetCreator, QC with scripts/qc_xgmml.py) and should be validated for BridgeDb cross-reference coverage before use.

## Related tools

- **CyTargetLinker** (Cytoscape app providing the 'extend' command, linksets, visual style, and layout applied to the network) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Platform (3.6.0+) whose automation interface executes cytargetlinker extend via command line)
- **stringApp** (Creates the seed protein-protein interaction network via Disease Query (query, cutoff, limit parameters))
- **R** (External scripting tool driving the Cytoscape automation API through cytargetlinker-automation example scripts) — https://github.com/CyTargetLinker/cytargetlinker-automation
- **qc_xgmml.py** (Validates that XGMML linksets are well-formed and structurally valid before extension) — https://github.com/CyTargetLinker/linkset-automation

## Examples

```
cytargetlinker extend idAttribute="display name" linkSetFiles="../Link Sets/chembl_23_hsa_20180126.xgmml,../Link Sets/omim-rare-disease-has-20180411.xgmml" network=current
```

## Evaluation signals

- Extended Rett syndrome network contains exactly 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM
- qc_xgmml.py exits 0 on the linksets (non-empty graph, edges referencing existing nodes with datasource/interaction attributes)
- BridgeDb mapped-type fraction meets --min-mapped-frac (e.g. 0.9) so nodes carry cross-references needed for id matching
- Session saved as rett_syndrome_extended_network.cys after extension
- Extension is reproducible when re-run from the same seed network and linkset versions

## Limitations

- Linkset resources lack deposited Zenodo DOIs (ChEMBL, Orphanet, Reactome marked 'not yet deposited'), complicating version pinning
- A failed BridgeDb mapper leaves every node with only its input id — structurally valid linksets that cannot match the network
- The linkset-automation repo is explicitly 'Under construction'; the covered resource list is still being extended

## Evidence

- [methods] Extension with ChEMBL and OMIM linksets reproduces reported counts: "the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM"
- [methods] Extend command syntax: [evidence span withheld — anchor status: not_found]
- [abstract] Automation motivation: "The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools (e.g. R, Jupyter, Python, etc)."
- [readme] QC requirement for linksets: "This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id — structurally valid, but useless for matching a network."
- [discussion] Repeatability of workflows: [evidence span withheld — anchor status: not_found]
