---
name: xgmml-linkset-generation
description: Use when building a custom-made linkset to extend a Cytoscape network
  with CyTargetLinker, e.g. converting a tab-delimited relationship table (source/target
  id, type, datasource columns) into a validated XGMML linkset.
when_to_use_negative:
- You only want to consume existing linksets to extend a network — that is CyTargetLinker's
  'extend' function, not linkset generation
- The linkset you need is already provided on the CyTargetLinker website (e.g. WikiPathways
  13-species or TFLink 5-species linksets with Zenodo DOIs)
- You need to filter an already-extended network by resource support — use filterOverlap
  instead
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: LinksetCreator
  role: Converts the preprocessed input.csv into the XGMML linkset file
  repo: https://github.com/CyTargetLinker/link-set Creator
- name: scripts/qc_xgmml.py
  role: 'Quality control of the produced XGMML linkset: checks well-formedness, edge
    endpoint resolvability, required attributes, and BridgeDb mapping coverage'
  repo: https://github.com/CyTargetLinker/linkset-automation
- name: CyTargetLinker
  role: Consumes the generated linkset via its 'extend' function to add interactions
    to a Cytoscape network
  repo: https://github.com/CyTargetLinker/cytargetlinker
- name: BridgeDb
  role: Provides cross-reference identifier mapping so linkset nodes can match network
    identifiers; coverage is asserted during QC
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.12688/f1000research.14613.2
    title: 'CyTargetLinker app update: A flexible solution for network extension in
      Cytoscape'
  source_content_hash: sha256:c8fae8d96258863fc929be7b499fe695ab536c865b1ae5f8077950887a37a42f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CyTargetLinker/link-set Creator
  tools:
  - BridgeDb
  - CyTargetLinker
  - LinksetCreator
  - Python
  - R
  - scripts/qc_xgmml.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# xgmml-linkset-generation

## Summary

Generates CyTargetLinker linksets in XGMML format from tab-delimited relationship tables (input.csv with source/target identifiers, entity types, and datasource), either manually or via automated pipelines, and validates them with scripts/qc_xgmml.py. Renaming RegINs to linksets reflects the app's broadened applicability beyond regulatory interactions.

## When to use

Use when building a custom-made linkset to extend a Cytoscape network with CyTargetLinker, e.g. converting a tab-delimited relationship table (source/target id, type, datasource columns) into a validated XGMML linkset.

## When NOT to use

- You only want to consume existing linksets to extend a network — that is CyTargetLinker's 'extend' function, not linkset generation
- The linkset you need is already provided on the CyTargetLinker website (e.g. WikiPathways 13-species or TFLink 5-species linksets with Zenodo DOIs)
- You need to filter an already-extended network by resource support — use filterOverlap instead

## Inputs

- Tab-delimited relationship table (input.csv) with source/target identifiers, entity types, and datasource columns
- BridgeDb identifier-mapping config files (BridgeDb versions updated per release)
- Resource download link for the source database (e.g. WikiPathways, TFLink, ChEMBL, Orphanet, Reactome)

## Outputs

- XGMML linkset file (linkset.xgmml) with a <graph>, nodes with identifiers list + type attribute, and edges with datasource/interaction attributes
- QC validation report from scripts/qc_xgmml.py (non-zero exit code on failure)

## How to apply

Read the tab-delimited relationship table (input.csv) holding source/target identifiers, entity types, and datasource columns; run the preprocessing script to get the input.csv for the current version, then run the LinksetCreator to emit an XGMML file. The output must contain a single <graph> element whose nodes carry an identifiers list and a type attribute, and whose edges reference existing source/target nodes and carry datasource and interaction attributes, mirroring the linkset structure CyTargetLinker's 'extend' function parses. Validate the produced linkset with python3 scripts/qc_xgmml.py — the script fails on empty graphs by default and supports --mapped-type/--min-mapped-frac (e.g. --min-mapped-frac 0.9) to require BridgeDb cross-reference coverage, which catches a BridgeDb mapper that silently left every node carrying only its input id. If QC exits zero, run the linkset upload-to-storage script and update the CTL website; the same check runs in CI via .github/workflows/qc-xgmml.yml.

## Related tools

- **LinksetCreator** (Converts the preprocessed input.csv into the XGMML linkset file) — https://github.com/CyTargetLinker/link-set Creator
- **scripts/qc_xgmml.py** (Quality control of the produced XGMML linkset: checks well-formedness, edge endpoint resolvability, required attributes, and BridgeDb mapping coverage) — https://github.com/CyTargetLinker/linkset-automation
- **CyTargetLinker** (Consumes the generated linkset via its 'extend' function to add interactions to a Cytoscape network) — https://github.com/CyTargetLinker/cytargetlinker
- **BridgeDb** (Provides cross-reference identifier mapping so linkset nodes can match network identifiers; coverage is asserted during QC)

## Examples

```
python3 scripts/qc_xgmml.py --mapped-type gene --min-mapped-frac 0.9 linkset.xgmml
```

## Evaluation signals

- scripts/qc_xgmml.py exits with code 0 on the produced linkset.xgmml (non-empty graph, resolvable edge source/target endpoints, required identifiers/type/datasource/interaction attributes)
- With --mapped-type gene --min-mapped-frac 0.9, the fraction of gene nodes with more than one identifier meets the threshold (catches a BridgeDb mapper that failed to load)
- The XGMML contains a single <graph> element and edges whose source/target reference existing nodes
- The same QC check passes in CI via .github/workflows/qc-xgmml.yml
- The generated linkset can be consumed by 'cytargetlinker extend', which parses the linkset and extracts relevant interactions for the selected network

## Limitations

- ChEMBL, Orphanet, and Reactome linksets are 'not yet deposited' — no persistent Zenodo DOI exists for them yet; the license of the eventual deposit may still need deciding
- Empty graphs fail QC by default; passing --allow-empty is needed for legitimately empty files
- A BridgeDb mapper that failed to load leaves every node with only its input id — structurally valid XGMML but useless for matching a network, so QC only catches it via the mapped-fraction check
- The WikiPathways pipeline no longer needs the BridgeDb-update/preprocessing steps (it resolves the release and generates configs on every run), so workflow steps differ per resource

## Evidence

- [readme] qc validates XGMML structure: "`scripts/qc_xgmml.py` validates that an XGMML linkset is well-formed and has the structure every CyTargetLinker linkset carries (a `<graph>`, nodes with an `identifiers` list + `type`"
- [methods] workflow steps for linkset creation: "3. Run the preprocessing script to get the input.csv file for this version."
- [intro] renaming RegINs to linksets: "we renamed the previously used Regulatory Interaction Networks (RegINs) to linksets to make the broader applicability more explicit."
- [readme] BridgeDb mapping coverage QC: "This catches a BridgeDb mapper that failed to load, which leaves every node carrying only its input id"
- [discussion] user-friendly linkset generation: [evidence span withheld — anchor status: not_found]
