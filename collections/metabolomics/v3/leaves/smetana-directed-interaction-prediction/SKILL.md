---
name: smetana-directed-interaction-prediction
description: Use when you have a collection of genome-scale metabolic models (e.g.,
  SBML from gap-filled community members) and need to predict which community member
  pairs cross-feed which metabolites under a defined medium.
when_to_use_negative:
- You need community-level flux distributions from joint optimization rather than
  pairwise cross-feeding potentials — use whole-community optimization approaches
  like SteadyCom or MICOM instead
- Your community members are not reconstructed/gap-filled genome-scale metabolic models
  in a SMETANA-readable (SBML) format; interaction prediction requires functional
  models first
- You only have a single isolate model with no community context, since pairwise donor-to-receiver
  exchanges cannot be computed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SMETANA
  role: Computes the directed donor-to-receiver metabolite cross-feeding metrics between
    community member models under the community medium
  repo: github.com/cdanielmachado/smetana
- name: COMMIT
  role: Provides the gap-filled community member models that are converted to SMETANA
    input; also supplies a second set of predicted exchanges for cross-validation
  repo: github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: Matlab environment in which the COMMIT iterative gap-filling (run_iterative_gap_filling.m)
    producing the input models is run
  repo: github.com/opencobra/cobratoolbox
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:4c16dcd28d3bdd8b95bda96dbe66473e2899640b5b58321d094e70a022ee3fa3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - SMETANA
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# smetana-directed-interaction-prediction

## Summary

Runs SMETANA over gap-filled genome-scale metabolic community models to predict directed donor-to-receiver metabolite exchanges between member pairs, then symmetrizes them into pairwise interaction potentials for cross-feeding analysis.

## When to use

Use when you have a collection of genome-scale metabolic models (e.g., SBML from gap-filled community members) and need to predict which community member pairs cross-feed which metabolites under a defined medium.

## When NOT to use

- You need community-level flux distributions from joint optimization rather than pairwise cross-feeding potentials — use whole-community optimization approaches like SteadyCom or MICOM instead
- Your community members are not reconstructed/gap-filled genome-scale metabolic models in a SMETANA-readable (SBML) format; interaction prediction requires functional models first
- You only have a single isolate model with no community context, since pairwise donor-to-receiver exchanges cannot be computed

## Inputs

- Gap-filled community member genome-scale metabolic models (e.g., COMMIT gap-filled Schlaeppi community, 24 members, converted to SMETANA/SBML input format)
- Community medium definition (e.g., minimal medium from the intersection of KBase auxotrophic media)
- Optional second interaction-prediction result set (e.g., COMMIT exchange predictions) for agreement analysis

## Outputs

- Directed metabolite exchange predictions per member pair with SMETANA exchange scores
- Symmetrized undirected pairwise interaction matrix between community members
- Agreement statistics: MIP (0.98 for Schlaeppi), number of interacting members (17 of 24), fraction of possible interactions predicted by both methods (43%), total agreement (60%), and the intersection of shared metabolites (eight metabolites including L-arginine, L-asparagine, L-lysine, and NH4+)

## How to apply

Load the gap-filled community member models (here, produced by COMMIT's iterative gap-filling, run_iterative_gap_filling.m) and convert them to SMETANA input format as a collection of SBML models. Run SMETANA (github.com/cdanielmachado/smetana, installed via pip) over all member pairs under the community medium to obtain directed donor-to-receiver metabolite exchanges with exchange scores. Parse the SMETANA output to extract directed interactions per metabolite, then symmetrize the directed interactions into undirected pairwise overlaps between community members. Finally compute agreement statistics against a second interaction-prediction method (e.g., COMMIT): member interaction potential (MIP), the number of members with predicted interactions, the fraction of possible interactions predicted by both methods, total agreement (shared predictions plus shared non-interacting pairs), and the intersection of shared metabolites across predicted interactions.

## Related tools

- **SMETANA** (Computes the directed donor-to-receiver metabolite cross-feeding metrics between community member models under the community medium) — github.com/cdanielmachado/smetana
- **COMMIT** (Provides the gap-filled community member models that are converted to SMETANA input; also supplies a second set of predicted exchanges for cross-validation) — github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (Matlab environment in which the COMMIT iterative gap-filling (run_iterative_gap_filling.m) producing the input models is run) — github.com/opencobra/cobratoolbox

## Examples

```
pip install smetana && smetana community_models/*.sbml --medium medium.csv
```

## Evaluation signals

- For the Schlaeppi reference community, MIP should equal 0.98 and predicted interactions should involve 17 of 24 members
- Cross-validation against COMMIT should yield 43% of possible isolate-pair interactions predicted by both methods and 60% total agreement (43% shared interactions + 17% shared non-interacting pairs)
- The metabolite intersection across predicted interactions should contain exactly eight metabolites, including L-arginine, L-asparagine, L-lysine, and NH4+
- SMETANA output must resolve to directed interactions before symmetrization; verify each predicted pair carries the exchanged metabolite and its exchange score
- Input models must load in SMETANA after conversion from the COMMIT/COBRA format; conversion failures (identifier mismatches) indicate models were not properly mapped to a common ID space

## Limitations

- SMETANA predicts interaction potential, not realized fluxes; agreement with a second method (COMMIT) was only 60%, with 43% of possible pairs co-predicted as interacting and 17% co-predicted as non-interacting
- Only sensitivity and precision could be computed in the underlying study since true negatives could not be determined
- Comparison/merging of models was hampered by lack of reaction cross-references to other namespaces, so identifier translation was only done 'whenever possible', which can affect exchange metabolite matching

## Evidence

- [intro] SMETANA-based directed interaction prediction: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [supplementary] SMETANA input format and implementation source: [evidence span withheld — anchor status: not_found]
- [readme] SMETANA accepts a collection of genome-scale metabolic models: [evidence span withheld — anchor status: not_found]
- [abstract] Interaction inspection identifies community roles: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
