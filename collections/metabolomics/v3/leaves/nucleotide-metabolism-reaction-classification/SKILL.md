---
name: nucleotide-metabolism-reaction-classification
description: Use when MINNO (or equivalent KEGG-vs-metabolomics comparison) has surfaced
  candidate missing reactions in nucleotide or nicotinate pathways and they must be
  assigned to a pathway class and a support tier before writing a gap table or report.
when_to_use_negative:
- When the organism's metabolic network has no empirical metabolomics / boundary-flux
  layer and the goal is pure in-silico gap-filling from genome annotation alone (MINNO's
  key value is metabolite-derived evidence, not purely genomic prediction).
- When the pathway of interest lies outside the nucleotide (purine/pyrimidine) or
  nicotinate branches that MINNO's 66 base KEGG pathways cover; the article's missing-reaction
  evidence is specific to these classes.
- When the user needs a fully resolved stoichiometric model rather than a classification
  of candidate missing reactions — this skill produces an annotated gap table, not
  a curated FBA-ready model.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3407
tools:
- name: MINNO
  role: Web-based mapping tool that takes empirical metabolomics data and genomic
    data to refine the KEGG metabolic network and surface candidate missing reactions;
    its boundary-flux visualization is the upstream evidence the classification is
    built on.
  repo: https://github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Reference sequence database used to assess homology for reactions lacking
    primary-literature support, providing the fallback support tier in Table S1.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:e3eb76096857b4fced265e31eef24035aced5ceff5aeeb3d4ab51b532502500d
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nucleotide-metabolism-reaction-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify reaction gaps found by MINNO between an organism's observed metabolite boundary fluxes and the KEGG base network into purine, pyrimidine, or nicotinate/nicotinamide metabolism, then annotate each missing reaction as literature-supported or homology-supported via NCBI-RefSeq.

## When to use

Use when MINNO (or equivalent KEGG-vs-metabolomics comparison) has surfaced candidate missing reactions in nucleotide or nicotinate pathways and they must be assigned to a pathway class and a support tier before writing a gap table or report.

## When NOT to use

- When the organism's metabolic network has no empirical metabolomics / boundary-flux layer and the goal is pure in-silico gap-filling from genome annotation alone (MINNO's key value is metabolite-derived evidence, not purely genomic prediction).
- When the pathway of interest lies outside the nucleotide (purine/pyrimidine) or nicotinate branches that MINNO's 66 base KEGG pathways cover; the article's missing-reaction evidence is specific to these classes.
- When the user needs a fully resolved stoichiometric model rather than a classification of candidate missing reactions — this skill produces an annotated gap table, not a curated FBA-ready model.

## Inputs

- MINNO-derived list of missing reactions inferred from boundary flux profiles of cultured Borrelia spp. (B. burgdorferi, B. parkeri, B. turicatae)
- KEGG base/ortholog metabolic network layout for purine, pyrimidine, and nicotinate metabolism
- NCBI-RefSeq genetic annotation data for the organism(s) under study
- Primary literature records (PubMed or equivalent) supporting or refuting each candidate missing reaction

## Outputs

- Normalized missing-reactions table (e.g. missing_reactions_table.csv) with one row per gap: reaction identifier, pathway class (purine / pyrimidine / nicotinate), species column(s), and support tier (primary literature vs NCBI-RefSeq homology)
- Reaction-gap count summary reconcilable to the published breakdown (18 total KEGG-missing reactions, 9 literature-supported)

## How to apply

After MINNO infers missing reactions by comparing temporal boundary fluxes (e.g. thymine produced from thymidine implying pyrimidine-nucleoside phosphorylase) against the KEGG base network, assign each gap to purine, pyrimidine, or nicotinate metabolism using the pathway labels already present in the MINNO/KEGG layout. For each classified reaction, decide its support tier: search the primary literature for direct confirmation (e.g. publications confirming missing purine reactions of B. burgdorferi, or literature backing the four missing pyrimidine reactions), and fall back to homology evidence from the NCBI-RefSeq database (as reported in Table S1 for the remaining reactions). Record per-species presence (B. burgdorferi vs B. parkeri/B. turicatae) and pathway class in a normalized table so the counts reconcile with the published breakdown: 8 purine gaps for B. burgdorferi (6 literature-confirmed), 6 purine gaps shared by B. parkeri/B. turicatae, 4 pyrimidine gaps for B. burgdorferi (all literature-supported), 2 pyrimidine gaps shared by B. parkeri/B. turicatae, and 1 nicotinate gap shared by all three species, totaling 18 KEGG-missing reactions with 9 literature-supported. The classification output is the artifact against which a future agent should check species/pathway consistency and the literature-vs-homology split.

## Related tools

- **MINNO** (Web-based mapping tool that takes empirical metabolomics data and genomic data to refine the KEGG metabolic network and surface candidate missing reactions; its boundary-flux visualization is the upstream evidence the classification is built on.) — https://github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Reference sequence database used to assess homology for reactions lacking primary-literature support, providing the fallback support tier in Table S1.)

## Examples

```
python parse_minno_gaps.py --si Table_S1.xlsx --species Borrelia_burgdorferi Borrelia_parkeri Borrelia_turicatae --out missing_reactions_table.csv
```

## Evaluation signals

- Row count and pathway-class distribution must reconcile with the published breakdown: 8 purine + 4 pyrimidine gaps for B. burgdorferi, 6 purine + 2 pyrimidine gaps shared by B. parkeri/B. turicatae, and 1 nicotinate gap shared by all three species (18 total).
- Literature-support flag counts must sum to 9 of the 18 gaps; the remaining 9 rely on NCBI-RefSeq homology and are explicitly flagged as such.
- Every row must carry a non-empty reaction identifier traceable to the KEGG base network or MINNO's Table S1.
- Species column must be populated consistently with the MINNO boundary-flux evidence (e.g. B. burgdorferi-only vs shared-with-B. parkeri/B. turicatae).
- Any reaction labeled literature-supported must have a citable source; if only RefSeq homology exists, the support tier must be downgraded accordingly.

## Limitations

- The article notes 'No version number stated for the MINNO software', so results may vary between software builds and must be tied to the specific MINNO deposit used.
- Supporting results reside in supplementary Table S1, which is not included in the article text itself; the classification therefore depends on access to the SI or the MINNO GitHub deposit.
- No changelog is present in the source content, making it hard to trace changes to the pathway assignments over time.
- The hybrid genomics-metabolomics approach was validated on only three Borrelia species; generalization to organisms whose nucleotide metabolism differs substantially is not established here.

## Evidence

- [discussion] Reference to SI Table S1 not present in article body: [evidence span withheld — anchor status: not_found]
