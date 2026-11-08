---
name: ncbi-refseq-homology-verification
description: Use when MINNO or a similar flux-based workflow flags reactions as absent
  from the KEGG database and each candidate must be classified as literature-supported
  or RefSeq-homology-supported before being added to the network.
when_to_use_negative:
- The reaction is already present in the KEGG base network that MINNO loads — no verification
  is needed because it is annotated, not inferred.
- The goal is de novo pathway reconstruction from raw sequencing reads rather than
  triage of a small set of flux-inferred candidate reactions.
- Neither literature nor RefSeq homology is available for the organism in question;
  the reaction remains unverified and should be flagged as hypothetical rather than
  forced into either category.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3391
tools:
- name: MINNO
  role: Produces the candidate list of KEGG-missing reactions via boundary-flux analysis
    that are then triaged for literature or homology support
  repo: github.com/LewisResearchGroup/MINNO
- name: NCBI-RefSeq
  role: Homology database queried to verify that remaining (non-literature) missing
    reactions have annotated gene/protein support in related organisms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04501
    title: 'MINNO: An Open Source Software for Refining Metabolic Networks and Investigating
      Complex Network Activity Using Empirical Metabolomics Data'
  source_content_hash: sha256:2b9d73e0e6ad7ada2933797c879fceb9db6ed8d9a76a977c9cafe14119327c83
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ncbi-refseq-homology-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verify that metabolic reactions inferred as missing from a KEGG-derived network (e.g. via MINNO boundary-flux analysis) are supported by primary literature or by homology to annotated genes in the NCBI-RefSeq database, producing a categorized table of evidence sources for each reaction.

## When to use

Use when MINNO or a similar flux-based workflow flags reactions as absent from the KEGG database and each candidate must be classified as literature-supported or RefSeq-homology-supported before being added to the network.

## When NOT to use

- The reaction is already present in the KEGG base network that MINNO loads — no verification is needed because it is annotated, not inferred.
- The goal is de novo pathway reconstruction from raw sequencing reads rather than triage of a small set of flux-inferred candidate reactions.
- Neither literature nor RefSeq homology is available for the organism in question; the reaction remains unverified and should be flagged as hypothetical rather than forced into either category.

## Inputs

- Compiled table of reactions identified as missing from the KEGG database by MINNO (reaction identifiers plus the metabolite boundary-flux evidence that motivated each inference)
- Access to primary literature/publications describing the enzymatic reactions
- NCBI-RefSeq database access (https://www.ncbi.nlm.nih.gov/refseq/) for homology lookup

## Outputs

- CSV table mapping each missing reaction identifier to an evidence-source label ('literature' or 'NCBI-RefSeq homology')
- Updated metabolic network incorporating only evidence-verified reactions

## How to apply

Start from the compiled table of missing reactions produced by MINNO's boundary-flux inference (e.g. the 18 KEGG-missing reactions identified for Borrelia species). For each reaction, first check primary publications — in the B. burgdorferi purine/pyrimidine case, six purine reactions and all four pyrimidine reactions were confirmed via Table S1 literature evidence — and tag those as 'literature'. For reactions lacking publication support, query the NCBI-Reference Sequence (RefSeq) database (https://www.ncbi.nlm.nih.gov/refseq/) for homologous genes/proteins and tag those as 'NCBI-RefSeq homology'. The rationale is that MINNO infers reactions purely from metabolite boundary fluxes, so each inferred reaction needs independent genotypic or bibliographic grounding before it can be trusted as a real enzymatic step. Finally, write the reaction identifiers with their evidence-source labels to a CSV so the refined network's provenance is auditable.

## Related tools

- **MINNO** (Produces the candidate list of KEGG-missing reactions via boundary-flux analysis that are then triaged for literature or homology support) — github.com/LewisResearchGroup/MINNO
- **NCBI-RefSeq** (Homology database queried to verify that remaining (non-literature) missing reactions have annotated gene/protein support in related organisms)

## Examples

```
open https://lewisresearchgroup.github.io/MINNO/  # load KEGG layout + RefSeq annotations in MINNO, infer missing reactions from boundary fluxes, then triage each via Table S1 literature check or NCBI-RefSeq homology search (https://www.ncbi.nlm.nih.gov/refseq/)
```

## Evaluation signals

- The categorized table must contain exactly the number of reactions identified as KEGG-missing (e.g. 18 for the Borrelia use case).
- The split between 'literature'-tagged and 'NCBI-RefSeq homology'-tagged reactions must match the reported breakdown (9 literature / 9 RefSeq for the Borrelia case, per Table S1).
- Every reaction in the input missing-reaction table appears exactly once in the output CSV with a non-empty evidence label.
- No reaction tagged 'literature' lacks a citable publication, and no reaction tagged 'NCBI-RefSeq homology' lacks a RefSeq match.
- The refined network should not contain any reaction that failed both the literature and homology checks.

## Limitations

- Supporting evidence details reside in supplementary Table S1, which is not reproduced in the main text, so per-reaction assignments may require consulting the SI directly.
- Boundary-flux inference can suggest reactions whose homology support is 'good' but not definitive; the article reports homology rather than experimentally verified enzyme activity for RefSeq-supported reactions.
- The 66 base KEGG pathways bundled with MINNO cover primary metabolic pathways; reactions outside those pathways will not be surfaced by the workflow at all.

## Evidence

- [abstract] MINNO identified 18 missing reactions from the KEGG database, of which nine were supported by the primary literature.: [evidence span withheld — anchor status: not_found]
