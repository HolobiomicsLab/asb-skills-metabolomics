---
name: reaction-evidence-source-categorization
description: Decision support for classifying newly inferred missing metabolic reactions
  (e.g. purine/pyrimidine gaps in Borrelia) into literature-backed versus homology-backed
  evidence categories.
when_to_use_negative:
- Input reactions are already annotated in KEGG with full pathway membership — no
  evidence categorization is needed since there are no missing-reaction candidates
  to classify.
- The organism's genome lacks NCBI-RefSeq annotation and no primary literature exists
  for candidate reactions — neither evidence class can be established and the categorization
  cannot be completed.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: MINNO
  role: Produces the list of reactions missing from the KEGG database that are subsequently
    categorized into literature versus NCBI-RefSeq evidence classes.
  repo: https://github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Provides the homology database used to assign the 'NCBI-RefSeq homology' evidence
    label to reactions lacking primary-literature support.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:39c1a85a2361ff2a160ec9e67153c9b56b6673cd22506e9a75abd2f114cd6941
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
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reaction-evidence-source-categorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Categorizes each reaction flagged as missing from the KEGG database into primary-literature-supported versus NCBI-RefSeq-homology-supported evidence classes, turning MINNO's gap-detection output into a curated, source-traceable reaction table.

## When to use

Decision support for classifying newly inferred missing metabolic reactions (e.g. purine/pyrimidine gaps in Borrelia) into literature-backed versus homology-backed evidence categories.

## When NOT to use

- Input reactions are already annotated in KEGG with full pathway membership — no evidence categorization is needed since there are no missing-reaction candidates to classify.
- The organism's genome lacks NCBI-RefSeq annotation and no primary literature exists for candidate reactions — neither evidence class can be established and the categorization cannot be completed.

## Inputs

- Table of missing reactions identified from the KEGG database (18 reactions with reaction identifiers)
- Primary literature sources confirming individual missing reactions (e.g. Table S1 entries for purine and pyrimidine reactions)
- NCBI-RefSeq database homology data for the organism's genome/gene annotations

## Outputs

- Categorized reaction table mapping each missing reaction identifier to an evidence-source label ('literature' or 'NCBI-RefSeq homology')
- Final split summary (e.g. 9 literature-supported and 9 NCBI-RefSeq-supported of 18 total)

## How to apply

Start from the compiled table of missing reactions that MINNO identified from the KEGG database (18 reactions in the Borrelia case). For each reaction, first check primary literature confirmation — in the B. burgdorferi purine/pyrimidine example, six purine reactions were confirmed by prior publications and all four pyrimidine reactions were literature-supported — and label these 'literature'. For every remaining reaction, verify homology against the NCBI-RefSeq database and label these 'NCBI-RefSeq homology'; the article's Table S1 reports the remaining nine reactions 'show good homology as in the NCBI-RefSeq database'. Summarize the split as a final tally (nine literature-supported and nine NCBI-RefSeq-supported) and write the categorized table with reaction identifiers and evidence-source labels to a CSV file.

## Related tools

- **MINNO** (Produces the list of reactions missing from the KEGG database that are subsequently categorized into literature versus NCBI-RefSeq evidence classes.) — https://github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Provides the homology database used to assign the 'NCBI-RefSeq homology' evidence label to reactions lacking primary-literature support.)

## Evaluation signals

- The categorized table must contain exactly 18 reactions for the B. burgdorferi purine/pyrimidine case.
- The final tally must split as 9 literature-supported and 9 NCBI-RefSeq-homology-supported reactions.
- Each reaction identifier must carry exactly one evidence-source label ('literature' or 'NCBI-RefSeq homology') with no unclassified entries.
- Purine reactions with published confirmation (six) and all four pyrimidine reactions must be labeled 'literature', matching Table S1.
- Output CSV must include reaction identifiers and evidence-source labels as separate fields.

## Limitations

- Supporting results reside in supplementary Table S1, which is not included in the main text — cross-checking the full literature and homology assignments requires accessing the supplementary file.
- The NCBI-RefSeq homology check depends on the organism being represented in the RefSeq database; divergent organisms may lack adequate homology to support a reaction.
- Only two evidence classes are used ('literature' and 'NCBI-RefSeq homology'); reactions supported by weaker or partial evidence outside these categories cannot be captured by this skill.

## Evidence
