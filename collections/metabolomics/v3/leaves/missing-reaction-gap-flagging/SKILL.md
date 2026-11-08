---
name: missing-reaction-gap-flagging
description: Use when KEGG+RefSeq-derived organism-specific networks (e.g. Borrelia)
  leave activity without an annotated reaction — boundary fluxes indicate flux through
  a node pair absent from KEGG.
when_to_use_negative:
- Input is a fully reconstructed FBA model with all reactions already annotated —
  boundary-flux gap flagging is for refining KEGG/RefSeq-derived networks, not re-validating
  curated models.
- No empirical metabolomics data (temporal intensities or boundary fluxes) is available
  — without flux/intensity evidence, flagging relies purely on annotations and cannot
  infer missing reactions.
- The target organism is not present in KEGG (e.g. a truly novel isolate with no ortholog
  layout) — MINNO's base pathways would not map.
- If the goal is functional pathway enrichment or differential metabolite analysis
  rather than structural gap detection, use pathway-activity tools instead.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MINNO
  role: Web-based mapping tool that takes empirical metabolomics data to refine metabolic
    networks and flag missing reactions from KEGG
  repo: https://github.com/LewisResearchGroup/MINNO
- name: D3.js
  role: JavaScript library MINNO uses to create the dynamic interactive network visualizations
    in web browsers
- name: KEGG
  role: Source of the 66 base metabolic pathway layouts and ortholog data used to
    build the organism-specific network
- name: NCBI-RefSeq
  role: Source of genetic annotation data to generate organism-specific pathway nodes
    and confirm homology of missing reactions (Table S1)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:52117ca9e82edce0a9a445eaa853e4edb87d16718acd5eff9477ebfa457b49ea
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# missing-reaction-gap-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Flags missing or unannotated reactions in a KEGG-derived metabolic network by overlaying empirical, time-resolved metabolite intensity profiles and boundary fluxes onto the species-specific pathway graph, exposing disconnected nodes and light-gray edges that represent candidate gaps.

## When to use

Use when KEGG+RefSeq-derived organism-specific networks (e.g. Borrelia) leave activity without an annotated reaction — boundary fluxes indicate flux through a node pair absent from KEGG.

## When NOT to use

- Input is a fully reconstructed FBA model with all reactions already annotated — boundary-flux gap flagging is for refining KEGG/RefSeq-derived networks, not re-validating curated models.
- No empirical metabolomics data (temporal intensities or boundary fluxes) is available — without flux/intensity evidence, flagging relies purely on annotations and cannot infer missing reactions.
- The target organism is not present in KEGG (e.g. a truly novel isolate with no ortholog layout) — MINNO's base pathways would not map.
- If the goal is functional pathway enrichment or differential metabolite analysis rather than structural gap detection, use pathway-activity tools instead.

## Inputs

- KEGG base/ortholog pathway layout (66 primary metabolic pathways shipped with MINNO)
- NCBI-RefSeq genetic annotation data for the target species
- Temporal metabolite intensity profiles from time-sampled in-vitro cultures
- Boundary flux data (intra- or extracellular) from the boundary-flux estimator step
- Optionally, BiGG metabolic network models

## Outputs

- Annotated network visualization in the web browser (D3.js) with dark nodes/edges for annotated reactions and light gray nodes/edges for unannotated/missing ones
- List of flagged missing reactions and disconnected nodes (e.g. 18 missing reactions from KEGG for the Borrelia study, 9 literature-supported)
- network_refinement_map.csv summarizing flagged gaps

## How to apply

Load the 66 base metabolic pathway layouts shipped with MINNO, combine them with NCBI-RefSeq genetic annotation data to generate an organism-specific network, and map empirical metabolite intensities and boundary flux data (intra- or extracellular) onto each KEGG compound node. Annotated reactions render as dark nodes/edges; where temporal intensity or boundary-flux evidence indicates activity but KEGG lacks a connecting reaction (e.g. thymidine→thymine via PnP, or an unannotated ribonucleotide reductase in B. parkeri and B. turicatae), flag the position as a missing reaction and record the disconnected node. Users can then search for genes and proteins corresponding to the flagged reaction using experimental or bioinformatics methods; validation is cross-checking flagged gaps against literature (e.g. purine/pyrimidine reactions later confirmed by publications, tracked in Table S1 against NCBI-RefSeq homology).

## Related tools

- **MINNO** (Web-based mapping tool that takes empirical metabolomics data to refine metabolic networks and flag missing reactions from KEGG) — https://github.com/LewisResearchGroup/MINNO
- **D3.js** (JavaScript library MINNO uses to create the dynamic interactive network visualizations in web browsers)
- **KEGG** (Source of the 66 base metabolic pathway layouts and ortholog data used to build the organism-specific network)
- **NCBI-RefSeq** (Source of genetic annotation data to generate organism-specific pathway nodes and confirm homology of missing reactions (Table S1))

## Examples

```
minno --kegg-base 66-pathways --refseq-species B_burgdorferi --metabolites intensities.csv --boundary-fluxes fluxes.csv --out network_refinement_map.csv
```

## Evaluation signals

- Reproduces the reported 18 missing reactions from KEGG, with 9 supported by primary literature
- Purine/pyrimidine subset matches published counts: 8 missing reactions for B. burgdorferi (6 purine, 4 pyrimidine literature-supported per Table S1)
- Flagged PnP gap (thymidine→thymine) is consistent with boundary flux evidence of thymine production
- B. parkeri and B. turicatae show an unannotated ribonucleotide reductase consistent with boundary flux profiles
- Flagged reactions show good homology against NCBI-RefSeq (Table S1 cross-check)

## Limitations

- No version number stated for MINNO in the article; behavior may vary with software releases.
- Supporting reaction details reside in supplementary Table S1 not included in the main text.
- Missing-reaction flagging is qualitative (boundary flux/intensity heuristic), not a mathematically modeled prediction — the article emphasizes that inference happens 'without complex mathematical modeling'.
- Flagged gaps require follow-up experimental or bioinformatics gene/protein searches; MINNO itself does not identify the gene.
- Performance depends on KEGG completeness; 18 gaps out of a base database of 8794+ species pathways were found for Borrelia species alone.

## Evidence
