---
name: constraint-based-community-model-evaluation
description: Use when you have gap-filled community member GEMs (e.g., COMMIT-output
  SBML models) plus a community medium, and need to predict or cross-validate pairwise
  cross-feeding interactions and community interaction potential.
when_to_use_negative:
- Input is a set of draft, un-gap-filled single-organism reconstructions with no shared
  community medium — interaction scores computed on unfilled drafts are not comparable
  to the COMMIT-based cross-validation
- The goal is to generate draft reconstructions from genome data (use KBase, CarveMe,
  RAVEN 2.0, or AuReMe/Pathway Tools reconstruction workflows instead of interaction
  evaluation)
- You need whole-community flux simulation rather than pairwise interaction potential
  (SteadyCom or MICOM perform whole-community optimization)
edam_operation: http://edamontology.org/operation_3439
tools:
- name: SMETANA
  role: Runs species metabolic interaction analysis over all community member pairs
    to predict directed cross-feeding exchanges under the community medium
  repo: https://github.com/cdanielmachado/smetana
- name: COMMIT
  role: Produces the gap-filled community member models whose predicted interactions
    serve as one side of the cross-validation
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox v3.0
  role: Matlab dependency used to run the COMMIT iterative gap-filling procedure that
    generates the evaluated models
  repo: https://github.com/opencobra/cobratoolbox
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:79c039842880618ad0d5f9749babf969c5465b9e42d58713d6796e37693416cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cdanielmachado/smetana
  tools:
  - COBRA toolbox v3.0
  - COMMIT
  - SMETANA
  edam_operation: http://edamontology.org/operation_3439
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# constraint-based-community-model-evaluation

## Summary

Evaluates gap-filled community genome-scale metabolic models by cross-validating predicted metabolic interactions: SMETANA is run over all member pairs under the community medium and its directed donor-to-receiver exchanges are compared against COMMIT's predicted interactions. For the Schlaeppi Arabidopsis soil community this yields MIP, agreement fractions, and a shared-metabolite intersection that validate the gap-filling result.

## When to use

Use when you have gap-filled community member GEMs (e.g., COMMIT-output SBML models) plus a community medium, and need to predict or cross-validate pairwise cross-feeding interactions and community interaction potential.

## When NOT to use

- Input is a set of draft, un-gap-filled single-organism reconstructions with no shared community medium — interaction scores computed on unfilled drafts are not comparable to the COMMIT-based cross-validation
- The goal is to generate draft reconstructions from genome data (use KBase, CarveMe, RAVEN 2.0, or AuReMe/Pathway Tools reconstruction workflows instead of interaction evaluation)
- You need whole-community flux simulation rather than pairwise interaction potential (SteadyCom or MICOM perform whole-community optimization)

## Inputs

- Gap-filled community member genome-scale metabolic models in SBML format (e.g., the 24-member Schlaeppi community from COMMIT gap filling)
- Community medium definition (nutrient set shared across members, e.g., derived from the intersection of KBase auxotrophic media excluding amino acids and carbon sources except glucose)
- COMMIT-predicted directed interaction/metabolite exchange lists for comparison
- SMETANA installation (pip installable; github.com/cdanielmachado/smetana)

## Outputs

- Directed donor-to-receiver metabolite exchange predictions with metabolite exchange scores per isolate pair
- Symmetrized undirected pairwise interaction matrix over community members
- Member interaction potential (MIP) and count of members with predicted interactions
- Agreement statistics: fraction of possible interactions predicted by both methods and total agreement
- Intersection set of shared metabolites across predicted interactions

## How to apply

Load the gap-filled community member models produced by the COMMIT iterative gap-filling procedure (run_iterative_gap_filling.m from the pwendering/COMMIT repository) and convert them to SMETANA input format (a collection of genome-scale metabolic models in SBML). Run SMETANA (github.com/cdanielmachado/smetana) over all member pairs under the community medium to predict directed donor-to-receiver metabolite exchanges, then parse the SMETANA output to extract directed interactions with their metabolite exchange scores. Symmetrize the directed interactions into undirected pairwise overlaps between community members, and compute the member interaction potential (MIP), the number of members with predicted interactions, the fraction of possible interactions predicted by both SMETANA and COMMIT, total agreement, and the intersection of shared metabolites across predicted interactions. The rationale is that agreement between two independently formulated interaction-prediction methods (COMMIT gap-filling vs. SMETANA scoring) validates that the gap-filled community supports a consistent cross-feeding structure; on the Schlaeppi community this validation reached 60% total agreement with a MIP of 0.98.

## Related tools

- **SMETANA** (Runs species metabolic interaction analysis over all community member pairs to predict directed cross-feeding exchanges under the community medium) — https://github.com/cdanielmachado/smetana
- **COMMIT** (Produces the gap-filled community member models whose predicted interactions serve as one side of the cross-validation) — https://github.com/pwendering/COMMIT
- **COBRA toolbox v3.0** (Matlab dependency used to run the COMMIT iterative gap-filling procedure that generates the evaluated models) — https://github.com/opencobra/cobratoolbox

## Examples

```
pip install smetana
```

## Evaluation signals

- Computed member interaction potential (MIP) equals 0.98 for the Schlaeppi community
- Predicted interactions involve exactly 17 of the 24 community members
- About 43% of all possible isolate-pair interactions are predicted by both SMETANA and COMMIT
- Total agreement (both-interacting plus both-non-interacting pairs) reaches 60%
- The shared-metabolite intersection contains exactly eight metabolites, including L-arginine, L-asparagine, L-lysine, and NH4+

## Limitations

- Agreement is assessed only between SMETANA and COMMIT; true negatives for interaction absence cannot be determined, so only overlap-style measures (sensitivity/precision-type) are available, not full accuracy
- Results depend on the community medium: the minimal medium excludes amino acids and carbon sources except glucose, and metabolites lacking PubChem properties were classified as not likely permeable by COMMIT
- Only 26% of InChI keys could be translated to PubChem CIDs and only 11% had XLogP3 values, so permeability-based leakage predictions are incomplete for many metabolites
- SMETANA predictions require converting models from the COMMIT pipeline (Matlab/SBML) into SMETANA's expected input format; identifier mismatches between name spaces can hamper comparisons

## Evidence

- [intro] SMETANA downstream analysis on gap-filled models: "we conducted a downstream analysis of the gap-filled models using the SMETANA tool"
- [supplementary] SMETANA implementation source: "(implementation from github.com/cdanielmachado/smetana)"
- [abstract] Purpose of interaction inspection: "Inspection of the metabolic interactions in the soil communities allows us to identify microbes with community roles of helpers and beneficiaries"
- [readme] SMETANA input format from README: [evidence span withheld — anchor status: not_found]
- [readme] SMETANA description from README: [evidence span withheld — anchor status: external_doc]
