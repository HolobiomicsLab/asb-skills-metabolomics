---
name: primary-literature-confirmation-mapping
description: Use when a metabolic network reconstruction (e.g. from KEGG/NCBI-RefSeq
  mapping in MINNO) has yielded predicted missing reactions and each candidate needs
  an evidence label — primary-literature confirmation vs homology support — before
  downstream use.
when_to_use_negative:
- Reactions already present in the reference database (e.g. annotated in KEGG) — confirmation
  mapping only applies to predicted-missing candidates.
- No literature or homology evidence is available for a candidate, in which case it
  should remain flagged as unsupported rather than forced into a category.
- The goal is purely visual network exploration in MINNO without curation intent —
  boundary-flux inspection alone does not require literature confirmation.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0092
tools:
- name: MINNO
  role: Produces the candidate missing-reaction list via boundary-flux analysis of
    temporal metabolite intensity profiles; the confirmation mapping then categorizes
    its output.
  repo: https://github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Reference sequence database used to establish homology evidence for candidate
    reactions that lack primary-literature support.
  repo: https://www.ncbi.nlm.nih.gov/refseq/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:6196bf97f5e094311f7872d0deeda8c8ea76193310ab84f91fd2a0e6f41228b7
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# primary-literature-confirmation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies whether computationally predicted (e.g. MINNO-inferred) missing metabolic reactions are supported by published experiments or by sequence homology in a reference database such as NCBI-RefSeq, producing an evidence-categorized reaction table. This step grounds boundary-flux-based network refinements in independently validated sources before they are accepted as real biology.

## When to use

Use when a metabolic network reconstruction (e.g. from KEGG/NCBI-RefSeq mapping in MINNO) has yielded predicted missing reactions and each candidate needs an evidence label — primary-literature confirmation vs homology support — before downstream use.

## When NOT to use

- Reactions already present in the reference database (e.g. annotated in KEGG) — confirmation mapping only applies to predicted-missing candidates.
- No literature or homology evidence is available for a candidate, in which case it should remain flagged as unsupported rather than forced into a category.
- The goal is purely visual network exploration in MINNO without curation intent — boundary-flux inspection alone does not require literature confirmation.

## Inputs

- Compiled table of candidate missing reactions from a network-reconstruction pass (e.g. 18 KEGG-missing reactions with reaction identifiers)
- Supporting-information table mapping reactions to publications (e.g. Table S1)
- NCBI-RefSeq database / homology search results for unconfirmed candidates

## Outputs

- CSV table of reactions annotated with evidence-source labels (literature vs NCBI-RefSeq homology)
- Tallied evidence split (e.g. 9 literature-supported / 9 NCBI-RefSeq-homology-supported out of 18)
- Prioritized curation list of reactions with independent support

## How to apply

Start from the compiled table of candidate missing reactions (e.g. the 18 reactions MINNO identified as absent from the KEGG database for Borrelia purine/pyrimidine metabolism). For each reaction, first check the primary literature — in the B. burgdorferi case, Table S1 records which of the eight missing purine/pyrimidine reactions were subsequently confirmed by publications: six purine reactions and all four pyrimidine reactions. Any candidate lacking a literature match is then assessed against the NCBI-RefSeq database for sequence homology to a protein known to catalyze the reaction. Assign each reaction one of the two evidence categories ('literature' or 'NCBI-RefSeq homology'), then tally the split — the reported outcome was nine literature-supported and nine homology-supported out of 18. Write the categorized table (reaction identifiers plus evidence-source labels) to a CSV file for reuse in downstream network curation.

## Related tools

- **MINNO** (Produces the candidate missing-reaction list via boundary-flux analysis of temporal metabolite intensity profiles; the confirmation mapping then categorizes its output.) — https://github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Reference sequence database used to establish homology evidence for candidate reactions that lack primary-literature support.) — https://www.ncbi.nlm.nih.gov/refseq/

## Examples

```
python categorize_reactions.py --input missing_reactions.csv --evidence table_s1.csv --refseq ncbi_refseq_hits.csv --output categorized_reactions.csv
```

## Evaluation signals

- The categorized table contains exactly the expected number of candidate reactions (18 in the published case) with no unassigned entries.
- The evidence split matches the reported distribution (9 literature-supported, 9 NCBI-RefSeq-homology-supported).
- Every literature label traces to a specific publication recorded in Table S1; every homology label traces to a RefSeq hit.
- All reaction identifiers in the output CSV correspond 1:1 to the input candidate list (no dropped or duplicated reactions).

## Limitations

- Supporting evidence resides in supplementary Table S1, which is not part of the main text — the exact per-reaction literature references must be retrieved separately.
- MINNO's boundary-flux inference can suggest reactions whose existence is only probable; homology ('good homology as in the NCBI-RefSeq database') supports but does not prove catalytic function in the organism.
- The evidence split depends on the completeness of the literature search; newly published confirmations can shift reactions between categories.

## Evidence
