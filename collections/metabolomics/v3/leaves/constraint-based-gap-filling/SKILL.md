---
name: constraint-based-gap-filling
description: Use when draft genome-scale metabolic reconstructions of a multi-species
  community fail to produce biomass on a minimal medium and gap filling must respect
  community composition and metabolite permeability/leakage.
when_to_use_negative:
- Gap filling a single isolate model where community composition and metabolite leakage
  are irrelevant — individual gap filling without community context is the appropriate
  comparator there
- 'Validation requiring specificity/true-negative metrics: since true negatives cannot
  be determined, only sensitivity (recall) and precision are computable for the removal-recovery
  test'
- Metabolites lacking PubChem properties must be assumed non-permeable, so if nearly
  all metabolites lack XLogP3 values (only 11% of PubChem entries had them), permeability
  classification will be unreliable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: COMMIT
  role: Implements the community-dependent iterative gap-filling loop (run_iterative_gap_filling.m)
    with random member orderings and medium enlargement
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: MATLAB framework in which the FastGapFilling LP and model handling run
  repo: https://github.com/opencobra/cobratoolbox
- name: IBM CPLEX v12.9
  role: Tested LP solver for the gap-filling LPs
  repo: https://www.ibm.com/analytics/cplex-optimizer
- name: Gurobi v9.1.1
  role: Alternative tested LP solver; glpk is not recommended
- name: HMMER
  role: Provides sequence-similarity evidence for the sequence-support penalty weight
    (E-value cutoff 10^-6)
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:4163f5f5b848ad8560afa07056e921d2e21f447287ded81c66cf03cdf5c530f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - Gurobi
  - Gurobi v9.1.1
  - HMMER
  - IBM CPLEX
  - IBM CPLEX v12.9
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# constraint-based-gap-filling

## Summary

Community-aware constraint-based gap filling (COMMIT) that repairs draft genome-scale metabolic reconstructions by adding a minimal set of reactions from an MNXref-based universal database, while exploiting metabolites secreted (leaked) by already-functional community members to shrink the added-reaction set relative to individual gap filling.

## When to use

Use when draft genome-scale metabolic reconstructions of a multi-species community fail to produce biomass on a minimal medium and gap filling must respect community composition and metabolite permeability/leakage.

## When NOT to use

- Gap filling a single isolate model where community composition and metabolite leakage are irrelevant — individual gap filling without community context is the appropriate comparator there
- Validation requiring specificity/true-negative metrics: since true negatives cannot be determined, only sensitivity (recall) and precision are computable for the removal-recovery test
- Metabolites lacking PubChem properties must be assumed non-permeable, so if nearly all metabolites lack XLogP3 values (only 11% of PubChem entries had them), permeability classification will be unreliable

## Inputs

- Draft genome-scale metabolic reconstructions for community members (MATLAB .mat cell arrays, e.g. data/models/_habitat_/_method_)
- MNXref-based universal gap-filling database (dbModel_MNXref_balanced / Universal-model-MNXref-balanced.mat)
- Community composition (e.g. Bulgarelli and Schlaeppi soil communities from At-SPHERE)
- Minimal medium definition (intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)
- Molecular property table for MNXref metabolites derived from PubChem (HBD, HBA, MW, TPSA, RB, XLogP3)
- Optional sequence-similarity evidence (HMMER alignments, E-value cutoff 10^-6)

## Outputs

- Gap-filled, biomass-producing community member reconstructions
- Per-member added-reaction sets under each ordering (minimal gap-filling solution)
- Selected optimal member ordering out of 100 random orderings
- Enlarged shared medium containing exportable metabolites of functional members
- Precision/recall table for removal-recovery validation (reaction_recovery_precision_recall.csv)

## How to apply

Load the draft reconstructions (MATLAB .mat cell arrays), the MNXref-balanced universal gap-filling database, the community composition, and a minimal medium (intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose) into MATLAB with COBRA toolbox v3.0. For each member, determine exportable metabolites via permeability rules (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6) at the 10% biomass-reduction threshold (suboptimal biomass factor f = 0.9), then iterate gap filling over 100 random member orderings, solving the FastGapFilling LP per member with penalty weights (transport 100, metabolic non-transport 50, directionality change 25, permeable transport 50, sequence support 25 with HMMER E-value cutoff 10^-6, uptake of secreted metabolites 1, exchange 10^5) and biomass flux bounds of 10^-3 lower / 2.81 h^-1 upper. After each member is gap-filled, enlarge the shared medium with its exportable metabolites so subsequent members can take up leaked metabolites, and select the ordering that satisfies the four specified criteria. Run with IBM CPLEX v12.9 or Gurobi v9.1.1 as the LP solver (glpk is not recommended). Exclude export/boundary reactions and reactions with stoichiometric coefficients above 20 from the gap-filling database, and replace MNXM01 with MNXM1 (both encode H+).

## Related tools

- **COMMIT** (Implements the community-dependent iterative gap-filling loop (run_iterative_gap_filling.m) with random member orderings and medium enlargement) — https://github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (MATLAB framework in which the FastGapFilling LP and model handling run) — https://github.com/opencobra/cobratoolbox
- **IBM CPLEX v12.9** (Tested LP solver for the gap-filling LPs) — https://www.ibm.com/analytics/cplex-optimizer
- **Gurobi v9.1.1** (Alternative tested LP solver; glpk is not recommended)
- **HMMER** (Provides sequence-similarity evidence for the sequence-support penalty weight (E-value cutoff 10^-6))

## Examples

```
git clone https://github.com/pwendering/COMMIT; then in MATLAB with COBRA toolbox v3.0: run code/matlab/gap_filling/run_iterative_gap_filling.m with dbModel_MNXref_balanced and the community composition
```

## Evaluation signals

- Community-aware gap filling must yield a smaller total added-reaction set than gap-filling individual reconstructions without community consideration, while maintaining biomass production and genomic support
- In the random-removal validation (1%, 2%, 5%, 10% removal fractions, 50 repetitions each), community-aware gap filling should show slightly improved precision and recall over individual gap filling
- Only sensitivity and precision are computable for the validation; if true negatives are reported, the metric implementation is wrong
- Gap filling must start from a minimal medium (the CarveMe reconstruction medium is removed beforehand), and the medium must be enlarged with exportable metabolites of already gap-filled members
- The chosen LP solver must be CPLEX or Gurobi; results obtained with glpk are unreliable per the workflow

## Limitations

- Penalty weights were tuned ad hoc 'until satisfying results were obtained for selected reconstructions' — no systematic selection procedure is documented, and the full COMMIT procedure is deferred to supplementary S1 Text
- Molecular properties were unavailable for many metabolites (only ~11% of PubChem entries had XLogP3), and metabolites without properties were classified as not likely permeable
- Criteria for choosing the 'best ordering' are referenced but not fully specified in the main text
- Comparison to reference models is hampered by lack of reaction cross-references to other name spaces; only sensitivity and precision (not specificity) are assessable
- 16S rRNA abundance data used for community roles were only available via personal provision, not a public accession

## Evidence

- [abstract] COMMIT considers permeability and community composition for secretion during gap filling: "We then devise an approach for gap filling, termed COMMIT, that considers metabolites for secretion based on their permeability and the composition of the community."
- [discussion] Permeability thresholds used to classify exportable metabolites: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] Biomass flux upper bound in the gap-filling LP: "The upper limit for the biomass flux for each isolate model was constrained to 2.81 h−1"
- [intro] COMMIT reduces the gap-filling solution relative to individual gap filling: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [readme] Repository scope: community-dependent gap filling for the two soil communities: "This repository contains all files used for community-dependent gap-filling and subsequent analysis"
