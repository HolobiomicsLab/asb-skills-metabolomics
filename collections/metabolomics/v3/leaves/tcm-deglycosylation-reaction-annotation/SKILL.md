---
name: tcm-deglycosylation-reaction-annotation
description: Use when bridging TCM prototype components to their predictable metabolites
  in MS data, and a reaction catalog including deglycosylation plus scored m/z and
  MS2-similarity matching is needed.
when_to_use_negative:
- The target metabolite cannot be rationalized as a one- or two-step product of a
  prototype via Metworks 1.3.0.200 reactions or deglycosylation (i.e., unpredictable
  metabolites requiring background subtraction approaches like PATBS instead)
- Only single-metabolite-to-prototype-database matching via secondary MS spectral
  similarity is needed (the MTSF-style scenario), rather than scored multi-dimension
  bridging
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metworks 1.3.0.200
  role: Source of the known predictable metabolic reaction definitions used as the
    base reaction catalog
- name: MDRB
  role: R-based mass dataset relevance bridging platform in which the predictable-metabolite
    matching filter and scoring are implemented
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation language for the MDRB data post-processing and scoring workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:028c1babab014c6acb42b8c3c65396295e20279ed0e38e2533bdd3d5818ed318
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tcm-deglycosylation-reaction-annotation

## Summary

Reconstructs the MDRB predictable-metabolite reaction catalog by combining known metabolic reactions from Metworks 1.3.0.200 with TCM-specific deglycosylation reactions, then scores one- and two-step metabolite-prototype assignments (m/z within ±5 ppm = 70 points; MS2 similarity degree 1/≥2/0 = 20/30/0 points).

## When to use

Use when bridging TCM prototype components to their predictable metabolites in MS data, and a reaction catalog including deglycosylation plus scored m/z and MS2-similarity matching is needed.

## When NOT to use

- The target metabolite cannot be rationalized as a one- or two-step product of a prototype via Metworks 1.3.0.200 reactions or deglycosylation (i.e., unpredictable metabolites requiring background subtraction approaches like PATBS instead)
- Only single-metabolite-to-prototype-database matching via secondary MS spectral similarity is needed (the MTSF-style scenario), rather than scored multi-dimension bridging

## Inputs

- Known predictable metabolic reaction definitions from Metworks 1.3.0.200
- TCM-specific deglycosylation reaction definitions
- Prototype component list (e.g., in vitro PATBS-characterized CKXR components) with accurate masses and MS2 spectra
- Candidate in vivo metabolite feature list with m/z values and MS2 spectra

## Outputs

- predictable_reaction_catalog_scores.csv (scored reaction catalog with Metworks 1.3.0.200-derived plus TCM deglycosylation reactions)
- Scored metabolite-prototype assignment table (70-point m/z and 20/30/0-point MS2 similarity components per pair)

## How to apply

In R, load the known predictable metabolic reaction definitions supplied by Metworks 1.3.0.200 as the base rule set, then append TCM-specific deglycosylation reactions and enumerate one-step and two-step transformation products to form the complete predictable reaction catalog. For each candidate metabolite-prototype pair, award 70 points when the m/z matches within a ±5 ppm tolerance. Score MS2 fragment similarity by match degree: 20 points for degree 1, 30 points for degree ≥2, and 0 points when no match is found. Combine the m/z and MS2 similarity components into the overall assignment score for each bridged pair, consistent with MDRB's integration of m/z, MS1/MS2 mass defect, and MS2 fragment similarity. Write the scored catalog and assignments to predictable_reaction_catalog_scores.csv for downstream component-relationship-network construction.

## Related tools

- **Metworks 1.3.0.200** (Source of the known predictable metabolic reaction definitions used as the base reaction catalog)
- **MDRB** (R-based mass dataset relevance bridging platform in which the predictable-metabolite matching filter and scoring are implemented) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation language for the MDRB data post-processing and scoring workflow)

## Evaluation signals

- The reaction catalog contains both Metworks 1.3.0.200-derived reactions and the appended TCM deglycosylation reactions, verified by inspection of predictable_reaction_catalog_scores.csv
- Every metabolite-prototype assignment with an m/z match inside ±5 ppm carries the 70-point m/z component score
- MS2 similarity scores are exactly 20 (match degree 1), 30 (match degree ≥2), or 0 (no match) in the assignments table
- Enumerated transformation products span both one-step and two-step reactions from each prototype
- Bridged pairs feed a component relationship network linking in vivo metabolites back to in vitro prototypes sharing the same parent nucleus structure

## Limitations

- Only one- or two-step reactions from the Metworks 1.3.0.200 catalog plus TCM deglycosylation are enumerated; multi-step or unpredictable transformations fall outside this filter
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the scoring strategy in the source article
- Identification of in vivo components remains difficult despite MS data post-processing, per the authors' acknowledged limitation

## Evidence

- [intro] Metworks reaction basis: "We based the known predictable metabolic responses provided by Metworks 1.3.0.200."
- [abstract] Scoring rationale: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [intro] MS2 fragment mass-defect bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [abstract] MDRB platform availability: "the developed MDRB platform is publicly available"
