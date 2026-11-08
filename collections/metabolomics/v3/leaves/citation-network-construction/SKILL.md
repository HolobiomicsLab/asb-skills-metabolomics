---
name: citation-network-construction
description: Use when the goal is to construct a heterogeneous non-biological network
  (e.g. authors linked to articles linked to journals) via automated two-step linkset
  extension in Cytoscape.
when_to_use_negative:
- The network is a biological interaction network better served by curated linksets
  (WikiPathways, ChEMBL, OMIM, TFLink, Reactome) rather than Wikidata-derived citation
  linksets
- Only a single entity type is needed — a one-step extension with one linkset suffices
  and the two-step mechanism is unnecessary
edam_operation: http://edamontology.org/operation_3083
tools:
- name: CyTargetLinker
  role: Performs the two-step 'extend' operations and supplies the visual style applied
    to the extended network
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: Cytoscape
  role: Hosts the network, executes the cytargetlinker extend API commands, and provides
    the force-directed layout
- name: R
  role: External tool driving the automated CyTargetLinker workflow programmatically
    via the new automation feature
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:041dc4060016a0b2b6fa08ce023e3f6cd0702a84f25cfc66c0c7bd9e4b8bfa6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/cytargetlinker
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  edam_operation: http://edamontology.org/operation_3083
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# citation-network-construction

## Summary

Builds a multi-type author→article→journal citation network by applying CyTargetLinker's two-step 'extend' mechanism with Wikidata-derived linksets (publications.xgmml, journals.xgmml) in Cytoscape, demonstrating the app's applicability beyond biological regulatory networks.

## When to use

Use when the goal is to construct a heterogeneous non-biological network (e.g. authors linked to articles linked to journals) via automated two-step linkset extension in Cytoscape.

## When NOT to use

- The network is a biological interaction network better served by curated linksets (WikiPathways, ChEMBL, OMIM, TFLink, Reactome) rather than Wikidata-derived citation linksets
- Only a single entity type is needed — a one-step extension with one linkset suffices and the two-step mechanism is unnecessary

## Inputs

- Initial Cytoscape network with five author nodes (idAttribute = 'shared name')
- Wikidata-derived author-article linkset (Link Sets/publications.xgmml, XGMML format)
- Wikidata-derived article-journal linkset (Link Sets/journals.xgmml, XGMML format)

## Outputs

- Two-step extended author-article-journal network (author_article_journal_network.xgmml)
- Styled network with gray author nodes, yellow article nodes, green journal nodes (Figure 3 layout)

## How to apply

Load the initial five author nodes into a network in Cytoscape (3.6.0+) with CyTargetLinker 4.0.0+ installed. Run 'cytargetlinker extend' with idAttribute="shared name", linkSetFiles="../Link Sets/publications.xgmml" and network=current to add author-article relationships from the Wikidata-derived linkset. Run 'cytargetlinker extend' a second time with idAttribute="shared name" and linkSetFiles="../Link Sets/journals.xgmml" to attach journal nodes to the newly added article nodes, exploiting the two-step extension mechanism. Apply the CyTargetLinker visual style (applyVisualstyle) and a force-directed layout (applyLayout), then color author nodes gray, article nodes yellow, and journal nodes green as in Figure 3, with minor manual layout adaptation for readability. Finally export the network to author_article_journal_network.xgmml and validate the two-step author→article→journal structure.

## Related tools

- **CyTargetLinker** (Performs the two-step 'extend' operations and supplies the visual style applied to the extended network) — https://github.com/CyTargetLinker/cytargetlinker
- **Cytoscape** (Hosts the network, executes the cytargetlinker extend API commands, and provides the force-directed layout)
- **R** (External tool driving the automated CyTargetLinker workflow programmatically via the new automation feature) — https://github.com/CyTargetLinker/cytargetlinker-automation

## Examples

```
cytargetlinker extend idAttribute="shared name" linkSetFiles="../Link Sets/publications.xgmml" network=current
```

## Evaluation signals

- Exported author_article_journal_network.xgmml contains exactly five gray author nodes
- Every author connects to journal nodes only via intermediate yellow article nodes, forming a two-step author→article→journal structure
- Journal nodes are green and article nodes are yellow after applying the CyTargetLinker visual style, matching Figure 3
- Both extend commands (publications.xgmml then journals.xgmml) report successful parsing and interaction extraction with network=current
- Linkset files pass structural QC: well-formed <graph>, nodes with identifiers + type, edges referencing existing nodes with datasource/interaction

## Limitations

- The linkset-automation repo is 'under construction' and ChEMBL, Orphanet, and Reactome linksets are not yet deposited with persistent DOIs
- The QC script fails empty graphs by default, so an extend step against an empty or malformed XGMML linkset will fail validation
- The Wikidata-derived author-article and article-journal linksets are custom-made for the manuscript's five authors, so reuse requires regenerating linksets for new author sets

## Evidence

- [methods] two custom made linksets for author-article and article-journal relationships from Wikidata16–18: "two custom made linksets for author-article and article-journal relationships from Wikidata16–18"
- [abstract] used CyTargetLinker to create an author-article-journal network for the five authors of this manuscript using a two-step extension mechanism.: "used CyTargetLinker to create an author-article-journal network for the five authors of this manuscript using a two-step extension mechanism."
- [abstract] The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools: "The new automation feature enables users to programmatically execute the app's functionality in Cytoscape (command line tool) and with external tools"
- [intro] we present three use cases that highlight the app’s purpose, how it can be easily combined with other Cytoscape apps and the advantages of the automation.: "we present three use cases that highlight the app’s purpose, how it can be easily combined with other Cytoscape apps and the advantages of the automation."
- [readme] `scripts/qc_xgmml.py` validates that an XGMML linkset is well-formed and has the structure every CyTargetLinker linkset carries: "`scripts/qc_xgmml.py` validates that an XGMML linkset is well-formed and has the structure every CyTargetLinker linkset carries"
