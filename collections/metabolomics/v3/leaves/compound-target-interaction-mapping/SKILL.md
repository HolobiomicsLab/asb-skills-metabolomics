---
name: compound-target-interaction-mapping
description: Use when a protein-protein or gene network in Cytoscape must be enriched
  with drug/compound-target edges from ChEMBL linksets, e.g. to annotate disease-gene
  networks with pharmacological interactions.
when_to_use_negative:
- The seed network already contains compound-target annotations, since CyTargetLinker
  extend adds linkset-matched edges and duplicates are not the intended use
- The network node identifiers do not match linkset identifiers (e.g. BridgeDb mapping
  failed leaving only input IDs), so extend would return no or wrong matches
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0208
tools:
- name: CyTargetLinker
  role: Provides the 'extend' function that parses the ChEMBL XGMML linkset and adds
    matching compound-target interactions to the current network
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: stringApp
  role: Builds the seed protein-protein interaction network via Disease Query (query='Rett
    syndrome', cutoff=0.4, limit=20) that is then extended
- name: Cytoscape
  role: Host application (3.6.0+) whose automation interface executes cytargetlinker
    extend programmatically from the command line or R
- name: R
  role: Drives the Cytoscape command-line automation via scripts in the cytargetlinker-automation
    repository
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:9e0981e5a22df4ea05c4f793737f6f3d4de093680690e1ad7ebdef80f298e23a
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
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# compound-target-interaction-mapping

## Summary

Maps compound-target interactions from a ChEMBL-derived CyTargetLinker linkset (XGMML) onto an existing Cytoscape network using the CyTargetLinker 'extend' function, adding drug-target edges to a seed network such as a stringApp Rett syndrome PPI.

## When to use

Use when a protein-protein or gene network in Cytoscape must be enriched with drug/compound-target edges from ChEMBL linksets, e.g. to annotate disease-gene networks with pharmacological interactions.

## When NOT to use

- The seed network already contains compound-target annotations, since CyTargetLinker extend adds linkset-matched edges and duplicates are not the intended use
- The network node identifiers do not match linkset identifiers (e.g. BridgeDb mapping failed leaving only input IDs), so extend would return no or wrong matches

## Inputs

- Cytoscape network (e.g. stringApp Disease Query PPI for 'Rett syndrome', cutoff=0.4, limit=20)
- ChEMBL compound-target linkset in XGMML format (chembl_23_hsa_20180126.xgmml)
- Node identifier attribute matching linkset IDs (idAttribute='display name')

## Outputs

- Extended Cytoscape network containing 37 additional compound-target edges from ChEMBL
- Session file (e.g. rett_syndrome_extended_network.cys)
- Optionally CyTargetLinker visual style and force-directed layout applied to the extended network

## How to apply

First build or load the seed network in Cytoscape (3.6.0+); in the Rett syndrome use case, a PPI is generated with stringApp via Disease Query (query='Rett syndrome', cutoff=0.4, limit=20). Obtain the ChEMBL linkset XGMML (e.g. chembl_23_hsa_20180126.xgmml) from the CyTargetLinker website, and ensure node identifiers match by setting idAttribute='display name'. Run the 'cytargetlinker extend' command with linkSetFiles pointing to the ChEMBL XGMML and network=current, so the extend function parses the linkset and extracts only the compound-target interactions matching nodes in the selected network. Optionally apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout), and validate that the expected number of new edges (37 compound-target interactions in the Rett syndrome case) was added.

## Related tools

- **CyTargetLinker** (Provides the 'extend' function that parses the ChEMBL XGMML linkset and adds matching compound-target interactions to the current network) — https://github.com/CyTargetLinker/cytargetlinker
- **stringApp** (Builds the seed protein-protein interaction network via Disease Query (query='Rett syndrome', cutoff=0.4, limit=20) that is then extended)
- **Cytoscape** (Host application (3.6.0+) whose automation interface executes cytargetlinker extend programmatically from the command line or R)
- **R** (Drives the Cytoscape command-line automation via scripts in the cytargetlinker-automation repository) — https://github.com/CyTargetLinker/cytargetlinker-automation

## Examples

```
cytargetlinker extend idAttribute="display name" linkSetFiles="../Link Sets/chembl_23_hsa_20180126.xgmml" network="current"
```

## Evaluation signals

- The extended network contains exactly 37 compound-target interactions from ChEMBL, matching the reported Rett syndrome use-case count
- Every added edge connects a network node to a compound node sourced from the ChEMBL linkset (datasource/interaction attributes present in the XGMML)
- The linkset XGMML passes the qc_xgmml.py structural validation (graph with nodes carrying identifiers + type, edges with source/target/datasource/interaction)
- BridgeDb cross-reference coverage is adequate, e.g. --min-mapped-frac 0.9 on gene nodes in the linkset QC
- Session saved as rett_syndrome_extended_network.cys reproduces the edge counts on re-run

## Limitations

- Linkset versioning matters: the result depends on the exact linkset file (chembl_23_hsa_20180126.xgmml); the ChEMBL linkset in the automation repo is 'not yet deposited' so no persistent Zenodo DOI currently backs it
- Matching relies on identifier resolution; a failed BridgeDb mapper leaves every node carrying only its input id, which is structurally valid but useless for matching a network
- The workflow requires Cytoscape 3.6.0+ for the automation interface

## Evidence

- [methods] the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM: "the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM"
- [methods] Extend network with linksets using cytargetlinker extend: [evidence span withheld — anchor status: not_found]
- [methods] The key function is the extend function: [evidence span withheld — anchor status: not_found]
- [methods] compound-target interactions from ChEMBL9,10: "compound-target interactions from ChEMBL9,10"
- [readme] Linkset creation QC via qc_xgmml.py: "scripts/qc_xgmml.py` validates that an XGMML linkset is well-formed and has the structure every CyTargetLinker linkset carries"
