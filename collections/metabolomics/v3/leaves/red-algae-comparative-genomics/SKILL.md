---
name: red-algae-comparative-genomics
description: Use when a newly sequenced or emerging model organism (e.g. red alga
  C. crispus) lacks a GSMN and you need to build one from genome annotation plus orthology
  to existing algal/plant networks, then compare its size to other algal GSMNs.
when_to_use_negative:
- The organism already has a curated GSMN — use model refinement/validation workflows
  instead of de novo reconstruction
- The goal is pathway-level reaction/structure inference rather than whole-network
  reconstruction — use PathModel-style analogy reasoning instead
- No closely related species with existing GSMNs are available for orthology transfer,
  so the mapping step cannot import reactions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3174
tools:
- name: AuReMe
  role: Builds the draft metabolic network from genome annotation and supports orthology-based
    reconstruction and gap-filling
- name: PathModel
  role: ASP-based tool (with clingo/clyngor) for inferring missing biochemical reactions
    and metabolite structures, complementary to network reconstruction for pathway
    drift analysis
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: ASP grounder and solver underlying PathModel inference
  repo: https://github.com/potassco/clingo
- name: clyngor
  role: Python wrapper around clingo used by the PathModel pipeline
  repo: https://github.com/Aluriak/clyngor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:cd06442cbacf9b029835c522677d0366f640fcbd54080d4d78c3d41812441a53
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
  - clingo
  - clyngor
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# red-algae-comparative-genomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and characterize a genome-scale metabolic network (GSMN) for an emerging red algal model organism, Chondrus crispus, by combining AuReMe draft reconstruction with orthology-based reaction import from related algal/plant GSMNs and manual gap-filling, then benchmark its global features against other algal networks.

## When to use

Use when a newly sequenced or emerging model organism (e.g. red alga C. crispus) lacks a GSMN and you need to build one from genome annotation plus orthology to existing algal/plant networks, then compare its size to other algal GSMNs.

## When NOT to use

- The organism already has a curated GSMN — use model refinement/validation workflows instead of de novo reconstruction
- The goal is pathway-level reaction/structure inference rather than whole-network reconstruction — use PathModel-style analogy reasoning instead
- No closely related species with existing GSMNs are available for orthology transfer, so the mapping step cannot import reactions

## Inputs

- Genome annotation (gene/protein set) of C. crispus
- Existing GSMNs of A. thaliana, G. sulphuraria, and E. siliculosus for orthology mapping
- Manual curation list (e.g. alanine dehydrogenase reaction to gap-fill)

## Outputs

- C. crispus genome-scale metabolic network (SBML or AuReMe/PADMET format)
- Global model metrics table (reaction, enzyme, metabolite, gene, pathway counts)
- Comparative size assessment against other algal GSMNs (e.g. E. siliculosus)

## How to apply

First build a draft metabolic network with AuReMe from the C. crispus genome annotation. Then perform orthology-based reconstruction by mapping genes from the A. thaliana, G. sulphuraria, and E. siliculosus GSMNs onto C. crispus genes so their associated reactions are imported into the draft. Apply a manual curation/gap-filling step — here, adding the alanine dehydrogenase reaction — to restore disconnected or missing metabolic capabilities. Finally compute global model metrics (number of reactions, metabolites, genes, enzymes, pathways) and validate them against the published reference values (Table 1: 2,024 reactions, 2,006 enzymes, 2,196 metabolites, 1,108 pathways for C. crispus vs 1,977 reactions, 2,281 enzymes, 2,132 metabolites, 1,101 pathways for E. siliculosus).

## Related tools

- **AuReMe** (Builds the draft metabolic network from genome annotation and supports orthology-based reconstruction and gap-filling)
- **PathModel** (ASP-based tool (with clingo/clyngor) for inferring missing biochemical reactions and metabolite structures, complementary to network reconstruction for pathway drift analysis) — https://github.com/pathmodel/pathmodel
- **clingo** (ASP grounder and solver underlying PathModel inference) — https://github.com/potassco/clingo
- **clyngor** (Python wrapper around clingo used by the PathModel pipeline) — https://github.com/Aluriak/clyngor

## Examples

```
pathmodel infer -i data.lp -o output_folder -s 100
```

## Evaluation signals

- Global metrics of the reconstructed network match the published Table 1 values (output_matches_reference): 2,024 reactions, 2,006 enzymes, 2,196 metabolites, 1,108 pathways
- Comparative counts fall in the same range as the E. siliculosus GSMN (1,977 reactions, 2,281 enzymes, 2,132 metabolites, 1,101 pathways)
- Gap-filled reaction (alanine dehydrogenase) is present in the final network and connects previously disconnected metabolic capability
- Orthology mapping successfully imports reactions from all three donor GSMNs (A. thaliana, G. sulphuraria, E. siliculosus)

## Limitations

- Reconstruction relies on orthology to a few distantly related phyla; additional eukaryotic lineages and multiple species per phylum are needed for stronger support
- Manual gap-filling (e.g. alanine dehydrogenase) encodes user judgment and is not automated
- PathModel, the companion inference tool, is explicitly a working prototype with open scalability issues and a non-trivial manual burden of encoding molecular structures

## Evidence

- [other] Orthology-based reconstruction from related GSMNs: [evidence span withheld — anchor status: not_found]
- [other] Published global metrics for validation: [evidence span withheld — anchor status: not_found]
