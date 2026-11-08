---
name: gsmn-quality-metrics-reporting
description: Use when a newly reconstructed GSMN (e.g. from AuReMe draft reconstruction
  plus orthology mapping) needs its global size metrics tallied and compared with
  published networks to demonstrate completeness before curation.
when_to_use_negative:
- The network has not yet been reconstructed — metrics on a partial draft without
  orthology mapping or gap-filling are not comparable to curated GSMNs
- The goal is pathway-level inference of new reactions or metabolite structures (use
  PathModel-type reasoning instead of size metric reporting)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: AuReMe
  role: Draft GSMN reconstruction and computation of global model metrics (reactions,
    metabolites, genes) for the reconstructed network
- name: PathModel
  role: ASP-based tool of the same article ecosystem; the skill's context article
    applies GSMN reconstruction and metric reporting in emerging model organisms
  repo: https://github.com/pathmodel/pathmodel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:43469bd181db270fa883f3d7ce4eca1a4a6215432cacdcafd8bdec3e17ff557d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pathmodel/pathmodel
  repo_url: https://github.com/pathmodel/pathmodel
  tools:
  - AuReMe
  - PathModel
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# gsmn-quality-metrics-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and report the global quality metrics of a reconstructed genome-scale metabolic network (GSMN) — number of reactions, metabolites/enzymes, genes, and pathways — and benchmark them against published GSMNs of related organisms, as done for the C. crispus network versus E. siliculosus.

## When to use

Use when a newly reconstructed GSMN (e.g. from AuReMe draft reconstruction plus orthology mapping) needs its global size metrics tallied and compared with published networks to demonstrate completeness before curation.

## When NOT to use

- The network has not yet been reconstructed — metrics on a partial draft without orthology mapping or gap-filling are not comparable to curated GSMNs
- The goal is pathway-level inference of new reactions or metabolite structures (use PathModel-type reasoning instead of size metric reporting)

## Inputs

- Reconstructed GSMN from AuReMe (draft + orthology-mapped reactions + gap-filled reactions)
- Reference GSMN(s) of related organisms with published global metrics (e.g. E. siliculosus from Aite et al., 2018)
- Published reference table of expected network metrics (e.g. Table 1)

## Outputs

- Global model metrics: counts of reactions, metabolites, genes/enzymes, and pathways
- Comparative summary table (Table 1 style) benchmarking the new GSMN against other algal GSMNs

## How to apply

After building the draft network in AuReMe and completing reconstruction (here: orthology-based mapping of reactions from A. thaliana, G. sulphuraria, and E. siliculosus GSMNs onto C. crispus genes, followed by manual gap-filling with the alanine dehydrogenase reaction), compute the global model metrics: total reactions, metabolites, genes/enzymes, and pathway counts. Report these before curation so the pre-curation state is documented, and place them in a summary table (Table 1 style) alongside metrics from a reference GSMN of a related alga (E. siliculosus, 1,977 reactions) to contextualize size. Judge correctness by checking the reported figures match the published reference table (output_matches_reference) — for C. crispus: 2,024 reactions, 2,006 enzymes, 2,196 metabolites, 1,108 pathways before curation.

## Related tools

- **AuReMe** (Draft GSMN reconstruction and computation of global model metrics (reactions, metabolites, genes) for the reconstructed network)
- **PathModel** (ASP-based tool of the same article ecosystem; the skill's context article applies GSMN reconstruction and metric reporting in emerging model organisms) — https://github.com/pathmodel/pathmodel

## Evaluation signals

- output_matches_reference: reported counts equal the published Table 1 values (2,024 reactions, 2,006 enzymes, 2,196 metabolites, 1,108 pathways before curation for C. crispus)
- All four metric categories (reactions, enzymes, metabolites, pathways) are reported, not only reactions
- Comparison network (E. siliculosus: 1,977 reactions, 2,281 enzymes, 2,132 metabolites, 1,101 pathways) is cited with its original reference (Aite et al., 2018)
- Metrics are explicitly labeled as pre-curation values so they can be distinguished from post-curation counts

## Limitations

- Metrics reflect the pre-curation state; they will change after curation and should be labeled accordingly
- GSMN reconstruction in emerging model organisms is challenged by incomplete biochemical knowledge, so metric counts alone do not guarantee functional completeness
- Size comparisons across species depend on the reference GSMNs used for orthology mapping (A. thaliana, G. sulphuraria, E. siliculosus here)

## Evidence

- [other] C. crispus GSMN global metrics before curation: [evidence span withheld — anchor status: not_found]
- [other] Compute global model metrics in the workflow: [evidence span withheld — anchor status: not_found]
