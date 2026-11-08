---
name: accurate-mass-candidate-filtering
description: Use when annotating LC-MS features in untargeted metabolomics where only
  m/z and RTs are available and fragmentation spectra are not, so candidates must
  be selected by accurate mass before retention-time scoring.
when_to_use_negative:
- Fragmentation spectra are available, allowing higher-confidence annotation by MS/MS
  matching rather than mass filtering alone
- The available mass accuracy is much worse than 10 ppm (e.g. low-resolution MS),
  since the filter would not narrow the candidate set effectively
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: cmmrt
  role: Python package implementing the predicted-RT database, GP projection, and
    the mass-filter plus z-score ranking workflow
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the ECFP, MACCSFP and PFP fingerprints used to build the predicted-RT
    database being filtered by mass
- name: CEU Mass Mediator
  role: Metabolite annotation platform into which the authors intend to integrate
    this mass-filter and projection approach
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:51c932c6ea1c68a486f610ada78abee587f39adaf2ee9c4af97170a63b43bd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - CEU Mass Mediator
  - Python
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# accurate-mass-candidate-filtering

## Summary

Filter the predicted-RT metabolite database by accurate mass at a 10 ppm tolerance to obtain candidate annotations for each query feature, the first step of the paper's 7-step annotation workflow before RT-based z-score ranking.

## When to use

Use when annotating LC-MS features in untargeted metabolomics where only m/z and RTs are available and fragmentation spectra are not, so candidates must be selected by accurate mass before retention-time scoring.

## When NOT to use

- Fragmentation spectra are available, allowing higher-confidence annotation by MS/MS matching rather than mass filtering alone
- The available mass accuracy is much worse than 10 ppm (e.g. low-resolution MS), since the filter would not narrow the candidate set effectively

## Inputs

- Experimental m/z values of query LC-MS features
- Predicted-RT database of 6,823 HMDB molecules with KEGG numbers (ECFP/MACCSFP/PFP fingerprints from alvaDesc)
- Simulated mass-noise model for validation: N(0, (10/3 ppm)^2) truncated to ±10 ppm

## Outputs

- Per-query candidate lists (only queries with >3 candidates retained)
- Ranked candidate lists by projected-RT z-score for downstream top-3 accuracy evaluation
- Mass-only random-tie-break baseline accuracy for comparison

## How to apply

Query the HMDB-derived predicted-RT database (6,823 KEGG-numbered molecules with SMRT-trained DNN fingerprints) with each experimental m/z at 10 ppm tolerance; molecules outside the window are discarded. For validation experiments, add simulated mass error drawn from N(0, (10/3 ppm)^2), truncated to ±10 ppm so the correct molecule is always retained as a candidate. Restrict evaluation to queries that return more than three candidates, since the downstream ranking skill scores top-3 accuracy. Then rank the surviving candidates by the GP-projected RT z-score z = |y* - mu(x*)| / sigma(x*) rather than by mass alone, and compare against a mass-only random-tie-break baseline to quantify the added value of RT information.

## Related tools

- **cmmrt** (Python package implementing the predicted-RT database, GP projection, and the mass-filter plus z-score ranking workflow) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the ECFP, MACCSFP and PFP fingerprints used to build the predicted-RT database being filtered by mass)
- **CEU Mass Mediator** (Metabolite annotation platform into which the authors intend to integrate this mass-filter and projection approach)

## Examples

```
make test_projections
```

## Evaluation signals

- With only mass filtering (random tie-break), FEM long baseline top-3 accuracy should be ~59.57%
- Adding GP-projected RT z-scores should lift global top-3 accuracy to ~68% with 10 training molecules and ~70% with 50
- Every retained query must have returned more than three candidates from the 10 ppm mass search
- The correct molecule must always appear in the candidate list when simulated mass noise is truncated to ±10 ppm

## Limitations

- Candidates whose masses lie within the 10 ppm window but whose true RTs differ can still outrank the correct molecule; RT ranking only reaches ~70% top-3 accuracy
- Only molecules with a KEGG number in HMDB (6,823) are covered; metabolites absent from the database cannot be annotated
- The mass-noise simulation is a simplification; real instrument mass errors may not follow the truncated Gaussian model

## Evidence

- [methods] 10 ppm tolerance and truncated noise guaranteeing correct molecule returned: [evidence span withheld — anchor status: not_found]
- [abstract] correct molecule among top three candidates in 68% of cases: "our results show that in 68% of the cases the correct molecule was among the top three candidates filtered by mass and ranked according to z-scores"
- [intro] m/z ratios as first filter: "The m/z ratios are used as a first filter to obtain candidate annotations from the database."
- [methods] query restricted to more than three candidates: [evidence span withheld — anchor status: not_found]
- [readme] README confirms cmmrt implements the DNN + projection approach and the branch to use: "Note that, to integrate the proposal into the CEU Mass Mediator platform, the code in this repository will continue to be developed. Hence, branch `paper` should be used as reference for reproducing"
