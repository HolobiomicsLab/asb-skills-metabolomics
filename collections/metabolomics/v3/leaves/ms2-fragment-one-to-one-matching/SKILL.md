---
name: ms2-fragment-one-to-one-matching
description: Use when bridging prototype compounds to candidate metabolites from PATBS-processed
  LC-MS feature tables where MS1 mass-defect screening (±100 amu / ±25 mmu) is insufficient
  and MS2 fragment-level corroboration is needed.
when_to_use_negative:
- When the metabolite–prototype relationship is already established through predictable
  metabolic responses (e.g., Metworks 1.3.0.200 rules) rather than requiring mass-defect-based
  bridging
- When only secondary mass-spectrometry map similarity is available without mass-defect
  information, as in MTSF-style single-metabolite-to-database matching
- When candidate components fall outside the ±100 amu MS1 window of the prototype,
  since MS2 matching in MDRB is applied downstream of the MS1 screen
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging platform that performs the one-to-one
    MS2 fragment matching and scoring between prototypes and metabolites
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Language in which the MDRB post-processing and fragment-matching screening
    is implemented and run
- name: PATBS
  role: Upstream precise-and-thorough background-substraction technique producing
    the feature table (m/z, MS1, MS2, retention) that feeds the fragment matching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:9b55ff4a3077c38273c64f6491d52b960522eb82ad623f9c1c12932869cd598e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms2-fragment-one-to-one-matching

## Summary

One-to-one MS2 fragment matching between prototype and candidate metabolite in the MDRB (R) pipeline, scoring 5 points per fragment matched within a ±50 amu / ±25 mmu window, as one of three dimensions bridging TCM prototypes to their in vivo metabolites.

## When to use

Use when bridging prototype compounds to candidate metabolites from PATBS-processed LC-MS feature tables where MS1 mass-defect screening (±100 amu / ±25 mmu) is insufficient and MS2 fragment-level corroboration is needed.

## When NOT to use

- When the metabolite–prototype relationship is already established through predictable metabolic responses (e.g., Metworks 1.3.0.200 rules) rather than requiring mass-defect-based bridging
- When only secondary mass-spectrometry map similarity is available without mass-defect information, as in MTSF-style single-metabolite-to-database matching
- When candidate components fall outside the ±100 amu MS1 window of the prototype, since MS2 matching in MDRB is applied downstream of the MS1 screen

## Inputs

- PATBS-processed feature table with m/z, MS1, MS2, and retention information for prototypes and candidate metabolites (CSV, read in R)
- Prototype–metabolite pairs already passing the MS1 mass-defect screen (±100 amu / ±25 mmu, score 50)

## Outputs

- Per-pair MS2 fragment-matching score (5 points per matched fragment within ±50 amu / ±25 mmu)
- Aggregated scored screening results written to mass_defect_screening_results.csv

## How to apply

Load the PATBS-processed feature table (m/z, MS1, MS2, retention information for prototypes and candidate metabolites) into R via the MDRB platform. After the MS1 mass-defect screen (pairs within ±100 amu and ±25 mmu, score 50), perform one-to-one MS2 fragment matching between each prototype and its candidate metabolite, retaining fragments whose masses agree within ±50 amu and ±25 mmu mass defect, and award 5 points per matched fragment. This exploits the paper's observation that the mass defect of MS2 fragments of prototypes and metabolites stays within a limited range, providing information to bridge prototypes and metabolites. Aggregate this fragment score with the MS1 score (50) and the retention-behavior score (50), then apply the N-count filter (atom-count change ≤ 3) before writing scored pairs to mass_defect_screening_results.csv.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging platform that performs the one-to-one MS2 fragment matching and scoring between prototypes and metabolites) — https://github.com/933ZhangDD/MDRB
- **R** (Language in which the MDRB post-processing and fragment-matching screening is implemented and run)
- **PATBS** (Upstream precise-and-thorough background-substraction technique producing the feature table (m/z, MS1, MS2, retention) that feeds the fragment matching)

## Evaluation signals

- Every retained prototype–metabolite pair satisfies the ±50 amu / ±25 mmu MS2 fragment window in addition to the ±100 amu / ±25 mmu MS1 window
- Each matched MS2 fragment contributes exactly 5 points to the pair score, on top of the 50-point MS1 and 50-point retention scores
- Pairs failing the N-count filter (atom-count change > 3 between prototype and metabolite) are absent from the output
- Output table mass_defect_screening_results.csv contains only threshold-compliant pairs (qualitative check; no quantitative acceptance metric is reported in the paper)

## Limitations

- No quantitative acceptance metric or statistical validation is reported for the screening; only qualitative threshold compliance can be verified
- No software version is stated for the MDRB technique or the data-processing platform, complicating reproducibility
- Identification of in vivo components remains difficult despite MS data post-processing techniques
- The MDRB repository has no changelog

## Evidence

- [abstract] The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [intro] the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites.: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [abstract] We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes.: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [intro] we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB) data post-processing: "we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB) data post-processing"
