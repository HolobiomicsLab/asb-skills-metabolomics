---
name: gap-filling-reaction-set-comparison
description: Use when consensus microbial reconstructions (e.g. Schlaeppi 24-isolate
  or Bulgarelli 20-isolate communities) have been gap-filled both with and without
  community composition, and you must quantify whether community-aware gap filling
  reduced added-reaction set size.
when_to_use_negative:
- When the goal is predicting metabolic cross-feeding interactions rather than comparing
  gap-filling solutions — downstream interaction analysis uses SMETANA on the already
  gap-filled models
- When the CarveMe reconstruction medium has not been removed and the models are not
  on the minimal D-glucose medium — comparison would be confounded
- When comparing reconstructed models directly against reference models via reaction
  cross-references — the article notes this comparison was hampered by a lack of reaction
  cross-references to other name spaces
edam_topics: []
tools:
- name: COMMIT
  role: Performs the community-dependent conditional gap filling (100 iterations)
    whose added-reaction sets are compared; repo contains code/matlab/gap_filling/run_iterative_gap_filling.m
    and gap_fill_individual_models.m
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB framework (v3.0) used to load models with readCbModel and run the gap-filling
    LPs
  repo: https://github.com/opencobra/cobratoolbox
- name: memote
  role: Test suite used to independently assess the quality of the gap-filled functional
    models used in the comparison
- name: SMETANA
  role: Downstream analysis of the gap-filled models for directed metabolic interactions
    after the reaction-set comparison
  repo: https://github.com/cdanielmachado/smetana
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:490450f9d4ca34b00f312f92708a18b0752b6b0d2f1ce59bcbb08526dd729d06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - IBM CPLEX
  - SMETANA
  - memote
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# gap-filling-reaction-set-comparison

## Summary

Compares the sets of reactions added during gap filling of community-consensus metabolic reconstructions by a community-aware method (COMMIT) versus individual per-model gap filling, testing that community-aware filling yields significantly smaller added-reaction sets without changing genomic support.

## When to use

Use when consensus microbial reconstructions (e.g. Schlaeppi 24-isolate or Bulgarelli 20-isolate communities) have been gap-filled both with and without community composition, and you must quantify whether community-aware gap filling reduced added-reaction set size.

## When NOT to use

- When the goal is predicting metabolic cross-feeding interactions rather than comparing gap-filling solutions — downstream interaction analysis uses SMETANA on the already gap-filled models
- When the CarveMe reconstruction medium has not been removed and the models are not on the minimal D-glucose medium — comparison would be confounded
- When comparing reconstructed models directly against reference models via reaction cross-references — the article notes this comparison was hampered by a lack of reaction cross-references to other name spaces

## Inputs

- Consensus genome-scale metabolic reconstructions in SBML/COBRA format for the Schlaeppi (24 isolates) and Bulgarelli (20 isolates) soil communities from data/models/consensus
- Gap-filling database Universal-model-MNXref-balanced (MNXref identifiers, MNXM01 replaced by MNXM1 for H+)
- D-glucose minimal medium (intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)
- Molecular property data from PubChem/MNXref for permeability classification (HBD, HBA, MW, TPSA, RB, XLogP3)

## Outputs

- Added-reaction sets per reconstruction type and gap-filling method (COMMIT community-aware vs individual)
- Statistical comparison of added-reaction set sizes with genomic support unchanged
- memote quality metrics: overall score (~35%), consistency (~44%), GPR fraction (~90%)
- K-means (K=3) cluster assignments of added-reaction sets with pheatmap/igraph/ape visualizations

## How to apply

Configure paths in code/matlab/options.m and load the consensus reconstructions (data/models/consensus) with the COBRA toolbox readCbModel; define the D-glucose minimal medium as the intersection of KBase auxotrophic media excluding amino acids and carbon sources except glucose, removing the medium used during CarveMe reconstruction first. Run COMMIT conditional gap filling (run_iterative_gap_filling.m, 100 iterations) against the Universal-model-MNXref-balanced database using permeability rules (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6), reaction weights (transport 100, metabolic 50, directionality change 25, permeable transport 50, sequence support 25, uptake of secreted metabolites 1, exchange 10^5), and biomass flux limits of 2.81 h^-1 (upper) and 10^-3 (lower). Then run gap_fill_individual_models.m on the same reconstructions without community composition, and statistically test that the COMMIT added-reaction sets are significantly smaller with unchanged genomic support. Assess the resulting functional models with memote (expected score ~35%, consistency ~44%, GPR fraction ~90%), and cluster the added-reaction sets by reconstruction type and gap-filling method using K-means (K=3), visualizing with the R packages pheatmap, igraph, and ape.

## Related tools

- **COMMIT** (Performs the community-dependent conditional gap filling (100 iterations) whose added-reaction sets are compared; repo contains code/matlab/gap_filling/run_iterative_gap_filling.m and gap_fill_individual_models.m) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB framework (v3.0) used to load models with readCbModel and run the gap-filling LPs) — https://github.com/opencobra/cobratoolbox
- **memote** (Test suite used to independently assess the quality of the gap-filled functional models used in the comparison)
- **SMETANA** (Downstream analysis of the gap-filled models for directed metabolic interactions after the reaction-set comparison) — https://github.com/cdanielmachado/smetana

## Examples

```
git clone https://github.com/pwendering/COMMIT
```

## Evaluation signals

- COMMIT added-reaction sets are significantly smaller than those from individual gap filling, while genomic support is unchanged
- memote metrics of gap-filled functional models match reported values: score ~35%, consistency ~44%, GPR fraction ~90%
- Added-reaction sets cluster by reconstruction type and gap-filling method when clustered with K-means (K=3)
- Gap-filling LP respects biomass flux bounds of 2.81 h^-1 upper and 10^-3 lower, and reaction weights (transport 100, metabolic 50, directionality change 25, permeable transport 50, sequence support 25, uptake of secreted metabolites 1, exchange 10^5)
- Metabolite identifiers are MNXref-normalized with MNXM01 replaced by MNXM1 (both encode H+)

## Limitations

- Gap-filling penalty weights were tuned ad hoc 'until satisfying results were obtained for selected reconstructions'; no systematic selection procedure is documented
- The full COMMIT procedure specification is deferred to supplementary S1 Text, and criteria for choosing the 'best ordering' of reconstructions are referenced but not specified in the text
- Only 26% of InChI keys could be translated to PubChem CIDs and only 11% had XLogP3 values, so metabolites lacking properties are classified as not likely permeable, affecting permeability-based leakage decisions

## Evidence

- [abstract] significantly reduce the gap-filling solution compared to individual gap filling: "we could significantly reduce the gap-filling solution in comparison to filling gaps in individual reconstructions without affecting the genomic support"
- [intro] COMMIT reduces added reactions: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [readme] repository scope for comparison analysis: "for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions"
