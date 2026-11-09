---
name: molecular-weight-and-metal-atom-filtering
description: Use when a chemical list is about to be processed with fragment- or descriptor-based
  QSAR models (e.g., IFS biotransformation half-life prediction) and must first be
  screened for metals or MW > 1,000.
when_to_use_negative:
- The downstream QSAR model handles metal-containing or very large (MW > 1,000) molecules,
  so removing them would needlessly shrink coverage
- The goal is exposure or toxicity prioritization (e.g., SEEM/TEST/ToxPi ranking)
  rather than biotransformation half-life prediction — the filter is specific to the
  IFS applicability domain
- Input chemicals lack structures or molecular formulas, so metal content and molecular
  weight cannot be determined
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0209
tools:
- name: BioTransformer
  role: Downstream tool whose QSAR pipeline (IFS-based HLB prediction) the filter
    prepares inputs for; the workflow lists it as the tool applied after filtering
  repo: github.com/FangLabNTU/HExpMetDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:49ac1574d07a32c2aeeb358766cecb0757e83108ec783b8b5cb29afaffe5fe8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0209
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# Molecular Weight and Metal Atom Filtering

## Summary

Pre-filters chemical inventories by excluding compounds containing a metal atom or exceeding a molecular weight of 1,000 before running the Iterative Fragment Selection (IFS) QSAR for biotransformation half-life (HLB) prediction, since the IFS approach cannot handle such structures.

## When to use

Use when a chemical list is about to be processed with fragment- or descriptor-based QSAR models (e.g., IFS biotransformation half-life prediction) and must first be screened for metals or MW > 1,000.

## When NOT to use

- The downstream QSAR model handles metal-containing or very large (MW > 1,000) molecules, so removing them would needlessly shrink coverage
- The goal is exposure or toxicity prioritization (e.g., SEEM/TEST/ToxPi ranking) rather than biotransformation half-life prediction — the filter is specific to the IFS applicability domain
- Input chemicals lack structures or molecular formulas, so metal content and molecular weight cannot be determined

## Inputs

- Compiled chemical list with identifiers (e.g., CASRN) and structures/molecular formulas for the human exposome database
- Molecular weight per chemical (computed or database-sourced)
- Structural atom composition per chemical to detect metal atoms

## Outputs

- Filtered chemical subset eligible for IFS QSAR HLB prediction
- Exclusion log of chemicals removed due to metal atoms or MW > 1,000

## How to apply

Before applying the Iterative Fragment Selection QSAR (Arnot et al. 2014) to predict human biotransformation half-lives, load the compiled chemical list and check each entry for (a) the presence of any metal atom in its structure and (b) a molecular weight greater than 1,000. Remove any chemical meeting either criterion, because the IFS fragment-based approach cannot generate reliable predictions for these structures; in this study the filter was applied alongside retrieving log Kow and log Koa descriptors from the UFZ-LSER database v3.2.1. Record which chemicals were excluded and the surviving set size so that downstream prediction coverage (e.g., HLBs for 19,406 of >20,000 chemicals) can be reconciled against the full inventory. The rationale is to avoid wasted or invalid QSAR runs on structures outside the model's applicability domain.

## Related tools

- **BioTransformer** (Downstream tool whose QSAR pipeline (IFS-based HLB prediction) the filter prepares inputs for; the workflow lists it as the tool applied after filtering) — github.com/FangLabNTU/HExpMetDB

## Evaluation signals

- No chemical in the retained set contains a metal atom or has MW > 1,000
- Count of excluded chemicals reconciles the gap between the full exposome list (>20,000) and the 19,406 chemicals with successful HLB predictions
- Spot-check chemicals in the retained set (DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h) yield the reported HLB values
- Median predicted HLB of retained chemicals equals 4.96 h with ~70% (13,733/19,406) below 12 h

## Limitations

- Filtering reduces prediction coverage — chemicals with a metal atom or MW > 1,000 are left without predicted HLBs
- The thresholds (metal atom, MW > 1,000) reflect the IFS approach's applicability domain and may not generalize to other QSAR methods

## Evidence

- [discussion] IFS excludes metals and large molecules: "we cannot predict the HLBs of chemicals with a metal atom or molecular weight of >1,000 using the IFS approach"
- [methods] Workflow filtering step: [evidence span withheld — anchor status: not_found]
- [methods] Descriptor retrieval precedes filtering and prediction: [evidence span withheld — anchor status: not_found]
