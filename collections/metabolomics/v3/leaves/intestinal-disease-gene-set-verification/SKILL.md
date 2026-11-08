---
name: intestinal-disease-gene-set-verification
description: Use when a task requires confirming which genes (e.g. STK11, CFTR, BMPR1A,
  SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) are associated with multiple intestinal
  diseases in GutUDB.
when_to_use_negative:
- The analysis targets diseases outside GutUDB's scope — the database covers 56 intestinal
  diseases across six species, so non-intestinal disease-gene questions are out of
  scope
- The task requires de novo identification of shared genes rather than verification
  of the published list (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC,
  MEFV) already reported for these nine diseases
- The task concerns drug-disease interactions or other GutUDB modules (e.g. the 21,984
  drug-disease interactions in THERAPY) rather than disease-gene associations
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3324
tools:
- name: GutUDB
  role: Source database queried for disease-gene associations covering the nine main
    intestine-related diseases and the ten candidate common genes
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:ac0f483e6dcb847bde943ee8da6a9f87ff9406b56ccee134d5934d28a7a4d122
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3324
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# intestinal-disease-gene-set-verification

## Summary

Verifies a claimed shared-gene list across the nine main intestine-related diseases using GutUDB's 260,790 disease-gene associations, and annotates disease-specific exemplar genes such as STK11 for Peutz-Jeghers syndrome and SMAD4/BMPR1A for Juvenile Polyposis Syndrome.

## When to use

Use when a task requires confirming which genes (e.g. STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) are associated with multiple intestinal diseases in GutUDB.

## When NOT to use

- The analysis targets diseases outside GutUDB's scope — the database covers 56 intestinal diseases across six species, so non-intestinal disease-gene questions are out of scope
- The task requires de novo identification of shared genes rather than verification of the published list (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) already reported for these nine diseases
- The task concerns drug-disease interactions or other GutUDB modules (e.g. the 21,984 drug-disease interactions in THERAPY) rather than disease-gene associations

## Inputs

- GutUDB disease-gene association dataset (subset of the 260,790 disease-gene associations)
- List of nine main intestine-related diseases (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, small intestine cancer)
- Candidate common-gene list: STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV

## Outputs

- Verified gene-to-disease mapping table showing each common gene linked to all nine intestinal diseases
- Annotated exemplar records: STK11 → Peutz-Jeghers syndrome; SMAD4 and BMPR1A → Juvenile Polyposis Syndrome

## How to apply

Extract the gene-disease association records from the GutUDB disease-gene dataset covering the nine main intestine-related diseases (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, and small intestine cancer) and the ten candidate common genes. Cross-tabulate the associations to confirm each of STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV is linked to every disease in the list. Annotate the exemplar cases: STK11 as the gene for Peutz-Jeghers syndrome and SMAD4/BMPR1A as the genes for Juvenile Polyposis Syndrome. Write the verified gene-to-disease mapping, including the exemplar annotations, to the output table.

## Related tools

- **GutUDB** (Source database queried for disease-gene associations covering the nine main intestine-related diseases and the ten candidate common genes) — https://github.com/Ansanqi/GutUDB

## Evaluation signals

- Each of the ten genes (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) appears in the cross-tabulation as associated with all nine diseases in the list
- The output table contains exemplar annotations mapping STK11 to Peutz-Jeghers syndrome and SMAD4/BMPR1A to Juvenile Polyposis Syndrome
- The number of association records queried is consistent with the 260,790 disease-gene associations embedded in GutUDB
- Every disease listed in the output matches exactly one of the nine main intestine-related diseases — no extra or missing diseases
- Gene symbols in the output use standard HGNC-style symbols matching the source list

## Limitations

- The common-gene list is limited to the ten genes reported in GutUDB for the nine main intestine-related diseases; genes outside this curated set are not covered by this verification workflow
- Disease-gene associations are curated records embedded in GutUDB, not experimentally re-derived per task, so verification is bounded by the curation of the database
- GutUDB's repository shows no changelog ('_No changelog found._'), so dataset version drift between queries cannot be tracked from the repo alone

## Evidence

- [abstract] 260,790 disease-gene associations are embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [results] Common genes and exemplars for the nine main intestine-related diseases: [evidence span withheld — anchor status: not_found]
- [abstract] Scope of GutUDB disease coverage: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [discussion] No changelog in repository: "_No changelog found._"
