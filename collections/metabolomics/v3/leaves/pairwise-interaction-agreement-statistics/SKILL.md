---
name: pairwise-interaction-agreement-statistics
description: Use when two independent methods (e.g., SMETANA and COMMIT) have predicted
  metabolite exchanges over the same gap-filled community member models and you need
  to quantify how much their pairwise interaction predictions agree, including shared
  metabolites.
when_to_use_negative:
- Only one interaction-prediction method is available, so no cross-method agreement
  can be computed
- The community member models are raw draft reconstructions rather than COMMIT gap-filled
  models, since agreement statistics depend on the gap-filling solution
- The goal requires donor-to-receiver directionality of exchanges rather than symmetrized
  undirected pairwise overlaps
tools:
- name: SMETANA
  role: Predicts directed donor-to-receiver metabolite exchanges between community
    member pairs under the community medium; output is parsed and symmetrized for
    the agreement statistics
  repo: https://github.com/cdanielmachado/smetana
- name: COMMIT
  role: Provides the gap-filled community member models via run_iterative_gap_filling.m
    and the second set of predicted exchanges against which SMETANA agreement is computed
  repo: https://github.com/pwendering/COMMIT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:e806d7e1363fba94229149725d0e3dcffc6e456a6ddd58d126f2f5ea20ad857e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cdanielmachado/smetana
  tools:
  - COMMIT
  - SMETANA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pairwise-interaction-agreement-statistics

## Summary

Cross-validates metabolic interaction predictions by comparing directed donor-to-receiver metabolite exchanges from SMETANA with exchanges predicted by COMMIT over the same gap-filled community models, symmetrizing directed interactions into undirected pairwise overlaps and computing agreement statistics such as member interaction potential (MIP), fraction of co-predicted pairs, total agreement, and the shared-metabolite intersection.

## When to use

Use when two independent methods (e.g., SMETANA and COMMIT) have predicted metabolite exchanges over the same gap-filled community member models and you need to quantify how much their pairwise interaction predictions agree, including shared metabolites.

## When NOT to use

- Only one interaction-prediction method is available, so no cross-method agreement can be computed
- The community member models are raw draft reconstructions rather than COMMIT gap-filled models, since agreement statistics depend on the gap-filling solution
- The goal requires donor-to-receiver directionality of exchanges rather than symmetrized undirected pairwise overlaps

## Inputs

- Gap-filled community member genome-scale metabolic models (SBML) produced by COMMIT iterative gap-filling for the Schlaeppi community
- Community medium definition under which exchanges are evaluated
- SMETANA output with directed interactions and metabolite exchange scores
- COMMIT-predicted exchanges for the same community

## Outputs

- Member interaction potential (MIP) value (expected 0.98 for the Schlaeppi community)
- Number of members with predicted interactions (17 of 24 members)
- Fraction of possible isolate-pair interactions predicted by both methods (43%)
- Fraction of pairs predicted non-interacting by both (17%) and total agreement (60%)
- Intersection of shared metabolites across predicted interactions (exactly eight metabolites, including L-arginine, L-asparagine, L-lysine, NH4+)

## How to apply

Load the gap-filled community member models produced by the COMMIT iterative gap-filling procedure (run_iterative_gap_filling.m) and convert them to SMETANA input format (SBML). Run SMETANA over all member pairs under the community medium to predict directed donor-to-receiver exchanges with their metabolite exchange scores, then parse the SMETANA output to extract the directed interactions. Symmetrize the directed interactions into undirected pairwise overlaps between community members. Compute the agreement statistics: member interaction potential (MIP), number of members with predicted interactions, fraction of possible interactions predicted by both methods, fraction predicted non-interacting by both, total agreement, and the intersection of shared metabolites across predicted interactions. Rationale: symmetrization is needed because SMETANA returns directed exchanges whereas the agreement question is whether an interaction exists between a pair under either method.

## Related tools

- **SMETANA** (Predicts directed donor-to-receiver metabolite exchanges between community member pairs under the community medium; output is parsed and symmetrized for the agreement statistics) — https://github.com/cdanielmachado/smetana
- **COMMIT** (Provides the gap-filled community member models via run_iterative_gap_filling.m and the second set of predicted exchanges against which SMETANA agreement is computed) — https://github.com/pwendering/COMMIT

## Examples

```
pip install smetana
```

## Evaluation signals

- Computed member interaction potential (MIP) equals 0.98
- Predicted interactions involve exactly 17 of the 24 community members
- 43% of all possible isolate-pair interactions are predicted by both SMETANA and COMMIT, and 17% by neither, giving 60% total agreement
- The shared-metabolite intersection contains exactly eight metabolites, including L-arginine, L-asparagine, L-lysine, and NH4+
- Every undirected pair in the agreement table corresponds to at least one directed SMETANA interaction after symmetrization

## Limitations

- Symmetrizing directed SMETANA exchanges into undirected pairwise overlaps discards donor-to-receiver directionality
- Agreement results are conditional on the COMMIT gap-filling solution, whose penalty weights were tuned ad hoc without a systematic selection procedure
- SMETANA predictions depend on the chosen community medium; changing the medium changes the interaction set and hence the agreement statistics

## Evidence

- [readme] SMETANA computes directed cross-feeding metrics over community models: [evidence span withheld — anchor status: not_found]
- [intro] SMETANA used as downstream analysis of gap-filled models: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [abstract] Interaction inspection identifies community roles: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [supplementary] SMETANA implementation source cited in supplementary: "(implementation from github.com/cdanielmachado/smetana)"
