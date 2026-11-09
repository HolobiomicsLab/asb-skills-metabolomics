---
name: network-visualization-styling
description: Use when a network has just been extended in Cytoscape (e.g. via 'cytargetlinker
  extend') and needs the CyTargetLinker visual style, a force-directed layout, and
  node-type-specific coloring before export or figure production.
when_to_use_negative:
- The network has not yet been extended with linksets — styling should follow the
  'extend' step, not replace it
- The target is a single-type network (e.g. a PPI from stringApp's Rett syndrome Disease
  Query) where per-type node coloring is not the goal
- The required visual encoding is quantitative (e.g. expression fold-change mapping)
  rather than categorical node-type coloring
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CyTargetLinker
  role: Provides the visual style (applyVisualstyle) and layout automation (applyLayout)
    commands applied to the extended network
  repo: https://cytargetlinker.github.io/
- name: Cytoscape
  role: Platform in which the extended network is styled, laid out, manually adapted,
    and exported as XGMML
- name: R
  role: External tool used to call the CyTargetLinker automation commands programmatically
  repo: https://github.com/CyTargetLinker/cytargetlinker-automation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:fbf1431cb31eb80449826d6f6849d341c9ed5919ccad6b9a5e24fa0aa9d77fca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CyTargetLinker
  - Cytoscape
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# network-visualization-styling

## Summary

Apply the CyTargetLinker visual style and a force-directed layout to an extended Cytoscape network, then color node types (e.g. gray authors, yellow articles, green journals) so the multi-type structure is readable and matches the paper's figures.

## When to use

Use when a network has just been extended in Cytoscape (e.g. via 'cytargetlinker extend') and needs the CyTargetLinker visual style, a force-directed layout, and node-type-specific coloring before export or figure production.

## When NOT to use

- The network has not yet been extended with linksets — styling should follow the 'extend' step, not replace it
- The target is a single-type network (e.g. a PPI from stringApp's Rett syndrome Disease Query) where per-type node coloring is not the goal
- The required visual encoding is quantitative (e.g. expression fold-change mapping) rather than categorical node-type coloring

## Inputs

- Extended Cytoscape network (result of 'cytargetlinker extend' with linkset XGMML files)
- Node type attribute (e.g. node type column distinguishing authors, articles, journals)
- CyTargetLinker visual style definition (installed with CyTargetLinker 4.0.0+)

## Outputs

- Styled network with CyTargetLinker visual style and force-directed layout applied
- Color-coded node types (e.g. gray author, yellow article, green journal nodes)
- Exported styled network file, e.g. author_article_journal_network.xgmml

## How to apply

After the extension steps complete, run the CyTargetLinker automation commands 'applyVisualstyle' and 'applyLayout' with network=current to apply the app's dedicated visual style and a force-directed layout to the extended network. Then override node fill colors per node type — in the author-article-journal use case, author nodes gray, article nodes yellow, journal nodes green as in Figure 3. Perform minor manual layout adaptation for readability of the two-step author→article→journal structure. Finally export the styled network (e.g. to author_article_journal_network.xgmml). Rationale: the automated style plus layout makes the newly added linkset interactions visually distinguishable from the seed network, which the paper highlights as a key advantage of the automation.

## Related tools

- **CyTargetLinker** (Provides the visual style (applyVisualstyle) and layout automation (applyLayout) commands applied to the extended network) — https://cytargetlinker.github.io/
- **Cytoscape** (Platform in which the extended network is styled, laid out, manually adapted, and exported as XGMML)
- **R** (External tool used to call the CyTargetLinker automation commands programmatically) — https://github.com/CyTargetLinker/cytargetlinker-automation

## Examples

```
Rscript -e 'library(RCy3); commandsRun("cytargetlinker applyVisualstyle network=current"); commandsRun("cytargetlinker applyLayout network=current")'
```

## Evaluation signals

- The network renders with the CyTargetLinker visual style after 'cytargetlinker applyVisualstyle network=current'
- A force-directed layout is applied (applyLayout) and node positions reflect the author→article→journal topology
- Color check: five author nodes gray, article nodes yellow, journal nodes green, matching Figure 3
- The exported author_article_journal_network.xgmml preserves the two-step structure and styling attributes
- Visual readability achieved with only minor manual layout adaptation (no wholesale repositioning)

## Limitations

- Force-directed layout usually requires minor manual adaptation for readability of multi-type networks
- The CyTargetLinker visual style is designed for linkset-extended networks; node-type coloring must be set manually per use case (colors are not inferred automatically)

## Evidence

- [methods] Apply the Cy Target Linker visual style to the extended network: "Apply the Cy Target Linker visual style to the extended network"
- [methods] apply Layout network Apply a force-directed layout: "apply Layout network Apply a force-directed layout"
- [abstract] used Cy Target Linker to create an author-article-journal network for the five authors of this manuscript using a two-step extension mechanism.: "used Cy Target Linker to create an author-article-journal network for the five authors of this manuscript using a two-step extension mechanism."
