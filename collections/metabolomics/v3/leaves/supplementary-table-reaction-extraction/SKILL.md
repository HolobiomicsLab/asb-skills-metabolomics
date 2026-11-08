---
name: supplementary-table-reaction-extraction
description: Use when a paper reports reaction gaps (e.g., 18 missing KEGG reactions
  across Borrelia species) only in supplementary tables, and you need a machine-readable
  per-reaction table annotated with pathway class, species, and literature-support
  flags.
when_to_use_negative:
- The paper reports reaction gaps only in the main text (e.g., boundary-flux profiles)
  without a supplementary table; parse text directly instead.
- You need to re-run the MINNO boundary-flux analysis itself; this skill only curates
  already-published supplementary reaction records.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MINNO
  role: source of the missing-reaction records (Table S1) being extracted and normalized
  repo: github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: database used to annotate reactions that lack primary-literature support
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:08b384ed350b89930fd6a91d400af54a0c6a585d8b27afe7947c2ef9d067de23
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LewisResearchGroup/MINNO
  tools:
  - MINNO
  - NCBI-RefSeq
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# supplementary-table-reaction-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts missing-reaction records from a paper's Supporting Information (e.g., Table S1 of the MINNO/Analytical Chemistry article) into a normalized CSV listing each reaction's pathway class, species-specificity, and literature-support status.

## When to use

Use when a paper reports reaction gaps (e.g., 18 missing KEGG reactions across Borrelia species) only in supplementary tables, and you need a machine-readable per-reaction table annotated with pathway class, species, and literature-support flags.

## When NOT to use

- The paper reports reaction gaps only in the main text (e.g., boundary-flux profiles) without a supplementary table; parse text directly instead.
- You need to re-run the MINNO boundary-flux analysis itself; this skill only curates already-published supplementary reaction records.

## Inputs

- Supporting Information Table S1 (missing-reaction table) from the article SI
- MINNO GitHub deposit (github.com/LewisResearchGroup/MINNO) reaction records

## Outputs

- missing_reactions_table.csv (one row per missing reaction, with pathway class, species columns, and literature-support flag)

## How to apply

Download the Supporting Information file (Table S1) from the article SI and the MINNO GitHub deposit (github.com/LewisResearchGroup/MINNO). Parse the table into one row per missing reaction, capturing reaction identifiers, pathway assignment, and the species columns (B. burgdorferi, B. parkeri, B. turicatae). Normalize pathway labels into purine, pyrimidine, and nicotinate classes, and record in which species each reaction was found missing. Annotate each reaction with a primary-literature support flag, distinguishing publication-supported reactions (e.g., the six confirmed purine reactions and the four literature-backed pyrimidine reactions) from reactions justified by NCBI-RefSeq homology. Cross-check the compiled counts against the reported breakdown (8 purine reactions for B. burgdorferi, 6 purine reactions shared by B. parkeri/B. turicatae, 4 pyrimidine reactions for B. burgdorferi, 2 pyrimidine reactions shared by B. parkeri/B. turicatae, 1 nicotinate reaction shared by all three) and write the normalized table to missing_reactions_table.csv.

## Related tools

- **MINNO** (source of the missing-reaction records (Table S1) being extracted and normalized) — github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (database used to annotate reactions that lack primary-literature support)

## Examples

```
python scripts/extract_missing_reactions.py --si TableS1.xlsx --species B_burgdorferi B_parkeri B_turicatae --out missing_reactions_table.csv
```

## Evaluation signals

- Total row count equals 18 missing reactions (the reported count from KEGG)
- Pathway-class breakdown matches 8 purine (B. burgdorferi), 6 purine (B. parkeri/B. turicatae), 4 pyrimidine (B. burgdorferi), 2 pyrimidine (B. parkeri/B. turicatae), 1 nicotinate (all three)
- Exactly nine rows carry the primary-literature-support flag; the remainder are annotated as NCBI-RefSeq homology-supported
- Every row is assigned to one of the three Borrelia species columns

## Limitations

- Reactions supported only by NCBI-RefSeq homology (Table S1) are not backed by primary literature; do not treat them as experimentally confirmed.
- Supporting results reside in supplementary Table S1, which is not included in the main article section and must be fetched separately.
- No changelog or version number for the underlying MINNO deposit is present in the source content.

## Evidence
