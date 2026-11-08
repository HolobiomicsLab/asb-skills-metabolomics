---
name: accurate-mass-tolerance-filtering
description: Use when correlating in vivo LC-MS metabolite features with in vitro
  prototype components by accurate mass and retention time, e.g., MDRB step 1 prototype/isomer
  classification in TCM disposition studies.
when_to_use_negative:
- When the task is downstream MS/MS-based scoring (MS2 fragment similarity, MS1/MS2
  mass defect) of metabolite–prototype pairs — this skill covers only the step-1 accurate-mass/RT
  screen.
- When in vitro and in vivo data were acquired under non-comparable chromatographic
  methods, since the ±25 s RT window that separates prototype matches from isomers
  would be meaningless.
- When mouse metabolite features are already annotated with prototype identities or
  isomer flags — re-filtering adds no information.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Implementation language in which the MDRB step-1 m/z and RT matching against
    the prototype list is scripted
  repo: https://github.com/933ZhangDD/MDRB
- name: MDRB
  role: Data post-processing technique whose first step performs the ±5 ppm mass and
    ±25 s RT prototype/isomer filtering
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:6482f1d0c7f735793fe2e62b1760c5365ce04af9d9e4949e0b9253ed7d458b2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# accurate-mass-tolerance-filtering

## Summary

MDRB step-1 filtering that matches mouse (in vivo) metabolite features against the in vitro prototype list using accurate m/z within ±5 ppm, then splits matches by retention time (±25 s) into prototype components versus chromatographically distinct isomers.

## When to use

Use when correlating in vivo LC-MS metabolite features with in vitro prototype components by accurate mass and retention time, e.g., MDRB step 1 prototype/isomer classification in TCM disposition studies.

## When NOT to use

- When the task is downstream MS/MS-based scoring (MS2 fragment similarity, MS1/MS2 mass defect) of metabolite–prototype pairs — this skill covers only the step-1 accurate-mass/RT screen.
- When in vitro and in vivo data were acquired under non-comparable chromatographic methods, since the ±25 s RT window that separates prototype matches from isomers would be meaningless.
- When mouse metabolite features are already annotated with prototype identities or isomer flags — re-filtering adds no information.

## Inputs

- In vitro prototype component list (retention time and accurate m/z per component)
- Mouse (in vivo) metabolite feature list (retention time and accurate m/z per component)

## Outputs

- prototype_isomer_matches.csv — one row per mouse metabolite–prototype assignment, with prototype identity or isomer flag, m/z error, and RT difference

## How to apply

Load the in vitro prototype list and the mouse in vivo metabolite feature list — each containing retention time and accurate m/z per component — into R. For every mouse metabolite–prototype pair, screen on accurate mass, retaining pairs whose m/z difference is within ±5 ppm. Among the m/z-matched pairs, classify pairs whose retention-time difference is within ±25 s as prototype matches (the same component observed in vivo) and pairs whose RT falls outside ±25 s as candidate isomers (same nominal mass but chromatographically distinct). Annotate each mouse metabolite with its matched prototype identity or isomer flag and record the m/z error and RT difference for every assignment before writing the results to prototype_isomer_matches.csv. This filtering is the first step of the MDRB pipeline, which subsequently scores metabolite–prototype pairs across m/z, MS1/MS2 mass defect, and MS2 fragment similarity dimensions.

## Related tools

- **R** (Implementation language in which the MDRB step-1 m/z and RT matching against the prototype list is scripted) — https://github.com/933ZhangDD/MDRB
- **MDRB** (Data post-processing technique whose first step performs the ±5 ppm mass and ±25 s RT prototype/isomer filtering) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Every written assignment carries an m/z error within the ±5 ppm window.
- Prototype-match rows all have |ΔRT| ≤ 25 s; isomer rows all have |ΔRT| > 25 s.
- prototype_isomer_matches.csv exists and parses as valid CSV with the required annotation columns (prototype identity / isomer flag, m/z error, RT difference).
- Output contains exactly one row per mouse metabolite–prototype assignment.
- No quantitative accuracy metric exists for this step in the paper, so acceptance is by these structural invariants rather than a performance score.

## Limitations

- No quantitative accuracy metric, replicate count, or statistical validation is reported for this step in the paper — correctness is judged only by the ±5 ppm / ±25 s invariants.
- No software version is stated for the MDRB technique or the data-processing platform.
- The isomer rule is purely RT-exclusion-based: it flags same-m/z components outside the ±25 s RT window but does not further separate multiple co-eluting isomers.

## Evidence

- [other] MDRB tolerance thresholds: [evidence span withheld — anchor status: not_found]
- [intro] MDRB prototype-metabolite bridging: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [abstract] Three sequential filtering dimensions: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
