---
name: metabolite-leakage-medium-enlargement
description: Use when gap-filling multiple draft genome-scale metabolic models of
  a microbial community (e.g. consensus reconstructions from KBase, CarveMe, RAVEN
  2.0, AuReMe/Pathway Tools) and you want the solution to respect community composition
  and metabolite leakage.
when_to_use_negative:
- Gap-filling a single organism's reconstruction with no community context — leak-mediated
  medium enlargement has no meaning and standard individual gap filling is appropriate.
- 'When the goal is whole-community optimization via joint FBA-style methods (e.g.
  SteadyCom, MICOM) rather than per-member gap-filling: COMMIT is designed for gap-filling
  draft reconstructions, not community simulation.'
- When the draft models were not first normalized to MNXref identifiers and a common
  format — identifier mismatch across KBase/CarveMe/RAVEN/AuReMe outputs invalidates
  the shared medium and database.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3954
tools:
- name: COMMIT
  role: Implements the community-dependent gap-filling loop and medium-enlargement
    logic
  repo: github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: Provides the FastGapFilling LP solver used to gap-fill each member reconstruction
  repo: github.com/opencobra/cobratoolbox
- name: RAVEN 2.0
  role: Source of one of the four draft reconstructions used as gap-filling input
  repo: github.com/SysBioChalmers/RAVEN
- name: CarveMe
  role: Source of one of the four draft reconstructions used as gap-filling input
  repo: github.com/cdanielmachado/carveme
- name: HMMER
  role: Supplies sequence-similarity support terms with E-value cutoff 10^-6 for the
    gap-filling weights
- name: IBM CPLEX
  role: LP solver used with the COBRA toolbox for gap-filling optimization
- name: Gurobi
  role: Alternative LP solver used with the COBRA toolbox for gap-filling optimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:0dc6beb4a0028ebf11d914251221cefd8babda368dfab835506f9a24bb75e032
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRA toolbox
  - COMMIT
  - CarveMe
  - Gurobi
  - HMMER
  - IBM CPLEX
  - RAVEN 2.0
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3954
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-leakage-medium-enlargement

## Summary

COMMIT's gap-filling strategy that enlarges the shared community medium with metabolites secreted by already gap-filled member reconstructions, so subsequent members can take up leaked metabolites instead of receiving extra gap-filling reactions.

## When to use

Use when gap-filling multiple draft genome-scale metabolic models of a microbial community (e.g. consensus reconstructions from KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) and you want the solution to respect community composition and metabolite leakage.

## When NOT to use

- Gap-filling a single organism's reconstruction with no community context — leak-mediated medium enlargement has no meaning and standard individual gap filling is appropriate.
- When the goal is whole-community optimization via joint FBA-style methods (e.g. SteadyCom, MICOM) rather than per-member gap-filling: COMMIT is designed for gap-filling draft reconstructions, not community simulation.
- When the draft models were not first normalized to MNXref identifiers and a common format — identifier mismatch across KBase/CarveMe/RAVEN/AuReMe outputs invalidates the shared medium and database.

## Inputs

- Draft genome-scale metabolic reconstructions of community members (Matlab .mat cell arrays under data/models/<habitat>/<method>)
- Universal MNXref gap-filling database (dbModel_MNXref_balanced)
- Community composition (member proportions)
- Minimal medium definition

## Outputs

- Gap-filled member reconstructions consistent with the community composition
- An optimal member ordering over 100 random permutations that minimizes total added reactions while preserving biomass production and genomic support
- A medium enlarged with metabolites secreted by already gap-filled members

## How to apply

Start from a minimal medium derived as the intersection of the organisms' auxotrophic media, excluding amino acids and carbon sources except glucose; also remove any medium used during CarveMe reconstruction. For each member model in a random ordering, compute exportable metabolites by applying permeability rules (HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6) at a 10% biomass-reduction threshold (suboptimal biomass factor f=0.9 for sink reaction maximization), and solve the FastGapFilling LP with penalty weights of 100 for transport, 50 for metabolic non-transport, 25 for directionality change, 50 for permeable transport, 25 for sequence support (HMMER E-value cutoff 10^-6), 1 for uptake of secreted metabolites, and 10^5 for exchange, with biomass flux bounds of 10^-3 (lower) and 2.81 h^-1 (upper). After each member is gap-filled, enlarge the shared medium with its exportable metabolites so that subsequent members can take up leaked metabolites — this reduces the added-reaction sets of later members. Iterate over 100 random member orderings and select the ordering that satisfies the comparison criteria (e.g. smaller total added-reaction set than individual gap filling while maintaining biomass production and genomic support).

## Related tools

- **COMMIT** (Implements the community-dependent gap-filling loop and medium-enlargement logic) — github.com/pwendering/COMMIT
- **COBRA toolbox** (Provides the FastGapFilling LP solver used to gap-fill each member reconstruction) — github.com/opencobra/cobratoolbox
- **RAVEN 2.0** (Source of one of the four draft reconstructions used as gap-filling input) — github.com/SysBioChalmers/RAVEN
- **CarveMe** (Source of one of the four draft reconstructions used as gap-filling input) — github.com/cdanielmachado/carveme
- **HMMER** (Supplies sequence-similarity support terms with E-value cutoff 10^-6 for the gap-filling weights)
- **IBM CPLEX** (LP solver used with the COBRA toolbox for gap-filling optimization)
- **Gurobi** (Alternative LP solver used with the COBRA toolbox for gap-filling optimization)

## Examples

```
git clone https://github.com/pwendering/COMMIT; matlab -r "run_iterative_gap_filling(community, medium, dbModel_MNXref_balanced, 100)"
```

## Evaluation signals

- The selected ordering must yield a smaller total added-reaction set than gap-filling the individual reconstructions without community consideration.
- Biomass production must be maintained in every member after gap filling (biomass flux lower bound 10^-3, upper bound 2.81 h^-1 per isolate model).
- Added reactions must retain genomic support under the HMMER E-value cutoff of 10^-6.
- Compare the 100 random orderings against the specified criteria and confirm an optimal ordering exists that reduces added reactions.
- Verify that the shared medium grows monotonically as members are processed, containing exactly the exportable metabolites of already gap-filled members.

## Limitations

- Penalty weights were tuned ad hoc ('altered until satisfying results were obtained for selected reconstructions'); no systematic selection procedure is documented.
- The full COMMIT procedure specification is deferred to supplementary S1 Text and is not in the article body.
- Only 26% of InChI keys translated to PubChem CIDs and only 11% had XLogP3 values, so many metabolites lack properties and are classified as not likely permeable.
- Criteria for choosing the 'best ordering' are referenced but not fully specified in the article text.
- The boundary between helper and beneficiary roles depends on the community composition input; results are specific to the two soil communities (Bulgarelli, Schlaeppi) analyzed.

## Evidence

- [intro] Metabolites secreted by already gap-filled members enlarge the shared medium: "the enlarged medium is expected to reduce the added reaction sets of subsequent members"
- [intro] COMMIT aims at a minimal gap-filling solution respecting community composition: "COMMIT, aims to identify a minimal gap-filling solution that respects the composition of the microbial community"
- [discussion] Permeability thresholds used to classify exportable metabolites: "the applied rules were: HBD≤5,HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6."
- [discussion] Suboptimal biomass factor f=0.9 for sink reaction maximization: "In this study, f = 0.9 has been used (please see S1 Text for more detail)."
- [discussion] Weights were tuned by trial rather than systematically selected: "The penalties applied in this study resulted from the comparison of different weights that were altered until satisfying results were obtained for selected reconstructions"
- [intro] COMMIT decreased added reactions relative to individual gap filling: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
- [readme] Repository contains community-dependent gap-filling pipeline for two soil communities: "This repository contains all files used for community-dependent gap-filling and subsequent analysis of consensus genome-scale metabolic reconstructions for two soil communities (Bulgarelli et al."
