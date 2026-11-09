---
name: cross-feeding-metabolite-overlap-analysis
description: Use when two independent metabolic interaction predictors (e.g. SMETANA
  and COMMIT) have been run on the same gap-filled community models and you need to
  quantify their agreement at the level of member pairs and shared exchanged metabolites.
when_to_use_negative:
- The models are draft (not gap-filled) reconstructions — SMETANA interaction predictions
  require functional, gap-filled member models
- The analysis targets a single isolate model rather than a multi-member community
  with candidate pairwise cross-feeding
- You need directed donor-to-receiver exchange semantics from SMETANA itself — this
  skill symmetrizes directed interactions into undirected pairwise overlaps
edam_operation: http://edamontology.org/operation_0292
edam_topics: []
tools:
- name: SMETANA
  role: Computes cross-feeding interaction metrics over all member pairs under the
    community medium; installed via pip and run on the gap-filled models
  repo: github.com/cdanielmachado/smetana
- name: COMMIT
  role: Produces the community-dependent gap-filled isolate models and the independent
    interaction predictions being cross-validated
  repo: github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB environment in which the COMMIT iterative gap-filling procedure that
    generated the input models runs
  repo: github.com/opencobra/cobratoolbox
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:4049788122e9311188d2b0add6d8002a4162d90c064c19fc76e57120e3d4bddc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRA toolbox
  - COMMIT
  - SMETANA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-feeding-metabolite-overlap-analysis

## Summary

Cross-validates community metabolic interaction predictions from two independent methods (SMETANA and COMMIT) by converting gap-filled isolate models to SMETANA input, symmetrizing directed exchanges into pairwise overlaps, and computing agreement statistics (MIP, fraction of interactions predicted by both, shared-metabolite intersection).

## When to use

Use when two independent metabolic interaction predictors (e.g. SMETANA and COMMIT) have been run on the same gap-filled community models and you need to quantify their agreement at the level of member pairs and shared exchanged metabolites.

## When NOT to use

- The models are draft (not gap-filled) reconstructions — SMETANA interaction predictions require functional, gap-filled member models
- The analysis targets a single isolate model rather than a multi-member community with candidate pairwise cross-feeding
- You need directed donor-to-receiver exchange semantics from SMETANA itself — this skill symmetrizes directed interactions into undirected pairwise overlaps

## Inputs

- Gap-filled genome-scale metabolic models for each community member, produced by COMMIT iterative gap-filling (run_iterative_gap_filling.m), converted to SBML for SMETANA
- Community medium definition used during COMMIT gap-filling (e.g. the minimal medium derived from the intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)
- COMMIT-predicted directed metabolite exchanges for the same community (comparison reference)

## Outputs

- Member interaction potential (MIP) value for the community
- Count of community members participating in predicted interactions
- Fraction of all possible isolate-pair interactions predicted by both SMETANA and COMMIT, and total agreement percentage including pairs predicted non-interacting by both
- Intersection set of metabolites shared across predicted interactions (e.g. eight metabolites including L-arginine, L-asparagine, L-lysine, and NH4+)

## How to apply

Load the gap-filled community member models (e.g. the 24-member Schlaeppi community produced by the COMMIT iterative gap-filling procedure, run_iterative_gap_filling.m) and convert them to SMETANA input (SBML) format. Run SMETANA over all member pairs under the community medium to obtain directed donor-to-receiver metabolite exchanges with scores. Symmetrize these directed interactions into undirected pairwise overlaps between community members, then compute the member interaction potential (MIP), the number of members with predicted interactions, the fraction of possible interactions predicted by both methods, the total agreement (both-predicted plus both-non-interacting pairs), and the intersection of shared metabolites across predicted interactions. The rationale is that agreement between SMETANA's cross-feeding metrics and COMMIT's community-composition-aware gap-filling exchange predictions provides orthogonal validation that the interaction landscape is not an artifact of a single method.

## Related tools

- **SMETANA** (Computes cross-feeding interaction metrics over all member pairs under the community medium; installed via pip and run on the gap-filled models) — github.com/cdanielmachado/smetana
- **COMMIT** (Produces the community-dependent gap-filled isolate models and the independent interaction predictions being cross-validated) — github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB environment in which the COMMIT iterative gap-filling procedure that generated the input models runs) — github.com/opencobra/cobratoolbox

## Examples

```
pip install smetana
```

## Evaluation signals

- Computed MIP for the Schlaeppi community should equal 0.98
- Predicted interactions should involve 17 of the 24 community members
- About 43% of all possible isolate-pair interactions should be predicted by both SMETANA and COMMIT, giving 60% total agreement when 17% of pairs predicted non-interacting by both are included
- The metabolite intersection across predicted interactions should contain exactly eight metabolites, including L-arginine, L-asparagine, L-lysine, and NH4+
- Directed SMETANA outputs must be symmetrized consistently so pairwise overlaps are undirected and comparable to COMMIT's interaction sets

## Limitations

- Agreement is only 60% overall, so a substantial fraction of possible interactions is method-discrepant; the 43% both-predicted rate should not be read as full concordance
- SMETANA output must be symmetrized before comparison, losing directionality of donor-to-receiver exchanges
- Results depend on the community medium chosen for the interaction runs (here, the minimal medium derived from the intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)

## Evidence

- [intro] we conducted a downstream analysis of the gap-filled models using the SMETANA tool: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [readme] SMETANA README input/output description: [evidence span withheld — anchor status: external_doc]
- [supplementary] SMETANA implementation reference: "(implementation from github.com/cdanielmachado/smetana)"
- [readme] SMETANA citation and algorithm provenance: [evidence span withheld — anchor status: external_doc]
- [methods] COMMIT gap-filling procedure generating the input models: "git clone https://github.com/pwendering/COMMIT"
