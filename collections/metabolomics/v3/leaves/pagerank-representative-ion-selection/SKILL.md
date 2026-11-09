---
name: pagerank-representative-ion-selection
description: Use when clean consensus spectra from an LC–MS/MS untargeted metabolomics
  run contain multiple ionization variants (in-source fragments, adducts, dimers)
  of one precursor that must be deconvoluted to a single protonated [M+H]+ entity
  per variant group.
when_to_use_negative:
- When the precursor ions are already deconvoluted (automatic [M + H]+ ion deconvolution
  was already applied) and no adduct/in-source-fragment variants remain to be grouped
- When the analysis is library identification (step 6, Tremolo/GNPS search) rather
  than ion deconvolution — the IVAMN does not assign chemical identities
- When the data is negative-mode and the ionization rules table was built only for
  positive-mode shifts (in-source fragments, adducts, dimers) without the corresponding
  negative-mode rules
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NP3 MS Workflow
  role: Executes the Annotation/Merging steps 7–8 that build the IVAMN from pairwise
    ionization-variant links and runs PageRank to select [M + H]+ representative ions
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Used to open/inspect the GraphML IVAMN output and verify each variant cluster
    carries exactly one [M + H]+ selection
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:3307d68e88209c51e3113d37ce81e19b75757af16c1588236bbbfee1b31be454
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Cytoscape
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pagerank-representative-ion-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the [M + H]+ representative ion for each group of in-source/adduct ionization variants of the same precursor by running the PageRank algorithm on the ionization variant annotation molecular network (IVAMN) built by NP3 MS Workflow, and writes the selection as node/edge attributes in a GraphML network.

## When to use

Use when clean consensus spectra from an LC–MS/MS untargeted metabolomics run contain multiple ionization variants (in-source fragments, adducts, dimers) of one precursor that must be deconvoluted to a single protonated [M+H]+ entity per variant group.

## When NOT to use

- When the precursor ions are already deconvoluted (automatic [M + H]+ ion deconvolution was already applied) and no adduct/in-source-fragment variants remain to be grouped
- When the analysis is library identification (step 6, Tremolo/GNPS search) rather than ion deconvolution — the IVAMN does not assign chemical identities
- When the data is negative-mode and the ionization rules table was built only for positive-mode shifts (in-source fragments, adducts, dimers) without the corresponding negative-mode rules

## Inputs

- Per-sample MGF files of clean consensus spectra from NP3 MS Workflow steps 3–5 (e.g., from MSV000080502 Euphorbia dendroides LC–MS/MS data)
- Quantification table of precursor ions
- Ionization rules table encoding positive-mode mass shifts for in-source fragments, adducts, and dimers

## Outputs

- IVAMN GraphML network (e.g., ivamn_msv000080502.graphml) with [M+H]+ selections encoded as node/edge attributes
- One selected [M + H]+ representative ion per ionization variant group

## How to apply

Start from the per-sample MGF files of clean consensus spectra (NP3 MS Workflow steps 3–5, clustered with the NP3-shifted cosine function) plus the quantification table. Build an ionization rules table encoding the expected positive-mode mass shifts for in-source fragments, adducts, and dimers of each consensus precursor. In the Annotation/Merging steps 7–8, pairwise-connect consensus spectra whose m/z differences and fragmentation relationships match an ionization-rule entry, then construct the IVAMN graph from those links. Run the PageRank algorithm on the IVAMN to select the [M + H]+ representative ion for each variant group; encode the selection as node/edge attributes. Write the network as GraphML following the reproducibility convention: components written largest-first and edges sorted by source node ID in numerical ordering.

## Related tools

- **NP3 MS Workflow** (Executes the Annotation/Merging steps 7–8 that build the IVAMN from pairwise ionization-variant links and runs PageRank to select [M + H]+ representative ions) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Used to open/inspect the GraphML IVAMN output and verify each variant cluster carries exactly one [M + H]+ selection)

## Examples

```
import networkx as nx; G = nx.read_graphml('ivamn_msv000080502.graphml'); pr = nx.pagerank(G); reps = [n for n, a in G.nodes(data=True) if a.get('m_h_plus_representative')]
```

## Evaluation signals

- Every ionization variant cluster in the IVAMN contains exactly one PageRank-selected [M + H]+ representative ion
- The output GraphML is well-formed and loadable in Cytoscape (e.g., ivamn_msv000080502.graphml)
- Components are written largest-first and edges sorted by source node ID — rerunning the step must reproduce the identical GraphML
- Each selected representative's m/z difference to its grouped variants matches an entry in the ionization rules table (in-source fragment, adduct, or dimer shift)
- The [M + H]+ selection is stored as a node/edge attribute in the GraphML, not only in a side table

## Limitations

- Joining IVAMNs across jobs (join_jobs Step 7) with a noise cutoff required a bug fix (v1.5.1): the filtered IVAMN must reset its index to prevent matching errors in subsequent filters
- Reproducibility depends on the deterministic ordering convention (components largest-first, edges sorted by source node ID) introduced in v1.4.2; older outputs may not be diff-comparable
- The ionization rules table as described encodes expected positive-mode shifts only; coverage is limited to in-source fragments, adducts, and dimers encoded in that table — variants outside the rules will not be grouped

## Evidence
