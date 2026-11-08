---
name: community-composition-aware-ordering-search
description: Use when gap-filling multiple draft community member reconstructions
  (e.g. KBase/CarveMe/RAVEN/AuReMe MNXref models) where metabolite leakage and member
  processing order affect the total added-reaction set.
when_to_use_negative:
- Gap-filling a single isolate model in isolation, with no community composition or
  metabolite leakage to consider
- Input models use identifier spaces other than MNXref and cannot be translated —
  comparison/gap filling is hampered without cross-references
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: COMMIT
  role: Implements the iterative community-dependent gap-filling loop and random-ordering
    search
  repo: github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: Solves the FastGapFilling LPs and handles model manipulation in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: HMMER
  role: Provides sequence similarity weights for gap-filling reactions with E-value
    cutoff 10^-6
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e60a870ec5d7fe5fa35d728f72b26fa5e2b69248f8894cac22cf0c52f19e654e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - HMMER
  - IBM CPLEX
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# community-composition-aware-ordering-search

## Summary

Enumerates member-model orderings in COMMIT so that metabolites secreted by already gap-filled community members enlarge the shared medium for later members, minimizing added reactions relative to individual gap filling.

## When to use

Use when gap-filling multiple draft community member reconstructions (e.g. KBase/CarveMe/RAVEN/AuReMe MNXref models) where metabolite leakage and member processing order affect the total added-reaction set.

## When NOT to use

- Gap-filling a single isolate model in isolation, with no community composition or metabolite leakage to consider
- Input models use identifier spaces other than MNXref and cannot be translated — comparison/gap filling is hampered without cross-references

## Inputs

- Draft member reconstructions (Matlab .mat cell arrays under data/models/<habitat>/<method>)
- Universal MNXref gap-filling database (dbModel_MNXref_balanced)
- Community composition
- Minimal medium (intersection of auxotrophic media, excluding amino acids and carbon sources except glucose)

## Outputs

- 100 gap-filling solutions, one per random member ordering
- Selected optimal ordering with its added-reaction set
- Enlarged shared medium per community member

## How to apply

Initialize the gap-filling loop from a minimal medium (e.g. the intersection of auxotrophic media excluding amino acids and carbon sources except glucose). For each random ordering of member models, run FastGapFilling LPs sequentially with penalties (transport 100, metabolic non-transport 50, directionality change 25, permeable transport 50, sequence support 25, uptake of secreted metabolites 1, exchange 10^5) and biomass bounds 10^-3 lower / 2.81 h^-1 upper. After each member is completed, enlarge the medium with its exportable metabolites — determined by Lipinski-style permeability rules (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, -0.4≤XLogP3≤5.6) at suboptimal biomass factor f=0.9 — so later members can take up leaked metabolites instead of receiving gap-filled reactions. Iterate over 100 random orderings, then select the ordering satisfying the specified criteria, validating that the total added-reaction set is smaller than gap-filling individual reconstructions while maintaining biomass production and genomic support. The rationale is that an enlarged medium is expected to reduce the added reaction sets of subsequent members.

## Related tools

- **COMMIT** (Implements the iterative community-dependent gap-filling loop and random-ordering search) — github.com/pwendering/COMMIT
- **COBRA toolbox** (Solves the FastGapFilling LPs and handles model manipulation in MATLAB) — https://github.com/opencobra/cobratoolbox
- **HMMER** (Provides sequence similarity weights for gap-filling reactions with E-value cutoff 10^-6)

## Examples

```
matlab -batch "run_iterative_gap_filling('data/models/bulgarelli','dbModel_MNXref_balanced','media/minimal_medium.csv',100)"
```

## Evaluation signals

- Total added reactions must be smaller than gap-filling individual reconstructions without community consideration
- Biomass production must be maintained after gap filling
- Genomic support (HMMER E-value ≤ 10^-6) must not degrade
- 100 random orderings must be evaluated and compared against the specified criteria
- Enlarged medium should reduce added-reaction sets for subsequent members

## Limitations

- Penalty weights were tuned ad hoc until satisfying results, with no systematic selection procedure documented
- The full COMMIT procedure is specified only in supplementary S1 Text
- Criteria for choosing the 'best ordering' are referenced but not specified in the main text
- Only 26% of InChI keys mapped to PubChem CIDs and only 11% had XLogP3 values, so many metabolites default to not permeable

## Evidence

- [discussion] Gap filling that respects community composition and metabolite leakage: [evidence span withheld — anchor status: not_found]
- [discussion] Medium enlargement rationale: "the enlarged medium is expected to reduce the added reaction sets of subsequent members"
- [discussion] Optimal ordering existence: "the existence of an optimal ordering is likely"
- [discussion] Permeability thresholds: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] Ordering comparison criteria: "The goal is to identify the best ordering with respect to specified criteria. More specifically, the reconstructions are gap-filled following a given ordering."
- [readme] Repository scope: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
