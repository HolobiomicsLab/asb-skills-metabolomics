---
name: kegg-reaction-gap-tabulation
description: Use when a metabolomics-driven network-refinement study reports reaction
  gaps against a reference metabolic database (e.g. KEGG) and these must be normalized
  into a per-reaction table with pathway class, species, and literature-support flags.
when_to_use_negative:
- The target network is already complete in the reference database and no boundary-flux-inferred
  gaps exist to tabulate.
- Only gene-homology annotation (e.g. plain NCBI-RefSeq BLAST results) is available
  without empirical metabolomics/boundary-flux evidence, since the method's core is
  metabolomics-driven gap identification.
- The analysis targets a non-metabolic annotation gap (e.g. missing genes in a pathway
  file) rather than reactions absent from KEGG purine, pyrimidine, or nicotinate metabolism.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MINNO
  role: Visualization tool that combines KEGG base/ortholog network layouts, NCBI-RefSeq
    annotations, and temporal metabolite intensity profiles to infer missing reactions
    via boundary flux analysis; its SI tables are the source for the tabulated gaps.
  repo: https://github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Reference sequence database used to search for genes/proteins corresponding
    to identified missing reactions when primary literature support is unavailable.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:9c63c32341dabc8e325397b7adcab0191dfa1345517afeb425e5f854b072a007
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
  - MINNO
  - NCBI-RefSeq
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kegg-reaction-gap-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulate reactions identified as missing from the KEGG database via MINNO-based boundary-flux inference, recording each reaction's pathway (purine, pyrimidine, nicotinate), the Borrelia species in which it is absent, and whether primary literature supports its existence.

## When to use

Use when a metabolomics-driven network-refinement study reports reaction gaps against a reference metabolic database (e.g. KEGG) and these must be normalized into a per-reaction table with pathway class, species, and literature-support flags.

## When NOT to use

- The target network is already complete in the reference database and no boundary-flux-inferred gaps exist to tabulate.
- Only gene-homology annotation (e.g. plain NCBI-RefSeq BLAST results) is available without empirical metabolomics/boundary-flux evidence, since the method's core is metabolomics-driven gap identification.
- The analysis targets a non-metabolic annotation gap (e.g. missing genes in a pathway file) rather than reactions absent from KEGG purine, pyrimidine, or nicotinate metabolism.

## Inputs

- Supporting Information Table S1 (one row per missing reaction, with pathway and species columns)
- MINNO GitHub deposit (github.com/LewisResearchGroup/MINNO)
- Boundary flux profiles from temporal metabolite intensity data of cultured Borrelia cells
- NCBI-RefSeq genetic annotation data used for homology support of remaining reactions

## Outputs

- missing_reactions_table.csv — normalized per-reaction rows with reaction identifier, pathway class (purine/pyrimidine/nicotinate), species in which the reaction is missing, and primary-literature support flag
- Count summary matching the published gap breakdown (18 missing reactions total, 9 literature-supported)

## How to apply

Start from the paper's Supporting Information Table S1 and the MINNO GitHub deposit (LewisResearchGroup/MINNO), parsing the table into one row per missing reaction and capturing reaction identifiers, pathway assignment, and the species columns for B. burgdorferi, B. parkeri, and B. turicatae. Normalize pathway labels to the purine, pyrimidine, and nicotinate classes and record the species in which each reaction was found missing. Annotate each reaction with a primary-literature support flag, separating reactions whose existence was confirmed by publications (e.g. the six confirmed purine reactions in B. burgdorferi and the four literature-backed pyrimidine reactions) from those justified only by NCBI-RefSeq homology. Cross-check the compiled counts against the reported gap breakdown — 8 purine reactions for B. burgdorferi, 6 purine reactions shared by B. parkeri/B. turicatae, 4 pyrimidine reactions for B. burgdorferi, 2 pyrimidine reactions shared by B. parkeri/B. turicatae, and 1 nicotinate reaction shared by all three species — and write the normalized table to missing_reactions_table.csv. The rationale is that MINNO infers gaps by considering boundary fluxes of cultured cells, so each tabulated row traces back to an empirical flux observation plus a gene/protein homology search rather than to KEGG annotation alone.

## Related tools

- **MINNO** (Visualization tool that combines KEGG base/ortholog network layouts, NCBI-RefSeq annotations, and temporal metabolite intensity profiles to infer missing reactions via boundary flux analysis; its SI tables are the source for the tabulated gaps.) — https://github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Reference sequence database used to search for genes/proteins corresponding to identified missing reactions when primary literature support is unavailable.)

## Examples

```
python parse_si_table.py --si TableS1.xlsx --output missing_reactions_table.csv --pathways purine,pyrimidine,nicotinate --species B_burgdorferi,B_parkeri,B_turicatae
```

## Evaluation signals

- The total number of unique missing reactions across the three Borrelia species must equal 18.
- Exactly 9 of the 18 reactions must carry a primary-literature support flag.
- Pathway class counts must reproduce the published breakdown: 8 purine reactions (B. burgdorferi), 6 purine reactions (B. parkeri/B. turicatae), 4 pyrimidine reactions (B. burgdorferi), 2 pyrimidine reactions (B. parkeri/B. turicatae), and 1 nicotinate reaction shared by all three.
- Every row in missing_reactions_table.csv must trace to a boundary-flux observation recorded in the MINNO visualization (temporal metabolite intensity data from cultured cells), not to a KEGG annotation absence alone.
- Reactions lacking publication support must be flagged as homology-based (NCBI-RefSeq) rather than literature-confirmed.

## Limitations

- The article notes that supporting results for the full reaction list reside in supplementary Table S1, which is not included in the main text — the curator must obtain the SI file to tabulate all 18 reactions.
- MINNO's gap inference is contingent on the empirical metabolomics data and boundary fluxes of the cultured isolates; gaps observed in B. parkeri or B. turicatae may not generalize to other Borrelia strains.
- For reactions not backed by primary literature, the only supporting evidence is homology in the NCBI-RefSeq database, which the authors note is weaker evidence than confirmed publications.
- No version number for the MINNO software is stated in the article, so reproducibility depends on the current state of the GitHub deposit.

## Evidence
