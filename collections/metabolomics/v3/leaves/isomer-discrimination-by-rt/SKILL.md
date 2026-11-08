---
name: isomer-discrimination-by-rt
description: Use when matching LC-HRMS metabolite features from in vivo samples against
  a prototype list, and same-nominal-mass components must be classified as prototype
  matches vs. chromatographically distinct isomers.
when_to_use_negative:
- The metabolite candidates are already characterized and only downstream MS1/MS2
  mass-defect or MS2-fragment-similarity scoring (later MDRB steps) is needed.
- RT is not comparable between the prototype and metabolite lists (e.g., different
  LC columns, gradients, or instruments), since the ±25 s window assumes co-elution
  of the same component in both runs.
- Data are single-run features with no paired in vitro prototype list to bridge against
  — MTSF-style prototype-database matching or de novo annotation is more appropriate.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Scripting language in which the MDRB step-1 m/z (±5 ppm) and RT (±25 s) matching
    and isomer flagging are implemented
- name: MDRB
  role: Provides the step-1 prototype/isomer characterization logic that bridges in
    vivo metabolites to in vitro prototypes
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:d4e9f8f9a6a312f11223cbac791e2c1a78f045478302e6f488248ba8fc78ac76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# isomer-discrimination-by-rt

## Summary

Discriminates true prototype components from same-mass isomers when bridging in vivo mouse metabolite features to an in vitro prototype list (MDRB step 1): matching uses accurate m/z within ±5 ppm, and retention time within ±25 s separates a prototype match from a candidate isomer.

## When to use

Use when matching LC-HRMS metabolite features from in vivo samples against a prototype list, and same-nominal-mass components must be classified as prototype matches vs. chromatographically distinct isomers.

## When NOT to use

- The metabolite candidates are already characterized and only downstream MS1/MS2 mass-defect or MS2-fragment-similarity scoring (later MDRB steps) is needed.
- RT is not comparable between the prototype and metabolite lists (e.g., different LC columns, gradients, or instruments), since the ±25 s window assumes co-elution of the same component in both runs.
- Data are single-run features with no paired in vitro prototype list to bridge against — MTSF-style prototype-database matching or de novo annotation is more appropriate.

## Inputs

- In vitro prototype component list with accurate m/z and retention time per component
- Mouse (in vivo) metabolite feature list with accurate m/z and retention time per component

## Outputs

- prototype_isomer_matches.csv — one row per mouse metabolite–prototype assignment with prototype identity or isomer flag, m/z error, and RT difference

## How to apply

Load the in vitro prototype list and the mouse (in vivo) metabolite feature list, each with retention time and accurate m/z per component, into R as part of the MDRB pipeline. For every mouse metabolite–prototype pair, first screen on accurate mass, retaining pairs whose m/z difference is within ±5 ppm. Among m/z-matched pairs, classify those with |ΔRT| ≤ 25 s as prototype matches (same component observed in vivo) and those with |ΔRT| > 25 s as candidate isomers (same nominal mass but chromatographically distinct). Annotate each mouse metabolite with its matched prototype identity or isomer flag, and record the m/z error and RT difference for every assignment. The rationale is that identical components co-elute between in vitro and in vivo runs, so a mass match with a large RT shift indicates an isomer rather than the prototype itself. Write the annotated results to prototype_isomer_matches.csv with one row per mouse metabolite–prototype assignment.

## Related tools

- **R** (Scripting language in which the MDRB step-1 m/z (±5 ppm) and RT (±25 s) matching and isomer flagging are implemented)
- **MDRB** (Provides the step-1 prototype/isomer characterization logic that bridges in vivo metabolites to in vitro prototypes) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Every assignment carries an m/z error within the ±5 ppm window
- All prototype matches have |ΔRT| ≤ 25 s
- All isomer calls have |ΔRT| > 25 s with m/z still within ±5 ppm
- prototype_isomer_matches.csv exists in valid CSV format with the required annotation columns (prototype identity/isomer flag, m/z error, RT difference)
- No paper-reported quantitative accuracy metric exists for this step, so acceptance rests on the window invariants and output schema above

## Limitations

- No quantitative accuracy metric is reported for this step in the paper; validation relies on window invariants and output-file checks
- Identification of in vivo components remains difficult despite MS data post-processing techniques
- The method gives no statistical validation, replicate counts, or performance metrics for the bridging strategy
- No software versions are stated for the MDRB technique or the data-processing platform

## Evidence

- [full_text] MDRB step 1 characterizes prototypes in mice using RT within ± 25 s and m/z within ± 5 ppm, while isomers are defined as components with m/z within ± 5 ppm error but RT outside ± 25 s.: [evidence span withheld — anchor status: not_found]
- [intro] Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [discussion] we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
