---
name: partition-coefficient-descriptor-retrieval
description: Use when a QSAR half-life prediction step (e.g., IFS per Arnot et al.
  2014) requires log Kow and log Koa inputs for a chemical list, and descriptors must
  be fetched before filtering and prediction.
when_to_use_negative:
- The chemical contains a metal atom or has molecular weight > 1,000, which the IFS
  approach cannot handle — such chemicals should be excluded rather than queried
- The downstream analysis does not require partition-coefficient descriptors (e.g.,
  purely experimental half-life compilation or BioTransformer metabolite prediction,
  which does not consume log Kow/log Koa)
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: BioTransformer
  role: 'Downstream consumer of the retrieved descriptors: the IFS QSAR applied with
    these log Kow/log Koa inputs predicts HLBs later used alongside BioTransformer
    metabolite predictions'
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:39952ac741403cdcbac033faa565567ae9544626ef62ea2fde91e241daf4c31d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# partition-coefficient-descriptor-retrieval

## Summary

Retrieves log Kow and log Koa partition-coefficient descriptors for each chemical in a compiled exposome database from the UFZ-LSER database (v3.2.1), producing the descriptor table required by the Iterative Fragment Selection (IFS) QSAR to predict human biotransformation half-lives (HLBs).

## When to use

Use when a QSAR half-life prediction step (e.g., IFS per Arnot et al. 2014) requires log Kow and log Koa inputs for a chemical list, and descriptors must be fetched before filtering and prediction.

## When NOT to use

- The chemical contains a metal atom or has molecular weight > 1,000, which the IFS approach cannot handle — such chemicals should be excluded rather than queried
- The downstream analysis does not require partition-coefficient descriptors (e.g., purely experimental half-life compilation or BioTransformer metabolite prediction, which does not consume log Kow/log Koa)

## Inputs

- Compiled chemical list with identifiers (e.g., HExpMetDB human exposome chemicals with CASRN)
- Access to UFZ-LSER database v3.2.1 for log Kow and log Koa lookup
- Chemical structure/molecular-weight and metal-atom annotation for eligibility filtering

## Outputs

- Per-chemical table of log Kow and log Koa partition-coefficient descriptors for IFS input
- Filtered subset of chemicals eligible for IFS (no metal atom, MW <= 1,000)

## How to apply

Start from the compiled chemical list (e.g., the human exposome database of >20,000 chemicals) and query the UFZ-LSER database v3.2.1 to retrieve log Kow and log Koa for every chemical identifier. Assemble the retrieved partition coefficients into a per-chemical descriptor table keyed by chemical identifiers (CASRN/SMILES as available). Before passing descriptors downstream, exclude chemicals the IFS approach cannot handle — those containing a metal atom or with molecular weight > 1,000 — so only eligible chemicals (19,406 in this study) reach the half-life predictor. Spot-check known chemicals (DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h) after the downstream IFS run to confirm descriptor-driven predictions are consistent with reported values.

## Related tools

- **BioTransformer** (Downstream consumer of the retrieved descriptors: the IFS QSAR applied with these log Kow/log Koa inputs predicts HLBs later used alongside BioTransformer metabolite predictions) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- Descriptor table covers the chemical list with log Kow and log Koa populated for each identifier retrieved from UFZ-LSER v3.2.1
- Chemicals with a metal atom or MW > 1,000 are absent from the descriptor set passed to IFS
- Downstream IFS run predicts HLBs for the expected chemical count (19,406 in this study) with a median of 4.96 h
- ~70% of predictions (13,733/19,406) fall below 12 h half-life
- Spot-check chemicals match reported HLBs: DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h

## Limitations

- The IFS approach cannot predict HLBs for chemicals with a metal atom or molecular weight > 1,000, so such chemicals must be dropped at the descriptor/filter stage
- Descriptor coverage depends on UFZ-LSER database availability; chemicals lacking retrievable log Kow/log Koa cannot proceed to half-life prediction

## Evidence

- [abstract] abstract: "The biotransformation half-lives (HLBs) of all the chemicals were assessed using the Iterative Fragment Selection approach"
- [discussion] discussion: "we cannot predict the HLBs of chemicals with a metal atom or molecular weight of >1,000 using the IFS approach"
- [methods] methods: [evidence span withheld — anchor status: not_found]
- [results] results: [evidence span withheld — anchor status: not_found]
