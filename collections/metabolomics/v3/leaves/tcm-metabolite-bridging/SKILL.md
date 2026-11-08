---
name: tcm-metabolite-bridging
description: Use when you have a PATBS-processed feature table (m/z, MS1, MS2, retention)
  for prototypes and candidate metabolites and need to score/bridge metabolite–prototype
  relationships using MS1/MS2 mass-defect windows (±25 mmu) and fragment matching.
when_to_use_negative:
- The input is raw vendor LC-MS data rather than a PATBS-processed feature table with
  m/z, MS1, MS2, and retention columns
- Only single-metabolite-to-prototype MS2 map similarity matching is needed (MTSF-style
  matching suffices) rather than multi-dimensional mass-defect bridging
- Metabolites follow predictable, known metabolic responses that are already handled
  by tools such as Metworks 1.3.0.200
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging platform that executes the mass-defect
    screening, fragment matching, scoring, and N-count filter, writing mass_defect_screening_results.csv
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation language/runtime in which the MDRB post-processing pipeline
    is executed
- name: PATBS
  role: Upstream precise-and-thorough background-substraction technique that produces
    the non-targeted feature table consumed by MDRB
- name: Metworks 1.3.0.200
  role: Source of known predictable metabolic responses used as the complement to
    MDRB's unpredictable-metabolite screening
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:ba700c3ce996c477488573dd182903599dbf5ccd953eddc357d6643e21cf61f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tcm-metabolite-bridging

## Summary

An R-based MDRB (mass dataset relevance bridging) post-processing skill that scores metabolite–prototype pairs from PATBS-processed LC-MS feature tables by integrating m/z, MS1/MS2 mass-defect windows, and MS2 fragment similarity, enabling bridging of in vivo TCM metabolites to in vitro prototypes and construction of a component relationship network.

## When to use

Use when you have a PATBS-processed feature table (m/z, MS1, MS2, retention) for prototypes and candidate metabolites and need to score/bridge metabolite–prototype relationships using MS1/MS2 mass-defect windows (±25 mmu) and fragment matching.

## When NOT to use

- The input is raw vendor LC-MS data rather than a PATBS-processed feature table with m/z, MS1, MS2, and retention columns
- Only single-metabolite-to-prototype MS2 map similarity matching is needed (MTSF-style matching suffices) rather than multi-dimensional mass-defect bridging
- Metabolites follow predictable, known metabolic responses that are already handled by tools such as Metworks 1.3.0.200

## Inputs

- PATBS-processed feature table with m/z, MS1, MS2, and retention information for prototypes and candidate metabolites
- Prototype component database (e.g., in vitro components of the TCM under study)
- Atom-count (elemental composition) information for prototypes and candidate metabolites, needed for the N-count filter

## Outputs

- mass_defect_screening_results.csv — scored metabolite–prototype pairs with aggregated MS1, MS2-fragment, retention, and N-count dimension scores
- Component relationship network linking in vivo metabolites to in vitro prototypes

## How to apply

Load the PATBS-processed feature table containing m/z, MS1, MS2, and retention information for prototypes and candidate metabolites in R using the MDRB platform (933ZhangDD/MDRB). For unpredictable metabolites, apply the MS1 mass-defect screen: keep pairs with mass difference within ±100 amu and mass defect within ±25 mmu, awarding a base score of 50; then perform one-to-one MS2 fragment matching between prototype and metabolite within ±50 amu and ±25 mmu, awarding 5 points per matched fragment. Assign a retention score of 50 to pairs consistent in retention behavior, and apply the N-count filter, discarding pairs whose atom-count (N) change between prototype and metabolite exceeds 3. Aggregate the dimension scores, retain only threshold-compliant pairs, and write the results to mass_defect_screening_results.csv. The rationale is that metabolites sharing a prototype's parent nucleus exhibit minimal mass-defect variation in both MS1 and MS2, allowing metabolites to be bridged to prototypes without predicting structures as required by Metworks-based predictable-response approaches.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging platform that executes the mass-defect screening, fragment matching, scoring, and N-count filter, writing mass_defect_screening_results.csv) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation language/runtime in which the MDRB post-processing pipeline is executed)
- **PATBS** (Upstream precise-and-thorough background-substraction technique that produces the non-targeted feature table consumed by MDRB)
- **Metworks 1.3.0.200** (Source of known predictable metabolic responses used as the complement to MDRB's unpredictable-metabolite screening)

## Evaluation signals

- Every retained metabolite–prototype pair in mass_defect_screening_results.csv satisfies the ±100 amu / ±25 mmu MS1 mass-defect window
- Every retained pair satisfies the ±50 amu / ±25 mmu one-to-one MS2 fragment matching window
- No retained pair has an atom-count (N) change between prototype and metabolite exceeding 3
- Score composition is correct: base score of 50 for qualifying MS1 pairs, 5 points per matched MS2 fragment within ±50 amu / ±25 mmu, and 50 for retention-consistent pairs
- Discarded pairs are exactly those violating one of the window or N-count thresholds, so only threshold-compliant pairs appear in the output table

## Limitations

- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy; validation is limited to a qualitative threshold-compliance check
- No software versions are stated for the MDRB technique or any data-processing platform, and no changelog is available for the source repository
- No specific mass spectrometry instrument model or data acquisition parameters are named
- Identification of in vivo components remains difficult despite MS data post-processing techniques

## Evidence

- [abstract] Scoring approach integrating m/z, MS1/MS2 mass defect, and MS2 fragment similarity: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [abstract] MDRB is an R-based post-processing technique: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [intro] MS2 fragment mass-defect bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [discussion] MDRB bridges in vivo metabolite characterization with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
