---
name: refseq-gene-annotation-integration
description: Use when building a species-specific metabolic network for an organism
  whose KEGG map lacks complete reaction coverage, by combining NCBI-RefSeq gene annotations
  with the 66 base KEGG pathway layouts shipped in MINNO.
when_to_use_negative:
- When the target organism already has a fully populated, literature-curated genome-scale
  metabolic model (e.g., a BiGG model) that requires no further gap-filling
- When the goal is pure genome annotation rather than metabolic network refinement,
  since MINNO is specifically designed for metabolomics-driven network editing
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: MINNO
  role: Web-based mapping tool that overlays NCBI-RefSeq annotations onto KEGG base
    metabolic pathway layouts to generate and refine organism-specific metabolic networks
  repo: https://github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Source of genetic annotation data for the target organism used to build the
    species-specific network
- name: KEGG
  role: Supplies the base metabolic pathway layouts (66 pathways) that serve as the
    starting scaffold for the species-specific network
- name: D3.js
  role: JavaScript library used by MINNO to render the dynamic, interactive network
    visualization in web browsers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:0ee368f5ed628e7033b25623255678bd4b7ede289cc36e68be70c6d5da33d337
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# refseq-gene-annotation-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrate NCBI-RefSeq genetic annotation data with KEGG base metabolic pathway layouts inside the MINNO visualization tool to generate organism-specific metabolic networks whose annotated reactions appear as dark nodes/edges and unannotated or missing ones as light gray nodes/edges, enabling flagging of missing reactions.

## When to use

Use when building a species-specific metabolic network for an organism whose KEGG map lacks complete reaction coverage, by combining NCBI-RefSeq gene annotations with the 66 base KEGG pathway layouts shipped in MINNO.

## When NOT to use

- When the target organism already has a fully populated, literature-curated genome-scale metabolic model (e.g., a BiGG model) that requires no further gap-filling
- When the goal is pure genome annotation rather than metabolic network refinement, since MINNO is specifically designed for metabolomics-driven network editing

## Inputs

- NCBI-RefSeq genetic annotation data for the target organism (e.g., a Borrelia species)
- KEGG base/ortholog metabolic network layout data (66 base pathways covering all primary metabolic pathways)
- Temporal metabolite intensity profiles from time-course cultures
- Boundary flux estimates for intra- or extracellular metabolites

## Outputs

- Organism-specific metabolic network rendered in MINNO with annotated reactions as dark nodes/edges and unannotated or missing ones as light gray nodes/edges
- network_refinement_map.csv listing flagged missing reactions and disconnected nodes
- Missing-reaction list suitable for cross-checking against the primary literature

## How to apply

Load the 66 base metabolic pathway layouts from the KEGG database that ship with MINNO, then overlay NCBI-RefSeq genetic annotation data for the target organism (e.g., a Borrelia species) to generate organism-specific pathway nodes. Boundary fluxes and temporal metabolite intensities are then incorporated onto the resulting network so that annotated reactions appear as dark nodes/edges and unannotated or missing ones appear as light gray nodes/edges, making disconnected nodes and gaps visually tractable. Where flux or isotope-labeling evidence indicates activity but KEGG lacks a connecting reaction (e.g., pyrimidine-nucleoside phosphorylase converting thymidine to thymine, or an unannotated ribonucleotide reductase in B. parkeri and B. turicatae), flag the missing reaction. Write the annotated network, including flagged missing reactions and disconnected nodes, to a refinement map such as network_refinement_map.csv. Validate the refinement map by counting how many flagged reactions are supported by primary literature — in the source study, 18 missing reactions were identified and nine were literature-supported.

## Related tools

- **MINNO** (Web-based mapping tool that overlays NCBI-RefSeq annotations onto KEGG base metabolic pathway layouts to generate and refine organism-specific metabolic networks) — https://github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Source of genetic annotation data for the target organism used to build the species-specific network)
- **KEGG** (Supplies the base metabolic pathway layouts (66 pathways) that serve as the starting scaffold for the species-specific network)
- **D3.js** (JavaScript library used by MINNO to render the dynamic, interactive network visualization in web browsers)

## Examples

```
Open https://lewisresearchgroup.github.io/MINNO/, upload the 66-KEGG-pathway layout, the NCBI-RefSeq annotation, and the boundary-flux/metabolite-intensity table, then export network_refinement_map.csv.
```

## Evaluation signals

- All annotated reactions in the generated organism-specific network appear as dark nodes/edges, and unannotated or missing reactions appear as light gray nodes/edges
- network_refinement_map.csv reproduces the expected set of missing reactions — e.g., 18 missing reactions from the KEGG database, of which nine were literature-supported
- Species-specific counts per pathway match expectations (e.g., eight missing reactions in B. burgdorferi purine/pyrimidine metabolism, four in pyrimidine metabolism)
- Every flagged missing reaction has a biological rationale in the boundary flux or metabolite intensity data (e.g., thymine production from thymidine implying pyrimidine-nucleoside phosphorylase)

## Limitations

- The refinement depends on the quality and completeness of the NCBI-RefSeq annotations; poorly annotated organisms may yield many false gaps
- Missing-reaction flags rely on boundary flux or metabolite intensity evidence, so organisms with sparse or noisy metabolomics data may under- or over-report gaps
- The refinement map does not automatically validate whether the flagged reaction is biologically real — cross-checking against primary literature is required

## Evidence
