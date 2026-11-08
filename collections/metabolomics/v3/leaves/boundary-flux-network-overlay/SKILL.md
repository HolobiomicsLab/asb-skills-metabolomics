---
name: boundary-flux-network-overlay
description: Use when you have time-resolved metabolomics intensity data and/or boundary
  fluxes for an organism and want to identify reactions absent from KEGG (e.g., flagged
  light-gray nodes/edges) in a species-specific metabolic network.
when_to_use_negative:
- Input lacks empirical metabolomics intensity or boundary-flux data (only genomic
  annotation is available) — MINNO's refinement loop requires metabolite evidence
  to flag missing reactions
- The goal is a genome-scale stoichiometric FBA model rather than visual network refinement
  — MINNO explicitly avoids complex mathematical modeling and can access BiGG models
  only for network viewing
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3174
tools:
- name: MINNO
  role: Web-based mapping tool that assembles the KEGG/RefSeq organism network and
    accepts empirical metabolomics data to refine it
  repo: https://lewisresearchgroup.github.io/MINNO/
- name: D3.js
  role: JavaScript library used by MINNO to render the dynamic interactive network
    visualizations in web browsers
- name: KEGG
  role: Source of the 66 base metabolic pathway layouts and compound node identifiers
    onto which metabolites are mapped
- name: NCBI-RefSeq
  role: Source of genetic annotation data used to generate organism-specific pathway
    nodes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:88fcd57cdd6737c9547ca3dab089d708e84a4a991abf95223452ee2409198042
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - D3.js
  - KEGG
  - MINNO
  - NCBI-RefSeq
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# boundary-flux-network-overlay

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Overlay empirical temporal metabolite intensity profiles and boundary fluxes (e.g., extracellular consumption/secretion, isotope labeling) onto a KEGG/NCBI-RefSeq-derived organism-specific metabolic network in MINNO to visually flag disconnected nodes and missing reactions, without complex mathematical modeling.

## When to use

Use when you have time-resolved metabolomics intensity data and/or boundary fluxes for an organism and want to identify reactions absent from KEGG (e.g., flagged light-gray nodes/edges) in a species-specific metabolic network.

## When NOT to use

- Input lacks empirical metabolomics intensity or boundary-flux data (only genomic annotation is available) — MINNO's refinement loop requires metabolite evidence to flag missing reactions
- The goal is a genome-scale stoichiometric FBA model rather than visual network refinement — MINNO explicitly avoids complex mathematical modeling and can access BiGG models only for network viewing

## Inputs

- 66 base KEGG metabolic pathway layout files (shipped with MINNO, covering all primary metabolic pathways)
- NCBI-RefSeq genetic annotation data for the target organism (e.g., Borrelia species)
- Empirical metabolite intensity table sampled over time intervals
- Boundary flux / intra- or extracellular data table (e.g., extracellular metabolite consumption/secretion, isotope labeling patterns)

## Outputs

- Species-specific metabolic network visualization (D3.js interactive web view) with dark annotated and light-gray unannotated nodes/edges
- network_refinement_map.csv containing flagged missing reactions and disconnected nodes

## How to apply

Load the 66 base KEGG metabolic pathway layouts shipped with MINNO, then combine them with the organism's NCBI-RefSeq genetic annotation to build the species-specific network (annotated reactions render as dark nodes/edges; unannotated ones as light gray). Next, map each empirically detected metabolite from the boundary-flux/intensity table onto its KEGG compound node and overlay temporal intensity profiles and intra-/extracellular evidence such as boundary fluxes and isotope labeling patterns. Where flux or intensity evidence indicates activity but KEGG lacks a connecting reaction (e.g., PnP converting thymidine to thymine, or an unannotated ribonucleotide reductase in B. parkeri and B. turicatae), flag the position as a missing reaction or disconnected node. Write the annotated refinement map (network_refinement_map.csv) and use it to guide follow-up searches for genes/proteins corresponding to the flagged reactions.

## Related tools

- **MINNO** (Web-based mapping tool that assembles the KEGG/RefSeq organism network and accepts empirical metabolomics data to refine it) — https://lewisresearchgroup.github.io/MINNO/
- **D3.js** (JavaScript library used by MINNO to render the dynamic interactive network visualizations in web browsers)
- **KEGG** (Source of the 66 base metabolic pathway layouts and compound node identifiers onto which metabolites are mapped)
- **NCBI-RefSeq** (Source of genetic annotation data used to generate organism-specific pathway nodes)

## Examples

```
open https://lewisresearchgroup.github.io/MINNO/ in a browser; load KEGG base pathways + NCBI-RefSeq annotation, upload metabolite intensity/boundary flux table, export network_refinement_map.csv
```

## Evaluation signals

- The refinement map reproduces the 18 missing reactions reported from the KEGG database, nine of which are supported by primary literature
- For B. burgdorferi, the purine/pyrimidine subset yields eight missing reactions (six purine and four pyrimidine reactions, per Table S1 confirmation counts)
- Thymine production from thymidine in the boundary flux profile correctly flags a pyrimidine-nucleoside phosphorylase (PnP) reaction
- Boundary flux evidence correctly flags ribonucleotide reductase activity in B. parkeri and B. turicatae despite no KEGG annotation
- Annotated reactions render as dark nodes/edges and unannotated/missing ones as light gray nodes/edges in the D3.js view

## Limitations

- Missing-reaction flags are visual/inferential: users must separately search for genes and proteins corresponding to flagged reactions using experimental or bioinformatics methods
- Supporting validation details reside in supplementary Table S1, which is not included in the main text
- No version number stated for the MINNO software and no changelog present in the source content
- Boundary flux profiles can only suggest (not prove) the presence of an enzyme, e.g., ribonucleotide reductase in B. parkeri/B. turicatae is anticipated, not confirmed

## Evidence

- [abstract] MINNO web tool refines networks with metabolomics overlay: "we developed Metabolic Interactive Nodular Network for Omics (MINNO), a web-based mapping tool that takes in empirical metabolomics data to refine metabolic networks"
- [results] Overlay workflow of temporal profiles and boundary fluxes: "The user can then incorporate temporal metabolite intensity profiles and intra- or extracellular data onto the network to infer missing reactions by considering boundary fluxes"
- [results] PnP flagged from thymidine-to-thymine boundary flux: "the boundary flux profile of this species suggests the presence of pyrimidine-nucleoside phosphorylase (PnP) based on the production of thymine from thymidine"
- [discussion] 18 missing KEGG reactions as validation benchmark: "we identified 18 missing reactions from the KEGG database, of which nine were supported by the primary literature."
- [readme] MINNO visualization tool entry point: "Visualization tool link is here: https://lewisresearchgroup.github.io/MINNO/"
