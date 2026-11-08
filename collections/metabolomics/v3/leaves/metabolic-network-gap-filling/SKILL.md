---
name: metabolic-network-gap-filling
description: Use when building a GSMN from a newly annotated genome of an emerging
  model organism whose biochemical knowledge is incomplete and requires orthology
  mapping plus manual gap-filling.
when_to_use_negative:
- The organism already has a curated GSMN — refinement or curation workflows are more
  appropriate than draft reconstruction plus gap-filling
- The goal is inference of novel reactions or metabolite structures within a specific
  pathway (e.g. sterol or MAA biosynthesis) — use PathModel analogy reasoning instead
  of whole-network gap-filling
- No genome annotation and no reference GSMNs with orthologous genes are available
  — orthology-based reconstruction cannot proceed
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: AuReMe
  role: Workflow manager used to build the draft C. crispus metabolic network from
    genome annotation and run orthology-based reconstruction from reference GSMNs
- name: PathModel
  role: ASP-based tool (clingo/clyngor) associated with the article for inferring
    missing biochemical reactions and metabolite structures, complementing gap-filling
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver executing the ASP programs underlying PathModel's reaction
    inference
  repo: https://github.com/potassco/clingo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:2bde9c89bb296440418a751440d1aadea6eb0fa1e9ed18ee540c785520879291
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolic-network-gap-filling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Draft reconstruction of a genome-scale metabolic network (GSMN) in AuReMe, followed by orthology-based reaction import from reference algal/plant GSMNs and a manual gap-filling step, to produce a curated network with global metrics comparable to published algal models.

## When to use

Use when building a GSMN from a newly annotated genome of an emerging model organism whose biochemical knowledge is incomplete and requires orthology mapping plus manual gap-filling.

## When NOT to use

- The organism already has a curated GSMN — refinement or curation workflows are more appropriate than draft reconstruction plus gap-filling
- The goal is inference of novel reactions or metabolite structures within a specific pathway (e.g. sterol or MAA biosynthesis) — use PathModel analogy reasoning instead of whole-network gap-filling
- No genome annotation and no reference GSMNs with orthologous genes are available — orthology-based reconstruction cannot proceed

## Inputs

- Genome annotation of the target organism (C. crispus)
- Reference GSMNs for orthology mapping (A. thaliana, G. sulphuraria, E. siliculosus)
- Manually curated list of gap-filling reactions (e.g. alanine dehydrogenase)

## Outputs

- Reconstructed C. crispus genome-scale metabolic network (AuReMe/Pathway format)
- Global metrics table: counts of reactions (2,024), enzymes (2,006), metabolites (2,196), pathways (1,108)
- Comparison of network size against other algal GSMNs (e.g. E. siliculosus: 1,977 reactions, 2,281 enzymes, 2,132 metabolites, 1,101 pathways)

## How to apply

Build a draft metabolic network with AuReMe from the organism's genome annotation (here, the red alga Chondrus crispus). Perform orthology-based reconstruction by mapping genes from reference GSMNs — A. thaliana, G. sulphuraria, and E. siliculosus — onto the target genes to import their associated reactions. Apply a manual gap-filling step for reactions orthology cannot recover; in the C. crispus case this consisted of adding the alanine dehydrogenase reaction. Finally compute the global model metrics (number of reactions, metabolites, genes, enzymes, pathways) and check them against the published reference table (e.g. 2,024 reactions, 2,006 enzymes, 2,196 metabolites and 1,108 pathways before curation for C. crispus, versus 1,977 reactions for E. siliculosus).

## Related tools

- **AuReMe** (Workflow manager used to build the draft C. crispus metabolic network from genome annotation and run orthology-based reconstruction from reference GSMNs)
- **PathModel** (ASP-based tool (clingo/clyngor) associated with the article for inferring missing biochemical reactions and metabolite structures, complementing gap-filling) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver executing the ASP programs underlying PathModel's reaction inference) — https://github.com/potassco/clingo

## Examples

```
pathmodel infer -i data.lp -o output_folder -s 100
```

## Evaluation signals

- Global model metrics match the published Table 1 values: 2,024 reactions, 2,006 enzymes, 2,196 metabolites, 1,108 pathways before curation
- Comparison against E. siliculosus (Aite et al., 2018) shows C. crispus is larger: 2,024 vs 1,977 reactions
- The manually gap-filled reaction (alanine dehydrogenase) is present in the final network
- All reactions imported by orthology mapping from A. thaliana, G. sulphuraria and E. siliculosus GSMNs are present in the reconstruction
- Every reaction in the final network is traceable to a source: draft annotation, orthology import, or manual gap-filling

## Limitations

- Manual gap-filling depends on prior biochemical knowledge; the article notes the main limitation is 'the user time necessary to properly encode the starting knowledge base'
- Orthology-based reconstruction misses reactions performed by non-orthologous genes (non-orthologous gene displacement), a form of metabolic pathway drift; enzymes for some reactions (e.g. porphyra-334 dehydration and decarboxylation in the MAA pathway) remain unknown, making them not genomically tractable
- Comparisons rely on C. crispus and only a few distantly related phyla; additional eukaryotic lineages and multiple species per phylum are needed for stronger support

## Evidence

- [methods] Draft reconstruction step and orthology mapping of reference GSMNs: [evidence span withheld — anchor status: not_found]
- [methods] Manual gap-filling step: [evidence span withheld — anchor status: not_found]
- [results] Global metrics and comparison values: [evidence span withheld — anchor status: not_found]
