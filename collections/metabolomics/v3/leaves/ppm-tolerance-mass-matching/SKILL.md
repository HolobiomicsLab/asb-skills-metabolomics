---
name: ppm-tolerance-mass-matching
description: Use when candidate metabolites of a TCM prototype must be assigned a
  bridging score via exact-mass matching (±5 ppm) and MS2 fragment similarity, using
  a one-/two-step predictable reaction catalog.
when_to_use_negative:
- The candidate metabolite cannot be reached by one- or two-step Metworks 1.3.0.200
  reactions plus TCM deglycosylation, so it falls outside the predictable reaction
  catalog (unpredictable metabolites need the separate mass-defect/scoring branch
  of MDRB)
- Only MS1 data are available with no MS2 fragment spectra, since the MS2 similarity
  component (20/30/0 points) cannot be scored
- The metabolite list has not yet been produced by the PATBS non-targeted background-subtraction
  pipeline, so no candidate metabolites or fragment matches exist to score
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Implementation environment in which the reaction catalog is loaded, transformations
    enumerated, and m/z / MS2 scores computed
- name: MDRB
  role: Data post-processing platform whose predictable-metabolite matching filter
    applies the ±5 ppm and 20/30/0-point scoring to bridge prototypes and metabolites
  repo: https://github.com/933ZhangDD/MDRB
- name: Metworks 1.3.0.200
  role: Source of the base known predictable metabolic reaction definitions (one-
    and two-step) that seed the catalog before TCM deglycosylation reactions are appended
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:bf070a1e13fc438574256bb163694c435fa6dceca1f5bc384ffb3cf2e4a9d2a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ppm-tolerance-mass-matching

## Summary

Scores predictable TCM metabolite–prototype pairs by combining an m/z match score (70 points when matched within ±5 ppm) with an MS2 fragment-similarity score (20/30/0 points by match degree 1/≥2/0), over a predictable reaction catalog built from Metworks 1.3.0.200 one- and two-step reactions plus TCM deglycosylation. It is the scoring core of the MDRB bridging strategy for relating in vivo metabolites back to in vitro prototypes.

## When to use

Use when candidate metabolites of a TCM prototype must be assigned a bridging score via exact-mass matching (±5 ppm) and MS2 fragment similarity, using a one-/two-step predictable reaction catalog.

## When NOT to use

- The candidate metabolite cannot be reached by one- or two-step Metworks 1.3.0.200 reactions plus TCM deglycosylation, so it falls outside the predictable reaction catalog (unpredictable metabolites need the separate mass-defect/scoring branch of MDRB)
- Only MS1 data are available with no MS2 fragment spectra, since the MS2 similarity component (20/30/0 points) cannot be scored
- The metabolite list has not yet been produced by the PATBS non-targeted background-subtraction pipeline, so no candidate metabolites or fragment matches exist to score

## Inputs

- Known predictable metabolic reaction definitions from Metworks 1.3.0.200 (base rule set)
- TCM-specific deglycosylation reaction definitions
- Candidate metabolite–prototype pairs with measured m/z values (MS1) and MS2 fragment data from PATBS-processed spectra

## Outputs

- predictable_reaction_catalog_scores.csv — scored reaction catalog containing Metworks 1.3.0.200-derived and TCM deglycosylation reactions
- Assignments table carrying per-pair m/z scores (70 or 0) and MS2 similarity scores (20/30/0)

## How to apply

First build the predictable reaction catalog in R by loading the known predictable metabolic reactions from Metworks 1.3.0.200 as the base rule set, appending TCM-specific deglycosylation reactions, and enumerating one-step and two-step transformation products. For each candidate metabolite–prototype pair in the bridged dataset, compute the m/z assignment score, awarding 70 points when the m/z matches within a ±5 ppm tolerance. Separately score MS2 fragment similarity by match degree: 20 points for degree 1, 30 points for degree ≥2, and 0 points when no match is found. Combine the m/z and MS2 similarity components into the overall assignment score for each bridged pair. Write the scored reaction catalog and assignments to predictable_reaction_catalog_scores.csv. The rationale is that the MDRB scoring integrates three dimensions of mass spectral data (m/z, MS1/MS2 mass defect, and MS2 fragment similarity), so the ppm-tolerance match supplies the exact-mass dimension while the MS2 degree scoring supplies the fragment-similarity dimension.

## Related tools

- **R** (Implementation environment in which the reaction catalog is loaded, transformations enumerated, and m/z / MS2 scores computed)
- **MDRB** (Data post-processing platform whose predictable-metabolite matching filter applies the ±5 ppm and 20/30/0-point scoring to bridge prototypes and metabolites) — https://github.com/933ZhangDD/MDRB
- **Metworks 1.3.0.200** (Source of the base known predictable metabolic reaction definitions (one- and two-step) that seed the catalog before TCM deglycosylation reactions are appended)

## Evaluation signals

- The predictable reaction catalog contains both Metworks 1.3.0.200-derived reactions and the added TCM deglycosylation reactions
- Every metabolite–prototype pair with an m/z match within ±5 ppm carries exactly 70 points on the m/z component
- MS2 similarity scores are exactly 20 (match degree 1), 30 (match degree ≥2), or 0 (no match), and none of these levels is misassigned
- The overall assignment scores in predictable_reaction_catalog_scores.csv equal the sum of the m/z and MS2 similarity components for each pair
- Output file predictable_reaction_catalog_scores.csv exists and its assignments table can be verified by inspection

## Limitations

- m/z matching is exact-mass based with a ±5 ppm tolerance; false positives are possible when multiple metabolites share the same nominal mass in the catalog
- The MS2 similarity scoring is coarse (only degrees 0, 1, and ≥2), so pairs with a single fragment match (degree 1, 20 points) score lower than ≥2 matches (30 points) regardless of match quality
- No statistical validation, replicate counts, or quantitative performance metrics were reported for the strategy, and no specific mass spectrometry instrument model or acquisition parameters are stated
- Identification of in vivo components remains difficult despite MS data post-processing techniques, a limitation acknowledged by the authors

## Evidence

- [abstract] Scoring approach based on structural similarity and mass defect: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [abstract] Three dimensions of mass spectral data: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [intro] Metworks as reaction catalog basis: "We based the known predictable metabolic responses provided by Metworks 1.3.0.200."
- [intro] MDRB bridging of prototypes and metabolites: "we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB) data post-processing"
