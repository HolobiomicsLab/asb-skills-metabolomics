---
name: retention-time-alignment
description: Use when two LC-MS feature lists — an in vitro prototype list and an
  in vivo metabolite list, each with retention time and accurate m/z per component
  — must be bridged by RT and m/z matching.
when_to_use_negative:
- When only MS2 spectral-similarity matching against a prototype database is needed
  without in vitro/in vivo bridging — the article notes that technique 'only enables
  matching between single metabolite and the prototype database, and only utilizes
  similarity of the secondary mass spectrometry maps'.
- When the input is a single feature table already aligned across samples rather than
  two separate lists (in vitro prototypes vs. in vivo metabolites) requiring cross-list
  matching.
- When no retention-time information is available and only accurate-mass defect or
  MS2 fragment matching can be used.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging data post-processing technique whose
    first step implements the RT ±25 s / m/z ±5 ppm prototype and isomer matching
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation environment in which the prototype and isomer matching over
    RT and accurate m/z is executed
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:14573e7d3214c282ebbfd5a13f326406966c52ecd75a9e9f968d266b4298897d
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# retention-time-alignment

## Summary

An RT/m/z window-based matching step in the MDRB pipeline (R) that bridges in vitro prototype components to in vivo mouse metabolite features using a retention-time tolerance of ±25 s and an accurate-mass tolerance of ±5 ppm, flagging RT-shifted m/z-matched components as isomers.

## When to use

Use when two LC-MS feature lists — an in vitro prototype list and an in vivo metabolite list, each with retention time and accurate m/z per component — must be bridged by RT and m/z matching.

## When NOT to use

- When only MS2 spectral-similarity matching against a prototype database is needed without in vitro/in vivo bridging — the article notes that technique 'only enables matching between single metabolite and the prototype database, and only utilizes similarity of the secondary mass spectrometry maps'.
- When the input is a single feature table already aligned across samples rather than two separate lists (in vitro prototypes vs. in vivo metabolites) requiring cross-list matching.
- When no retention-time information is available and only accurate-mass defect or MS2 fragment matching can be used.

## Inputs

- In vitro prototype component list (retention time and accurate m/z per component)
- Mouse (in vivo) metabolite feature list (retention time and accurate m/z per component)

## Outputs

- prototype_isomer_matches.csv — one row per mouse metabolite–prototype assignment, annotated with prototype identity or isomer flag, m/z error, and RT difference

## How to apply

Load the in vitro prototype list and the mouse (in vivo) metabolite feature list, each containing retention time and accurate m/z per component, into R. For every mouse metabolite–prototype pair, screen on accurate mass, retaining pairs whose m/z difference is within ±5 ppm. Among the m/z-matched pairs, classify pairs whose retention-time difference is within ±25 s as prototype matches (same component observed in vivo) and pairs whose RT falls outside ±25 s as candidate isomers (same nominal mass but chromatographically distinct). Annotate each mouse metabolite with its matched prototype identity or isomer flag and record the m/z error and RT difference for every assignment. Write the annotated results to prototype_isomer_matches.csv with one row per mouse metabolite–prototype assignment. This is the first step of the MDRB pipeline, which is later followed by mass-defect and MS2-fragment-similarity scoring to bridge prototypes and metabolites.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging data post-processing technique whose first step implements the RT ±25 s / m/z ±5 ppm prototype and isomer matching) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation environment in which the prototype and isomer matching over RT and accurate m/z is executed) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Every assignment carries an m/z error within the ±5 ppm window.
- Prototype matches have |ΔRT| ≤ 25 s.
- Isomer calls have |ΔRT| > 25 s.
- Output file prototype_isomer_matches.csv exists in valid CSV format with annotation columns for prototype identity, isomer flag, m/z error, and RT difference.
- Every mouse metabolite–prototype assignment records both m/z error and RT difference.

## Limitations

- No quantitative accuracy metric is reported for this matching step in the paper, so validation relies on tolerance-window invariants rather than performance benchmarks.
- No software versions are stated for the MDRB technique or any data-processing platform.
- No specific mass spectrometry instrument model or data acquisition parameters are named in the article.
- Identification of in vivo components remains difficult despite MS data post-processing techniques, as acknowledged by the authors.

## Evidence

- [abstract] MDRB step 1 implementation context: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [intro] Stepwise identification of exposed components: "the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB"
- [discussion] Purpose of MDRB technique: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
