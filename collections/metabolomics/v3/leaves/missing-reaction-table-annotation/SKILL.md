---
name: missing-reaction-table-annotation
description: Use when MINNO's boundary-flux analysis has produced a set of candidate
  missing reactions from KEGG and each candidate must be triaged into 'literature-supported'
  versus 'NCBI-RefSeq homology' evidence categories before network refinement.
when_to_use_negative:
- The reaction set already carries curated evidence labels (e.g. a previously annotated
  gap-filling result), so re-triaging literature vs RefSeq homology would be redundant.
- The goal is to detect missing reactions de novo from metabolomics boundary fluxes
  rather than to categorize an already-identified list.
- The candidate reactions come from a pathway database other than KEGG (e.g. BiGG)
  where the article's literature/RefSeq triage workflow was not applied.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MINNO
  role: Produces the initial list of missing reactions from KEGG via boundary flux
    analysis; the annotated table is derived from MINNO's output.
  repo: github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Database queried to confirm homology for reactions lacking primary-literature
    support.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:a714aced72f59823098310f88dd8b1f7e26dcc4386e3776962739fe46051b329
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# missing-reaction-table-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates the set of reactions MINNO identifies as missing from the KEGG database by assigning each one an evidence-source label — primary-literature support or NCBI-RefSeq homology — and writing the categorized reaction table (with reaction identifiers and evidence-source labels) to a CSV file.

## When to use

Use when MINNO's boundary-flux analysis has produced a set of candidate missing reactions from KEGG and each candidate must be triaged into 'literature-supported' versus 'NCBI-RefSeq homology' evidence categories before network refinement.

## When NOT to use

- The reaction set already carries curated evidence labels (e.g. a previously annotated gap-filling result), so re-triaging literature vs RefSeq homology would be redundant.
- The goal is to detect missing reactions de novo from metabolomics boundary fluxes rather than to categorize an already-identified list.
- The candidate reactions come from a pathway database other than KEGG (e.g. BiGG) where the article's literature/RefSeq triage workflow was not applied.

## Inputs

- Table of missing reactions identified from the KEGG database (e.g. 18 reactions from MINNO boundary-flux analysis of Borrelia species)
- Supplementary evidence table (e.g. Table S1) linking each missing reaction to primary-literature confirmation
- NCBI-RefSeq database entries (genetic annotations) for homology checks

## Outputs

- CSV file of categorized reactions with reaction identifiers and evidence-source labels (literature vs NCBI-RefSeq homology)
- Summary count of reactions in each evidence category (e.g. 9 literature-supported, 9 NCBI-RefSeq-homology-supported)

## How to apply

Load the compiled table of missing reactions identified from the KEGG database (e.g. the 18 reactions MINNO surfaced from boundary flux profiles of Borrelia purine and pyrimidine metabolism). For each reaction, first check primary-literature confirmation (in this study, Table S1 confirmed six purine reactions and all four pyrimidine reactions, giving nine literature-supported entries overall). For reactions lacking literature support, verify homology against the NCBI-RefSeq database and assign the 'NCBI-RefSeq homology' label. Summarize the categorization as a split — in this case nine literature-supported and nine NCBI-RefSeq-homology-supported reactions — and write the categorized table with reaction identifiers and evidence-source labels to a CSV file. The rationale is that empirical boundary fluxes suggest a reaction exists, but the reaction must be traceable either to published biochemical evidence or to sequence homology before being added to the refined network.

## Related tools

- **MINNO** (Produces the initial list of missing reactions from KEGG via boundary flux analysis; the annotated table is derived from MINNO's output.) — github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Database queried to confirm homology for reactions lacking primary-literature support.)

## Examples

```
python annotate_missing_reactions.py --input missing_reactions.csv --evidence table_S1.csv --refseq --output annotated_reactions.csv
```

## Evaluation signals

- The categorized table contains exactly the expected number of reactions (e.g. 18) with no duplicates or unlabelled entries.
- Counts per evidence category match the article's reported split (9 literature-supported and 9 NCBI-RefSeq-homology-supported).
- Every reaction in the table carries exactly one evidence-source label drawn from {literature, NCBI-RefSeq homology}.
- Cross-check labels against Table S1: purine reactions reported as confirmed by publications should be labelled 'literature', and pyrimidine reactions all labelled 'literature'.
- Output file is a valid CSV with explicit reaction identifier and evidence-source columns.

## Limitations

- Supporting evidence resides in supplementary Table S1, which is not included in the article's main text, so the reaction-to-evidence mapping must be obtained separately.
- The triage categories are binary (literature vs RefSeq homology); reactions with mixed or partial evidence are not explicitly modelled.
- The article does not report MINNO's version number or a changelog, so reproducibility of the exact missing-reaction list depends on the same MINNO release and KEGG snapshot.

## Evidence
