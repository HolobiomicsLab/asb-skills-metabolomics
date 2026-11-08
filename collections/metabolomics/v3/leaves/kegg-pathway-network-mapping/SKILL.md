---
name: kegg-pathway-network-mapping
description: Use when building species-specific metabolic networks from KEGG layouts
  and NCBI-RefSeq annotations and refining them with empirical metabolomics data to
  flag missing or disconnected reactions.
when_to_use_negative:
- Input is a fully annotated, closed metabolic network (e.g. a validated genome-scale
  model in BiGG) where no missing-reaction discovery is needed
- Analysis goal is purely genomic annotation without metabolomics overlay, since the
  method requires empirical flux/intensity data to infer missing reactions
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MINNO
  role: Web-based mapping tool that takes empirical metabolomics data to refine metabolic
    networks; loads KEGG layouts and RefSeq annotations, and overlays temporal metabolite
    intensity and boundary flux data to flag missing reactions
  repo: github.com/LewisResearchGroup/MINNO
- name: D3.js
  role: JavaScript library used by MINNO to create dynamic interactive visualizations
    of the network in web browsers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:e6f4f30d106c9d69b4f366e9a17bd38513cee059a47ad05b3dee1ec755e5cf39
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kegg-pathway-network-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps empirical metabolite intensity and boundary-flux data onto organism-specific metabolic networks built from KEGG base pathway layouts plus NCBI-RefSeq annotations, then flags missing or disconnected reactions. This enables empirical refinement of genome-predicted networks where database annotations are incomplete.

## When to use

Use when building species-specific metabolic networks from KEGG layouts and NCBI-RefSeq annotations and refining them with empirical metabolomics data to flag missing or disconnected reactions.

## When NOT to use

- Input is a fully annotated, closed metabolic network (e.g. a validated genome-scale model in BiGG) where no missing-reaction discovery is needed
- Analysis goal is purely genomic annotation without metabolomics overlay, since the method requires empirical flux/intensity data to infer missing reactions

## Inputs

- KEGG base metabolic pathway layouts (66 pathways covering all primary metabolic pathways, bundled with MINNO)
- NCBI-RefSeq genetic annotation data for the target organism
- Temporal metabolite intensity profiles and intra- or extracellular boundary-flux data (e.g. thymine-to-thymidine conversion, isotope labeling patterns)

## Outputs

- Species-specific metabolic network with annotated reactions shown as dark nodes/edges and unannotated/missing reactions shown as light gray nodes/edges
- Flagged missing-reaction positions and disconnected nodes (e.g. PnP for thymidine-to-thymine conversion; unannotated ribonucleotide reductase in B. parkeri and B. turicatae)

## How to apply

Load the 66 base metabolic pathway layouts from the KEGG database shipped with MINNO, then load NCBI-RefSeq genetic annotation data for the target organism to generate species-specific pathway nodes. Next, map each empirically detected metabolite from the temporal intensity/boundary-flux table onto its corresponding KEGG compound node. Identify positions where flux evidence indicates activity (e.g. thymine produced from thymidine implying pyrimidine-nucleoside phosphorylase, or ribonucleotide reductase inferred in B. parkeri and B. turicatae) but KEGG lacks a connecting reaction, and flag these as missing reactions or disconnected nodes. Rationale: genetically constructed networks alone cannot capture species-specific metabolism, so overlaying temporal metabolite profiles and boundary fluxes onto the KEGG/RefSeq-derived network empirically refines it and reveals missing reactions that can then be validated by literature or further bioinformatics search.

## Related tools

- **MINNO** (Web-based mapping tool that takes empirical metabolomics data to refine metabolic networks; loads KEGG layouts and RefSeq annotations, and overlays temporal metabolite intensity and boundary flux data to flag missing reactions) — github.com/LewisResearchGroup/MINNO
- **D3.js** (JavaScript library used by MINNO to create dynamic interactive visualizations of the network in web browsers)

## Examples

```
Open MINNO at https://lewisresearchgroup.github.io/MINNO/, load KEGG base pathway layout + NCBI-RefSeq annotation, then upload the temporal metabolite/boundary-flux CSV to flag missing reactions.
```

## Evaluation signals

- The refinement map correctly reproduces the 18 missing reactions reported from the KEGG database
- Of those 18 missing reactions, at least nine are supported by primary literature
- For B. burgdorferi, eight missing purine/pyrimidine reactions are identified, with six purine and four pyrimidine reactions confirmed by publications
- Annotated reactions appear as dark nodes/edges and unannotated/missing ones as light gray nodes/edges, matching the species-specific network built from KEGG and RefSeq

## Limitations

- No version number is stated for the MINNO software
- The MINNO tool relies on KEGG base pathways and NCBI-RefSeq annotations, so reactions absent from both databases must be inferred empirically from boundary flux profiles, which is indirect evidence rather than direct gene confirmation

## Evidence
