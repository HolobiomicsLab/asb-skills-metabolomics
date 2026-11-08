---
name: metabolite-prototype-matching
description: Use when an in vitro prototype feature list (RT + accurate m/z) must
  be matched against an in vivo metabolite feature list to call prototype components
  vs. same-mass RT-shifted isomers.
when_to_use_negative:
- No in vitro prototype list exists, so metabolites must be identified de novo rather
  than bridged to prototypes
- The goal is MS2 fragment-similarity matching of a single metabolite against a prototype
  database (the MTSF approach), not accurate-mass/RT bridging
- In vitro and in vivo runs were not chromatographically aligned, making a ±25 s RT
  window meaningless for prototype vs. isomer discrimination
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Implementation language in which the MDRB prototype/isomer screening (±5 ppm
    m/z, ±25 s RT) is executed on the feature lists
  repo: https://github.com/933ZhangDD/MDRB
- name: MDRB
  role: R-based mass dataset relevance bridging data post-processing technique whose
    step 1 performs prototype and isomer characterization
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:156a0b840e5ff9293ec8091a4791a131c47502ef0376add9df2dc8486568c665
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
yes_rate: 1.0
n_claims: 7
---

# metabolite-prototype-matching

## Summary

MDRB step-1 matching that characterizes in vivo prototype components and their isomers in a mouse metabolite list by screening every mouse metabolite–prototype pair on accurate m/z (±5 ppm) and retention-time difference (±25 s), bridging in vitro and in vivo mass spectral datasets.

## When to use

Use when an in vitro prototype feature list (RT + accurate m/z) must be matched against an in vivo metabolite feature list to call prototype components vs. same-mass RT-shifted isomers.

## When NOT to use

- No in vitro prototype list exists, so metabolites must be identified de novo rather than bridged to prototypes
- The goal is MS2 fragment-similarity matching of a single metabolite against a prototype database (the MTSF approach), not accurate-mass/RT bridging
- In vitro and in vivo runs were not chromatographically aligned, making a ±25 s RT window meaningless for prototype vs. isomer discrimination

## Inputs

- In vitro prototype list with retention time and accurate m/z per component
- Mouse (in vivo) metabolite feature list with retention time and accurate m/z per component
- Matching tolerances: m/z ±5 ppm, RT ±25 s

## Outputs

- prototype_isomer_matches.csv — one row per mouse metabolite–prototype assignment with prototype identity or isomer flag, m/z error, and RT difference

## How to apply

Load the in vitro prototype list and the mouse (in vivo) metabolite feature list, each containing retention time and accurate m/z per component, into R. For every mouse metabolite–prototype pair, first screen on accurate mass, retaining pairs whose m/z difference is within ±5 ppm. Among the m/z-matched pairs, classify those with retention-time difference within ±25 s as prototype matches (same component observed in vivo) and those with RT outside ±25 s as candidate isomers (same nominal mass but chromatographically distinct). Annotate each mouse metabolite with its matched prototype identity or isomer flag, recording the m/z error and RT difference for every assignment. The rationale is to bridge in vitro prototypes to in vivo metabolites that share the same parent nucleus structure, feeding the downstream MDRB component relationship network.

## Related tools

- **R** (Implementation language in which the MDRB prototype/isomer screening (±5 ppm m/z, ±25 s RT) is executed on the feature lists) — https://github.com/933ZhangDD/MDRB
- **MDRB** (R-based mass dataset relevance bridging data post-processing technique whose step 1 performs prototype and isomer characterization) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Every assignment carries an m/z error within the ±5 ppm window
- Prototype matches all have |ΔRT| ≤ 25 s
- Isomer calls all have |ΔRT| > 25 s while satisfying the ±5 ppm m/z criterion
- prototype_isomer_matches.csv exists and is valid CSV with the required annotation columns (prototype identity/isomer flag, m/z error, RT difference)
- Every mouse metabolite–prototype pair was screened, so no pair is missing an m/z-error record

## Limitations

- No quantitative accuracy metric is reported for this step in the paper; acceptance rests on tolerance-window invariants and output-schema checks
- No software versions are stated for the MDRB technique or the data-processing platform
- No statistical validation, replicate counts, or performance metrics are reported for the overall strategy

## Evidence

- [results] MDRB step 1 characterizes prototypes in mice using RT within ± 25 s and m/z within ± 5 ppm, while isomers are defined as components with m/z within ± 5 ppm error but RT outside ± 25 s.: [evidence span withheld — anchor status: not_found]
- [intro] Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [discussion] we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
- [abstract] The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency"
