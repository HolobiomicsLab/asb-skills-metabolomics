---
name: orthology-based-reaction-annotation-transfer
description: Use when building a draft GSMN for a species with a genome annotation
  but no dedicated metabolic model, using curated reference algal/plant networks as
  orthology donors (e.g., A. thaliana, G. sulphuraria, E. siliculosus → C. crispus).
when_to_use_negative:
- The target organism already has a manually curated GSMN — refinement of that model
  is preferable to orthology-based draft transfer.
- No genome annotation for the target organism is available, so gene-to-gene orthology
  mapping cannot be performed.
- The goal is de novo inference of novel reactions/pathway variants rather than transferring
  known reactions — in that case PathModel-style inference is the relevant approach.
tools:
- name: AuReMe
  role: Builds the draft metabolic network from the genome annotation and supports
    orthology-based reaction import from reference GSMNs
- name: PathModel
  role: Complementary prototype (ASP-based, clingo) for inferring new biochemical
    reactions and metabolite structures when orthology transfer is insufficient to
    explain pathway variants
  repo: https://github.com/pathmodel/pathmodel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:3545b57f01beccc239562c73b1989b2908ed89a54de8cb562bc60a78829782a5
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# orthology-based-reaction-annotation-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct a draft genome-scale metabolic network (GSMN) for an organism lacking a curated model by mapping genes from reference GSMNs (A. thaliana, G. sulphuraria, E. siliculosus) onto the target organism's genes via orthology, importing the associated reactions into a draft network built with AuReMe.

## When to use

Use when building a draft GSMN for a species with a genome annotation but no dedicated metabolic model, using curated reference algal/plant networks as orthology donors (e.g., A. thaliana, G. sulphuraria, E. siliculosus → C. crispus).

## When NOT to use

- The target organism already has a manually curated GSMN — refinement of that model is preferable to orthology-based draft transfer.
- No genome annotation for the target organism is available, so gene-to-gene orthology mapping cannot be performed.
- The goal is de novo inference of novel reactions/pathway variants rather than transferring known reactions — in that case PathModel-style inference is the relevant approach.

## Inputs

- Target organism genome annotation (C. crispus) feeding a draft AuReMe reconstruction
- Curated reference GSMNs: A. thaliana, G. sulphuraria, and E. siliculosus metabolic networks
- Orthology mapping between target genes and reference GSMN genes
- Manual gap-filling specification (e.g., the alanine dehydrogenase reaction)

## Outputs

- Draft reconstructed C. crispus GSMN with 2,024 reactions, 2,006 enzymes, 2,196 metabolites and 1,108 pathways (before curation)
- Global model metrics (reaction, metabolite, gene counts) comparable to Table 1 of the article and to the E. siliculosus GSMN (1,977 reactions)

## How to apply

First build a draft metabolic network with AuReMe directly from the target organism's genome annotation. Then perform orthology-based reconstruction: map genes from curated reference GSMNs (A. thaliana, G. sulphuraria, and E. siliculosus) onto the target genes and import the reactions associated with the orthologous genes. Apply a manual gap-filling step for known metabolic functions missed by orthology — in the C. crispus reconstruction this meant adding the alanine dehydrogenase reaction. Finally compute global model metrics (number of reactions, metabolites, genes, pathways) and validate them against the published reference values: 2,024 reactions, 2,006 enzymes, 2,196 metabolites and 1,108 pathways for the pre-curation C. crispus network versus 1,977 reactions for E. siliculosus (Aite et al., 2018).

## Related tools

- **AuReMe** (Builds the draft metabolic network from the genome annotation and supports orthology-based reaction import from reference GSMNs)
- **PathModel** (Complementary prototype (ASP-based, clingo) for inferring new biochemical reactions and metabolite structures when orthology transfer is insufficient to explain pathway variants) — https://github.com/pathmodel/pathmodel

## Evaluation signals

- Computed total reaction count of the pre-curation C. crispus network equals 2,024 as reported in Table 1 (output_matches_reference).
- Enzyme, metabolite, and pathway counts equal 2,006, 2,196, and 1,108 respectively, matching Table 1 of the article.
- The network size is comparable to the E. siliculosus GSMN (1,977 reactions, 2,281 enzymes, 2,132 metabolites, 1,101 pathways; Aite et al., 2018).
- The manually gap-filled alanine dehydrogenase reaction is present in the final reconstructed network.

## Limitations

- Orthology transfer alone does not capture every metabolic function — the alanine dehydrogenase reaction had to be added manually as a gap-filling step.
- Reported metrics apply to the network before curation; subsequent curation changes the counts.
- The article notes that scalable, automated inference of pathway variants remains a prototype-level problem (PathModel), so fully automatic reconstruction may miss non-orthologous reaction variants (metabolic pathway drift).

## Evidence

- [methods] Draft network construction from annotation: [evidence span withheld — anchor status: not_found]
- [methods] Orthology-based reaction import: [evidence span withheld — anchor status: not_found]
- [methods] Manual gap-filling step: [evidence span withheld — anchor status: not_found]
- [results] Reference metrics for validation: [evidence span withheld — anchor status: not_found]
