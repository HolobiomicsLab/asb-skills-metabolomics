---
name: hereditary-polyposis-syndrome-gene-annotation
description: Use when a gene-disease mapping for the nine main intestine-related diseases
  (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation,
  diarrhea, ileus, inflammatory bowel disease, small intestine cancer) must be annotated
  with etiologic syndrome-level exemplars.
when_to_use_negative:
- The analysis targets non-intestinal diseases outside GutUDB's coverage of 56 intestinal
  diseases across six species
- The task requires variant-level detail (SNPs, CNVs, structural variations) rather
  than curated disease-gene associations
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GutUDB
  role: Source database queried and downloaded for disease-gene association records
    used to build and verify the gene-to-disease mapping and exemplar annotations
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:2e3ffc099e03fbaafadd06a08b3838c26f6d1ec1d0c3c907d381b73fc5491c39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hereditary-polyposis-syndrome-gene-annotation

## Summary

Annotates the shared intestine-disease gene list (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) from GutUDB's 260,790 disease-gene associations with exemplar hereditary polyposis syndrome assignments — STK11 to Peutz-Jeghers syndrome and SMAD4/BMPR1A to Juvenile Polyposis Syndrome.

## When to use

Use when a gene-disease mapping for the nine main intestine-related diseases (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, small intestine cancer) must be annotated with etiologic syndrome-level exemplars.

## When NOT to use

- The analysis targets non-intestinal diseases outside GutUDB's coverage of 56 intestinal diseases across six species
- The task requires variant-level detail (SNPs, CNVs, structural variations) rather than curated disease-gene associations

## Inputs

- GutUDB disease-gene association dataset (subset of the 260,790 disease-gene associations)
- List of nine main intestine-related diseases (colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, small intestine cancer)
- List of ten candidate common genes (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV)

## Outputs

- Verified gene-to-disease mapping table confirming each of the ten genes is linked to every disease in the list
- Exemplar annotation table mapping STK11 to Peutz-Jeghers syndrome and SMAD4/BMPR1A to Juvenile Polyposis Syndrome

## How to apply

Extract the gene-disease association records from the GutUDB disease-gene dataset (which embeds 260,790 disease-gene associations) covering the nine main intestine-related diseases and the ten candidate common genes. Cross-tabulate the associations to confirm each gene — STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV — is linked to every disease in the list. Then annotate the exemplar cases: STK11 as the gene for Peutz-Jeghers syndrome and SMAD4/BMPR1A as the genes for Juvenile Polyposis Syndrome. Use GutUDB's Query and Download functionalities to retrieve the association table, then write the verified gene-to-disease mapping including the exemplar annotations to the output table.

## Related tools

- **GutUDB** (Source database queried and downloaded for disease-gene association records used to build and verify the gene-to-disease mapping and exemplar annotations) — https://github.com/Ansanqi/GutUDB

## Evaluation signals

- Each of the ten genes (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) appears associated with all nine diseases in the cross-tabulated output table
- STK11 is explicitly annotated as linked to Peutz-Jeghers syndrome in the output
- SMAD4 and BMPR1A are both explicitly annotated as linked to Juvenile Polyposis Syndrome in the output
- Every gene-disease pair in the output table traces back to a record in the GutUDB disease-gene dataset

## Limitations

- The exemplar annotations are limited to the two etiologic exemplars surfaced by GutUDB (STK11–Peutz-Jeghers, SMAD4/BMPR1A–Juvenile Polyposis); other polyposis syndromes are not covered by this skill
- Shared-gene links do not imply causation for every disease; the curated associations reflect GutUDB's embedding of 260,790 disease-gene associations rather than per-disease experimental validation

## Evidence

- [abstract] In total, 260,790 disease‐gene associations were embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [other] the main intestine-related diseases share common genes STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, and MEFV, with STK11 linked to Peutz-Jeghers syndrome: [evidence span withheld — anchor status: not_found]
