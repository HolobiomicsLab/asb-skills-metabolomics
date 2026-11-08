---
name: community-metabolic-model-symmetrization
description: Use when two interaction-prediction methods (e.g., SMETANA and COMMIT)
  have been run over the same gap-filled community models and you need to compare
  them via symmetrized pairwise overlaps, MIP, and shared-metabolite intersections.
when_to_use_negative:
- The models are draft (not gap-filled) reconstructions — interaction predictions
  on incomplete models would not reflect the community-dependent gap-filling this
  comparison presupposes
- The question requires directional donor-to-receiver information — symmetrization
  deliberately collapses direction into undirected pairwise overlaps
- Only a single prediction method is available and no second interaction set exists
  for cross-validation
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: SMETANA
  role: Predicts directed donor-to-receiver metabolite exchanges between community
    member pairs under the community medium; output is parsed and symmetrized
  repo: github.com/cdanielmachado/smetana
- name: COMMIT
  role: Produces the community-dependent gap-filled member models whose SMETANA interactions
    are cross-validated, and provides the second interaction-prediction method for
    the agreement statistics
  repo: github.com/pwendering/COMMIT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:2b5f61c27147ef5b6ff028c2f5dab94cc0d444498cacb80bd2c27e45508b6016
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMMIT
  - SMETANA
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# community-metabolic-model-symmetrization

## Summary

Cross-validation of predicted metabolic interactions: run SMETANA over gap-filled community member models (e.g., COMMIT-gap-filled Schlaeppi community), then symmetrize directed donor-to-receiver metabolite exchanges into undirected pairwise overlaps to compute agreement statistics against a second interaction-prediction method.

## When to use

Use when two interaction-prediction methods (e.g., SMETANA and COMMIT) have been run over the same gap-filled community models and you need to compare them via symmetrized pairwise overlaps, MIP, and shared-metabolite intersections.

## When NOT to use

- The models are draft (not gap-filled) reconstructions — interaction predictions on incomplete models would not reflect the community-dependent gap-filling this comparison presupposes
- The question requires directional donor-to-receiver information — symmetrization deliberately collapses direction into undirected pairwise overlaps
- Only a single prediction method is available and no second interaction set exists for cross-validation

## Inputs

- Gap-filled genome-scale metabolic models for each community member (COMMIT iterative gap-filling output, e.g., from run_iterative_gap_filling.m)
- Models converted to SMETANA input format (SBML community model collection)
- Community medium definition used for interaction prediction
- Directed interaction list from a second method (e.g., COMMIT-predicted exchanges) for the overlap comparison

## Outputs

- Symmetrized undirected pairwise interaction matrix over community members
- Member interaction potential (MIP, e.g., 0.98 for the Schlaeppi community)
- Fraction of possible isolate-pair interactions predicted by both methods (43%) and total agreement (60%)
- Intersection of shared metabolites across predicted interactions (8 metabolites, incl. L-arginine, L-asparagine, L-lysine, NH4+)

## How to apply

Load the gap-filled community member models produced by COMMIT's iterative gap-filling (run_iterative_gap_filling.m) and convert them to SMETANA input format. Run SMETANA (implementation from github.com/cdanielmachado/smetana) over all member pairs under the community medium to obtain directed donor-to-receiver metabolite exchanges, then parse the SMETANA output to extract directed interactions with their metabolite exchange scores. Symmetrize the directed interactions into undirected pairwise overlaps between community members so that interactions predicted by either method in either direction become comparable. Compute member interaction potential (MIP), the number of members with predicted interactions, the fraction of possible interactions predicted by both methods, total agreement, and the intersection of shared metabolites across predicted interactions. For the Schlaeppi community this yielded MIP = 0.98, interactions involving 17 of 24 members, 43% of possible isolate-pair interactions predicted by both SMETANA and COMMIT, 17% predicted non-interacting by both (60% total agreement), and an 8-metabolite intersection including L-arginine, L-asparagine, L-lysine, and NH4+.

## Related tools

- **SMETANA** (Predicts directed donor-to-receiver metabolite exchanges between community member pairs under the community medium; output is parsed and symmetrized) — github.com/cdanielmachado/smetana
- **COMMIT** (Produces the community-dependent gap-filled member models whose SMETANA interactions are cross-validated, and provides the second interaction-prediction method for the agreement statistics) — github.com/pwendering/COMMIT

## Examples

```
pip install smetana
```

## Evaluation signals

- MIP value matches the reported 0.98 for the gap-filled Schlaeppi community
- Interactions involve exactly 17 of the 24 community members
- 43% of all possible isolate-pair interactions are predicted by both SMETANA and COMMIT and 17% by neither (60% total agreement)
- The shared-metabolite intersection contains exactly eight metabolites including L-arginine, L-asparagine, L-lysine, and NH4+

## Limitations

- Symmetrization discards exchange directionality, so donor–receiver roles of helpers and beneficiaries cannot be recovered from the undirected overlap
- Agreement statistics are specific to the community medium and the COMMIT gap-filled models; different media or gap-filling orderings will change the interaction sets
- SMETANA scores represent the potential for cross-feeding interactions rather than observed fluxes, limiting direct biological interpretation

## Evidence

- [intro] Downstream analysis with SMETANA on gap-filled models: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [methods] Workflow: parse directed SMETANA output and symmetrize into undirected overlaps: [evidence span withheld — anchor status: not_found]
- [results] Validated agreement statistics for the Schlaeppi community: [evidence span withheld — anchor status: not_found]
- [readme] SMETANA README: community models as input, cross-feeding metrics as output: [evidence span withheld — anchor status: external_doc]
