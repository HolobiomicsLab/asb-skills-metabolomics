---
name: metabolite-node-assignment-onto-pathway-graph
description: Use when empirical metabolomics data (time-course metabolite intensities,
  boundary fluxes, isotope labeling) must be placed onto KEGG-derived pathway nodes
  of a genome-annotated organism to infer missing or unannotated reactions.
when_to_use_negative:
- The organism or pathway of interest is not among the 66 KEGG base metabolic pathway
  layouts shipped with MINNO, and no KEGG ortholog layout exists
- The goal is quantitative stoichiometric modeling or flux balance analysis rather
  than visual inference of missing reactions from empirical overlays
- No empirical metabolomics data (intensities, boundary fluxes, isotope labeling)
  are available to overlay — the tool refines networks using empirical evidence, not
  genome annotation alone
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3407
tools:
- name: MINNO
  role: Web-based mapping tool that loads KEGG base layouts and RefSeq annotations,
    then accepts empirical metabolomics data to refine and visualize the species-specific
    network
  repo: https://github.com/LewisResearchGroup/MINNO
- name: D3.js
  role: JavaScript library MINNO uses to render the dynamic interactive network visualization
    in web browsers
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
  source_content_hash: sha256:5f497a86948086fbf95506fcf3e6ec8dfbf9144fe2b52cd27639f53b90111e61
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  repo_url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - D3.js
  - KEGG
  - MINNO
  - NCBI-RefSeq
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-node-assignment-onto-pathway-graph

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map empirically detected metabolites (temporal intensity profiles and boundary fluxes) onto KEGG compound nodes of a species-specific metabolic network built in MINNO, so that active-but-disconnected positions flag missing or unannotated reactions without mathematical modeling.

## When to use

Use when empirical metabolomics data (time-course metabolite intensities, boundary fluxes, isotope labeling) must be placed onto KEGG-derived pathway nodes of a genome-annotated organism to infer missing or unannotated reactions.

## When NOT to use

- The organism or pathway of interest is not among the 66 KEGG base metabolic pathway layouts shipped with MINNO, and no KEGG ortholog layout exists
- The goal is quantitative stoichiometric modeling or flux balance analysis rather than visual inference of missing reactions from empirical overlays
- No empirical metabolomics data (intensities, boundary fluxes, isotope labeling) are available to overlay — the tool refines networks using empirical evidence, not genome annotation alone

## Inputs

- 66 base metabolic pathway layout files from the KEGG database (shipped with MINNO)
- NCBI-RefSeq genetic annotation data for the target organism
- Empirical metabolite intensity table sampled over time intervals (temporal profiles)
- Boundary flux table (intra-/extracellular fluxes, e.g., thymine production from thymidine; isotope labeling patterns)
- Optionally, metabolic network models from the BiGG database

## Outputs

- Species-specific metabolic network with dark (annotated) vs light gray (unannotated/missing) nodes and edges rendered interactively via D3.js
- Flagged list of disconnected nodes and missing-reaction positions (e.g., network_refinement_map.csv)
- Count of missing reactions per pathway for cross-checking against literature (e.g., 18 total, 8 purine/pyrimidine for B. burgdorferi)

## How to apply

Load the 66 base metabolic pathway layouts shipped with MINNO from the KEGG database, then combine them with NCBI-RefSeq genetic annotation data for the target organism (e.g., Borrelia species) to generate organism-specific pathway nodes; annotated reactions render as dark nodes/edges and unannotated/missing ones as light gray nodes/edges. Load the empirical metabolite intensity and boundary flux table produced by upstream boundary-flux estimation and map each detected metabolite onto its corresponding KEGG compound node in the browser-based D3.js visualization. Overlay temporal intensity profiles and intra- or extracellular data (boundary fluxes, isotope labeling patterns) to inspect whether flux evidence indicates activity at positions where KEGG lacks a connecting reaction, such as PnP for thymidine-to-thymine conversion or unannotated ribonucleotide reductase in B. parkeri and B. turicatae. Flag disconnected nodes and missing-reaction positions and write the annotated network to network_refinement_map.csv. Validation compares flagged reactions against known gaps: MINNO's refinement should reproduce the 18 missing reactions reported from KEGG, nine supported by primary literature, including the eight missing purine/pyrimidine reactions for B. burgdorferi.

## Related tools

- **MINNO** (Web-based mapping tool that loads KEGG base layouts and RefSeq annotations, then accepts empirical metabolomics data to refine and visualize the species-specific network) — https://github.com/LewisResearchGroup/MINNO
- **D3.js** (JavaScript library MINNO uses to render the dynamic interactive network visualization in web browsers)
- **KEGG** (Source of the 66 base metabolic pathway layouts and compound node identifiers onto which metabolites are mapped)
- **NCBI-RefSeq** (Source of genetic annotation data used to generate organism-specific pathway nodes)

## Evaluation signals

- The refinement map reproduces the 18 missing reactions reported from the KEGG database, of which nine are supported by primary literature
- For B. burgdorferi, the purine/pyrimidine subset yields eight missing reactions, matching the reported per-pathway counts (purine plus four pyrimidine reactions, all pyrimidine ones literature-supported)
- Boundary flux evidence correctly flags PnP activity from thymine production out of thymidine and an unannotated ribonucleotide reductase in B. parkeri and B. turicatae
- Every empirically detected metabolite resolves to exactly one corresponding KEGG compound node in the species-specific network; unmatched metabolites are surfaced as disconnected nodes
- Annotated reactions render as dark nodes/edges and unannotated/missing ones as light gray nodes/edges in the D3.js view, allowing visual confirmation of the annotation state

## Limitations

- Inference of missing reactions is visual/evidence-based, without complex mathematical modeling, so flagged reactions require follow-up searches for corresponding genes and proteins using experimental or bioinformatics methods
- MINNO ships with only 66 KEGG base metabolic pathways, covering primary metabolic pathways; pathways outside this set require the BiGG database or other network sources
- Some flagged reactions could only be confirmed by subsequent publications (six of eight purine reactions), so not all flags are immediately literature-supported
- Supporting results reside in supplementary Table S1, which is not included in the main text

## Evidence
