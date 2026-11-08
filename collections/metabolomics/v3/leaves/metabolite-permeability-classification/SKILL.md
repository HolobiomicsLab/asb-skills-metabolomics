---
name: metabolite-permeability-classification
description: Use when computing exportable metabolites from a draft reconstruction
  prior to community-dependent gap filling, or when building the leakage-enlarged
  minimal medium for COMMIT.
when_to_use_negative:
- A curated SBML model already specifies exchange/excretion reactions explicitly —
  do not re-derive permeability classes
- The metabolite set is not normalized to MNXref identifiers, so InChI-key lookups
  against PubChem will fail or be unreliable
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3314
tools:
- name: COMMIT
  role: Applies the permeability rules to flag exportable metabolites that drive community-dependent
    gap filling
  repo: github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: Computes exportable metabolites via sink-reaction maximization at the suboptimal
    biomass factor f = 0.9 before the permeability filter is applied
  repo: https://github.com/opencobra/cobratoolbox
- name: PubChem
  role: Source of the molecular property values (HBD, HBA, MW, TPSA, RB, XLogP3) used
    by the threshold rules
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e0d6d1bdaf3f9524fb5aa5fe30e4b801a4c86ed19aad620a6715e700c65b1726
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - HMMER
  - IBM CPLEX
  - PubChem
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-permeability-classification

## Summary

Classify MNXref metabolites as likely- or not-permeable using Lipinski-style molecular property thresholds (HBD, HBA, MW, TPSA, rotatable bonds, XLogP3) so that only plausible secreted compounds are made available for community gap filling in COMMIT.

## When to use

Use when computing exportable metabolites from a draft reconstruction prior to community-dependent gap filling, or when building the leakage-enlarged minimal medium for COMMIT.

## When NOT to use

- A curated SBML model already specifies exchange/excretion reactions explicitly — do not re-derive permeability classes
- The metabolite set is not normalized to MNXref identifiers, so InChI-key lookups against PubChem will fail or be unreliable

## Inputs

- Draft genome-scale metabolic reconstructions with MNXref metabolite identifiers (Matlab .mat from KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools)
- MNXref metabolite-to-InChI-key reference files
- PubChem compound records with molecular properties (HBD, HBA, MW, TPSA, RB, XLogP3)

## Outputs

- Per-metabolite permeability label (likely / not likely permeable)
- Set of exportable metabolites per member model at f = 0.9 suboptimal biomass
- Enlarged minimal medium of secreted metabolites used to shape subsequent gap-filling LPs

## How to apply

For each metabolite in the MNXref-based gap-filling database, retrieve molecular properties from PubChem (hydrogen-bond donors, hydrogen-bond acceptors, molecular weight, topological polar surface area, rotatable bonds, and predicted octanol/water partition coefficient XLogP3). Apply the COMMIT permeability rules — HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6 — to label a metabolite as likely permeable; any metabolite missing one or more of these properties is classified as not likely to be permeable. Because InChI-key translation to PubChem CIDs is incomplete (only ~26% translated, ~11% with XLogP3), prepare for a large class of unclassified metabolites and document them. The resulting exportable-metabolite set, determined at the 10% biomass-reduction threshold (suboptimal factor f = 0.9 for sink-reaction maximization), feeds the shared medium enlargement step after each member model is gap filled.

## Related tools

- **COMMIT** (Applies the permeability rules to flag exportable metabolites that drive community-dependent gap filling) — github.com/pwendering/COMMIT
- **COBRA toolbox** (Computes exportable metabolites via sink-reaction maximization at the suboptimal biomass factor f = 0.9 before the permeability filter is applied) — https://github.com/opencobra/cobratoolbox
- **PubChem** (Source of the molecular property values (HBD, HBA, MW, TPSA, RB, XLogP3) used by the threshold rules)

## Examples

```
git clone https://github.com/pwendering/COMMIT; then in MATLAB with COBRA toolbox v3.0, run run_iterative_gap_filling.m with the permeability rules applied to the MNXref metabolite set.
```

## Evaluation signals

- Every metabolite lacking any of the six PubChem properties is labeled not permeable — verify no metabolite with missing data is marked permeable
- All XLogP3 values fall in the accepted window −0.4 to 5.6; values outside must be rejected
- Exportable-metabolite set size should track the reported limitation that only ~26% of InChI keys translated to CIDs and only ~11% had XLogP3
- After medium enlargement, subsequent member gap-filling solutions should be smaller than individual gap-filling sets (COMMIT's reported outcome)
- The chosen exportable set must satisfy the four COMMIT ordering-selection criteria while preserving biomass production and genomic support

## Limitations

- Only ~26% of InChI keys could be translated to PubChem CIDs and only ~11% of entries had XLogP3 values, so most metabolites are auto-classified as not permeable by default
- The permeability rules are Lipinski-style heuristics tuned for the COMMIT gap-filling loop; they are not a physiologically validated secretion predictor
- Weights and thresholds were tuned ad hoc 'until satisfying results were obtained for selected reconstructions', so no systematic selection procedure is documented

## Evidence

- [discussion] Lipinski-style permeability thresholds: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] Missing properties default to not permeable: "If no properties were available, the respective metabolite was classified as not likely to be permeable."
- [discussion] Sparse PubChem coverage: "so molecular properties for 110266 compounds could be obtained. Since not all entries had an associated XLogP3 value (only 11894: 11%)"
- [abstract] COMMIT leakage-driven gap filling: "We then devise an approach for gap filling, termed COMMIT, that considers metabolites for secretion based on their permeability and the composition of the community."
- [readme] COMMIT repository scope: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities"
