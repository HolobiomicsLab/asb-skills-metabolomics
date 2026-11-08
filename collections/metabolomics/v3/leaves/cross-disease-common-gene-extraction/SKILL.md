---
name: cross-disease-common-gene-extraction
description: Use when you need to determine whether a defined group of intestine-related
  diseases (e.g., colon cancer, colorectal cancer, IBD, constipation, diarrhea, ileus)
  shares common genes, and which genes are flagged as etiologically linked to specific
  syndromes.
when_to_use_negative:
- The diseases of interest are outside GutUDB's 56 covered intestinal diseases (e.g.,
  non-intestinal cancers or systemic conditions)
- You need drug-disease or therapeutic interactions — use GutUDB's THERAPY module
  instead of the disease-gene association data
- You need variant-level genomic detail (SNPs, CNVs, SVs, mutated genes) rather than
  gene-level disease associations
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GutUDB
  role: Source database queried for disease-gene association records across the nine
    main intestine-related diseases; its Query and Download functionalities retrieve
    the association records for cross-tabulation
  repo: Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:c24c5fffb4debd24adce7abef8a149308462020907f3320e32988877134ba3fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-disease-common-gene-extraction

## Summary

Extracts the set of genes shared across multiple intestinal diseases from GutUDB's 260,790 disease-gene associations, then annotates disease-specific exemplar genes (e.g., STK11 for Peutz-Jeghers syndrome) to produce a verified gene-to-disease mapping.

## When to use

Use when you need to determine whether a defined group of intestine-related diseases (e.g., colon cancer, colorectal cancer, IBD, constipation, diarrhea, ileus) shares common genes, and which genes are flagged as etiologically linked to specific syndromes.

## When NOT to use

- The diseases of interest are outside GutUDB's 56 covered intestinal diseases (e.g., non-intestinal cancers or systemic conditions)
- You need drug-disease or therapeutic interactions — use GutUDB's THERAPY module instead of the disease-gene association data
- You need variant-level genomic detail (SNPs, CNVs, SVs, mutated genes) rather than gene-level disease associations

## Inputs

- GutUDB disease-gene association dataset (260,790 associations covering 56 intestinal diseases)
- List of target intestine-related diseases (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, small intestine cancer)
- Candidate common gene list (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV)

## Outputs

- Cross-tabulated gene-disease association matrix confirming each gene is linked to every target disease
- Annotated gene-to-disease mapping table with exemplar syndrome annotations (STK11 → Peutz-Jeghers syndrome; SMAD4/BMPR1A → Juvenile Polyposis Syndrome)

## How to apply

Query GutUDB's disease-gene association dataset (which embeds 260,790 disease-gene associations across 56 intestinal diseases) for the nine main intestine-related diseases and the ten candidate common genes (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV). Cross-tabulate the retrieved association records so that each gene-disease pair is explicitly confirmed, rather than relying on per-disease gene lists. Annotate exemplar cases where the database ties a gene to a specific syndrome: STK11 with Peutz-Jeghers syndrome and SMAD4/BMPR1A with Juvenile Polyposis Syndrome. Write the final gene-to-disease mapping, including exemplar annotations, to an output table. Use GutUDB's Query and Download functionalities to pull the underlying association records.

## Related tools

- **GutUDB** (Source database queried for disease-gene association records across the nine main intestine-related diseases; its Query and Download functionalities retrieve the association records for cross-tabulation) — Ansanqi/GutUDB

## Evaluation signals

- Each of the ten genes (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) appears in the association records for all nine target diseases
- STK11 is explicitly annotated as the exemplar gene for Peutz-Jeghers syndrome in the output mapping
- SMAD4 and BMPR1A are jointly annotated as exemplar genes for Juvenile Polyposis Syndrome
- The output table covers exactly nine diseases and ten genes with no unverified gene-disease pairs
- All records trace back to GutUDB's embedded set of 260,790 disease-gene associations

## Limitations

- The common-gene list reflects associations embedded in GutUDB as published in 2024; it may not capture associations added in later database updates (the repo shows no changelog)
- Commonality across diseases does not by itself establish causality — exemplar annotations (STK11/Peutz-Jeghers, SMAD4/BMPR1A/Juvenile Polyposis) carry etiological weight, but other shared genes may be associative only
- Coverage is limited to GutUDB's 56 intestinal diseases across six species; queries outside that scope will fail or return partial data

## Evidence

- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [other] common genes STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV, with STK11 linked to Peutz-Jeghers syndrome and SMAD4/BMPR1A to Juvenile Polyposis Syndrome: [evidence span withheld — anchor status: not_found]
