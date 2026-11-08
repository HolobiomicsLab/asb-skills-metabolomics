---
name: mass-defect-based-metabolite-bridging
description: Use when you have LC-MS feature lists for in vivo metabolites plus a
  prototype (in vitro parent component) list from PATBS non-targeted identification,
  and need to bridge metabolites to prototypes and characterize metabolites across
  biological matrices.
when_to_use_negative:
- No in vitro prototype list is available, since the bridging approach fundamentally
  requires prototype-metabolite pairs to score mass defect variations
- You need only single-metabolite-to-prototype-database matching based on secondary
  mass spectrometry map similarity — the MTSF-style approach the article critiques
  as limited, distinct from MDRB's multi-dimensional scoring
- Compounds without a shared parent nucleus structure, where the mass-defect-based
  bridging rationale (metabolite MS2 fragment mass defects within a certain range
  of the prototype's) does not hold
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0092
tools:
- name: MDRB
  role: Core R-based data post-processing platform that sequentially analyzes m/z,
    MS1/MS2 mass defect, and MS2 fragment similarity to score and bridge metabolite-prototype
    pairs
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Precise-and-thorough background-subtraction technique that supplies the non-targeted
    identification feature lists of prototypes and metabolites that MDRB consumes
- name: R
  role: Implementation language of the MDRB post-processing pipeline
- name: Metworks 1.3.0.200
  role: Source of the known predictable metabolic responses used in the predictable-pathway-matching
    step of MDRB characterization
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:20c5814cedb314fd0662b223a0c97822f17122b815cbb4fe072c29218df9044b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - Metworks 1.3.0.200
  - PATBS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 31
---

# mass-defect-based-metabolite-bridging

## Summary

A mass dataset relevance bridging (MDRB) data post-processing technique that characterizes in vivo TCM metabolites by bridging them to in vitro prototypes using a scoring approach integrating m/z, MS1/MS2 mass defect, and MS2 fragment similarity. It replaces manual characterization (110 metabolites, 31%) with an automated pipeline that raised CKXR metabolite characterization to 227 metabolites (65%).

## When to use

Use when you have LC-MS feature lists for in vivo metabolites plus a prototype (in vitro parent component) list from PATBS non-targeted identification, and need to bridge metabolites to prototypes and characterize metabolites across biological matrices.

## When NOT to use

- No in vitro prototype list is available, since the bridging approach fundamentally requires prototype-metabolite pairs to score mass defect variations
- You need only single-metabolite-to-prototype-database matching based on secondary mass spectrometry map similarity — the MTSF-style approach the article critiques as limited, distinct from MDRB's multi-dimensional scoring
- Compounds without a shared parent nucleus structure, where the mass-defect-based bridging rationale (metabolite MS2 fragment mass defects within a certain range of the prototype's) does not hold

## Inputs

- PATBS-processed feature table with m/z, MS1, MS2, and retention information for prototypes (in vitro components) and candidate in vivo metabolites
- CKXR prototype (in vitro component) list from the PATBS non-targeted identification step
- In vivo metabolite feature lists per biological matrix (plasma, urine, feces, lung) with MS2 fragment spectra
- Known predictable metabolic responses from Metworks 1.3.0.200

## Outputs

- Scored metabolite-prototype pair table (mass_defect_screening_results.csv) with aggregated dimension scores
- Component relationship network linking in vivo metabolites to in vitro prototypes across matrices
- Characterized metabolite counts per matrix (66 plasma, 222 urine, 30 feces, 40 lung) and overall characterization rate (227 metabolites, 65%)

## How to apply

Load the PATBS-processed feature table containing m/z, MS1, MS2, and retention information for prototypes and candidate metabolites into the R-based MDRB platform. Sequentially analyze three dimensions of mass spectral data — m/z, mass defect of MS1 and MS2, and similarity of MS2 fragments — scoring metabolite-prototype pairs on structural similarity and minimal mass defect variations. For the unpredictable-metabolite step, keep pairs whose MS1 mass difference is within ±100 amu and mass defect within ±25 mmu (score 50), perform one-to-one MS2 fragment matching within ±50 amu / ±25 mmu (5 points per matched fragment), assign a retention score of 50 for retention-consistent pairs, and discard pairs whose atom-count (N) change exceeds 3; write qualifying pairs to mass_defect_screening_results.csv. For predictable metabolic pathways, base known metabolic responses on Metworks 1.3.0.200. Finally, construct the component relationship network linking in vivo metabolites to in vitro prototypes across plasma, urine, feces, and lung matrices, and compute the overall characterization rate.

## Related tools

- **MDRB** (Core R-based data post-processing platform that sequentially analyzes m/z, MS1/MS2 mass defect, and MS2 fragment similarity to score and bridge metabolite-prototype pairs) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Precise-and-thorough background-subtraction technique that supplies the non-targeted identification feature lists of prototypes and metabolites that MDRB consumes)
- **R** (Implementation language of the MDRB post-processing pipeline)
- **Metworks 1.3.0.200** (Source of the known predictable metabolic responses used in the predictable-pathway-matching step of MDRB characterization)

## Evaluation signals

- Total characterized metabolite count and rate match the reported 227 (65%) versus the manual baseline of 110 metabolites (31%)
- Per-matrix characterized counts match 66 plasma, 222 urine, 30 feces, 40 lung
- Every retained metabolite-prototype pair satisfies the ±100 amu / ±25 mmu MS1 window, the ±50 amu / ±25 mmu MS2 fragment window, and the N-count change ≤3 rule
- The scored screening results table (mass_defect_screening_results.csv) is written with aggregated dimension scores for qualifying pairs
- The constructed component relationship network connects in vivo metabolites to in vitro prototypes across all four matrices (plasma, urine, feces, lung)

## Limitations

- No software versions are stated for the MDRB technique or its data-processing platform, complicating reproducibility
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy beyond the characterized-metabolite counts
- No changelog is available for the MDRB source repository
- Identification of in vivo components remains difficult despite MS data post-processing techniques
- No specific mass spectrometry instrument model or data acquisition parameters are named

## Evidence

- [abstract] abstract: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] intro: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [discussion] discussion: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
