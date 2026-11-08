---
name: ppi-network-construction-from-disease-query
description: Use when the analysis goal is to generate a PPI network for a specific
  disease in Cytoscape as a seed network for later linkset-based extension (e.g. compound-target
  or gene-disease annotation).
when_to_use_negative:
- The network already exists (e.g. a differentially expressed gene list is the input)
  — use gene-pathway extension workflows instead of a disease query
- The goal is a non-biological network such as the author-article-journal network
  built via the two-step Wikidata linkset extension
- The target interaction data is not protein-protein associations (e.g. only pathway
  membership is needed)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3366
tools:
- name: stringApp
  role: Runs the Disease Query that creates the disease-focused PPI network seed
- name: Cytoscape
  role: Hosts the network; 3.6.0+ provides the automation interface used for command-line
    execution
- name: CyTargetLinker
  role: Extends the stringApp-created PPI network with ChEMBL and OMIM linksets via
    the 'extend' function
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: R
  role: Optionally drives the Cytoscape automation API via the cytargetlinker-automation
    scripts
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:a7f84d35bb59166aab95b59441516f2ca74758666828c07bf193b9697b26a309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  - stringApp
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ppi-network-construction-from-disease-query

## Summary

Builds a disease-focused protein-protein interaction network in Cytoscape using stringApp's Disease Query (e.g. query="Rett syndrome", cutoff=0.4, limit=20), producing a seed network that can subsequently be extended with external linksets such as ChEMBL and OMIM via CyTargetLinker.

## When to use

Use when the analysis goal is to generate a PPI network for a specific disease in Cytoscape as a seed network for later linkset-based extension (e.g. compound-target or gene-disease annotation).

## When NOT to use

- The network already exists (e.g. a differentially expressed gene list is the input) — use gene-pathway extension workflows instead of a disease query
- The goal is a non-biological network such as the author-article-journal network built via the two-step Wikidata linkset extension
- The target interaction data is not protein-protein associations (e.g. only pathway membership is needed)

## Inputs

- Disease name string (e.g. 'Rett syndrome')
- stringApp Disease Query parameters: cutoff=0.4 confidence score, limit=20 interaction partners
- Cytoscape 3.6.0+ with stringApp installed

## Outputs

- Disease-specific protein-protein interaction network in Cytoscape (.cys session, e.g. rett_syndrome_extended_network.cys)
- Seed network ready for CyTargetLinker extension with compound-target and gene-disease linksets

## How to apply

In Cytoscape (3.6.0+) with stringApp installed, run a Disease Query for the disease of interest — in the Rett syndrome use case, query="Rett syndrome", cutoff=0.4 (string confidence score threshold) and limit=20 (maximum interaction partners per protein) — to create the protein-protein interaction network. The resulting network serves as the input network for CyTargetLinker's 'extend' function with idAttribute="display name", where linksets such as chembl_23_hsa_20180126.xgmml (compound-target) and omim-rare-disease-has-20180411.xgmml (gene-disease) add annotations. The whole workflow can be driven programmatically via the Cytoscape command-line automation API, optionally from R using the cytargetlinker-automation scripts, to keep it repeatable and reproducible. After extension, optionally apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout), then save the session (e.g. rett_syndrome_extended_network.cys).

## Related tools

- **stringApp** (Runs the Disease Query that creates the disease-focused PPI network seed)
- **Cytoscape** (Hosts the network; 3.6.0+ provides the automation interface used for command-line execution)
- **CyTargetLinker** (Extends the stringApp-created PPI network with ChEMBL and OMIM linksets via the 'extend' function) — https://github.com/CyTargetLinker/cytargetlinker
- **R** (Optionally drives the Cytoscape automation API via the cytargetlinker-automation scripts) — https://github.com/CyTargetLinker/cytargetlinker-automation

## Examples

```
cytargetlinker extend idAttribute="display name" linkSetFiles="../Link Sets/chembl_23_hsa_20180126.xgmml,../Link Sets/omim-rare-disease-has-20180411.xgmml" network=current
```

## Evaluation signals

- After extension with chembl_23_hsa_20180126.xgmml and omim-rare-disease-has-20180411.xgmml, the network contains exactly 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM
- The Disease Query used cutoff=0.4 and limit=20 for query='Rett syndrome', matching the reported build parameters
- Network node identifiers resolve via idAttribute="display name" so linkset nodes actually match network nodes
- Final session saves without errors as rett_syndrome_extended_network.cys with the CyTargetLinker visual style and force-directed layout applied

## Limitations

- Requires Cytoscape 3.6.0+ for the automation interface; older versions cannot run the command-line workflow
- Query parameters (cutoff, limit) directly determine network size and thus downstream extension counts; deviating from cutoff=0.4 / limit=20 will not reproduce the 37 ChEMBL and 18 OMIM added interactions
- Linkset files are dated snapshots (e.g. chembl_23_hsa_20180126.xgmml); using newer releases changes added-interaction counts

## Evidence

- [methods] stringApp Disease Query parameters: [evidence span withheld — anchor status: not_found]
- [methods] extension counts: "the network was extended with 37 compound-target interactions from ChEMBL and 18 gene-disease associations from OMIM"
- [methods] extend command: [evidence span withheld — anchor status: not_found]
- [abstract] automation rationale: "The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools (e.g. R, Jupyter, Python, etc)."
- [supplementary] tutorials with R code: "Tutorials including R code for use cases: https://github. com/Cy Target Linker/cytargetlinker-automation"
