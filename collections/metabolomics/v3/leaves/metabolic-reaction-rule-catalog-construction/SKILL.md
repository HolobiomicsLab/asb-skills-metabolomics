---
name: metabolic-reaction-rule-catalog-construction
description: Use when constructing the predictable-metabolite matching rule set for
  a mass spectrometry bridging analysis — i.e., you need to enumerate metabolic transformation
  products of prototypes and score m/z and MS2 similarity for each candidate metabolite-prototype
  pair.
when_to_use_negative:
- The analysis target is non-targeted background subtraction of unassigned features
  rather than bridging metabolites to prototypes via reaction rules (PATBS handles
  that step)
- Only single metabolite-to-prototype MS2 spectral matching is needed and no reaction-catalog
  or mass-defect-based scoring is required (this was the limitation of the prior MTSF
  approach)
- The reaction space of interest is beyond two transformation steps from the prototype
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metworks 1.3.0.200
  role: Provides the base set of known predictable metabolic reaction definitions
    used to enumerate one- and two-step transformation products
- name: MDRB
  role: R-based mass dataset relevance bridging platform in which the reaction catalog
    is consumed to score and bridge metabolite-prototype pairs
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation language for loading the reaction rules, appending TCM deglycosylation
    reactions, enumerating products, and writing the scored assignment table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:bc8aca827d90112c55d2dc5939ec28b4f9494a7c7fcac89a0279a46460361908
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks 1.3.0.200
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolic-reaction-rule-catalog-construction

## Summary

Builds a predictable metabolite reaction catalog by combining the known one- and two-step metabolic reactions from Metworks 1.3.0.200 with TCM-specific deglycosylation reactions, then scores candidate metabolite-prototype pairs by m/z match (±5 ppm → 70 points) and MS2 fragment similarity degree (1 → 20, ≥2 → 30, 0 → 0 points) within the MDRB post-processing workflow.

## When to use

Use when constructing the predictable-metabolite matching rule set for a mass spectrometry bridging analysis — i.e., you need to enumerate metabolic transformation products of prototypes and score m/z and MS2 similarity for each candidate metabolite-prototype pair.

## When NOT to use

- The analysis target is non-targeted background subtraction of unassigned features rather than bridging metabolites to prototypes via reaction rules (PATBS handles that step)
- Only single metabolite-to-prototype MS2 spectral matching is needed and no reaction-catalog or mass-defect-based scoring is required (this was the limitation of the prior MTSF approach)
- The reaction space of interest is beyond two transformation steps from the prototype

## Inputs

- Known predictable metabolic reaction definitions from Metworks 1.3.0.200
- TCM-specific deglycosylation reaction definitions
- Prototype component structures/masses (e.g., PATBS-characterized in vitro components of the herb)
- Candidate in vivo metabolite feature list with measured m/z and MS2 spectra

## Outputs

- Complete predictable reaction catalog (one- and two-step Metworks reactions plus TCM deglycosylation reactions)
- Scored metabolite-prototype assignment table (m/z score 0/70 and MS2 similarity score 0/20/30 per pair), e.g., predictable_reaction_catalog_scores.csv

## How to apply

In R, load the known predictable metabolic reaction definitions provided by Metworks 1.3.0.200 as the base rule set. Append the TCM-specific deglycosylation reactions to the base set and enumerate both one-step and two-step transformation products to form the complete predictable reaction catalog. For each candidate metabolite-prototype pair within the MDRB bridging framework, compute an m/z assignment score, awarding 70 points when the m/z matches within a ±5 ppm tolerance. Score MS2 fragment similarity by match degree — 20 points for degree 1, 30 points for degree ≥2, and 0 points when no match is found — then combine the m/z and MS2 components into the overall pair score and export the scored catalog and assignments (e.g., to predictable_reaction_catalog_scores.csv).

## Related tools

- **Metworks 1.3.0.200** (Provides the base set of known predictable metabolic reaction definitions used to enumerate one- and two-step transformation products)
- **MDRB** (R-based mass dataset relevance bridging platform in which the reaction catalog is consumed to score and bridge metabolite-prototype pairs) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation language for loading the reaction rules, appending TCM deglycosylation reactions, enumerating products, and writing the scored assignment table)

## Evaluation signals

- The reaction catalog contains reactions derived from both Metworks 1.3.0.200 and the appended TCM deglycosylation reactions
- m/z assignment scores are exactly 70 only when the pair matches within ±5 ppm
- MS2 similarity scores follow the degree rule: 20 for match degree 1, 30 for degree ≥2, and 0 when no match is found
- Overall pair scores equal the sum of the m/z and MS2 similarity components
- Inspection of the output file (e.g., predictable_reaction_catalog_scores.csv) shows the correct per-pair scores and complete catalog entries

## Limitations

- No software versions are stated for the MDRB data-processing platform beyond Metworks 1.3.0.200
- Reaction enumeration is limited to one- or two-step transformations, so metabolites requiring three or more steps from the prototype will not be bridged
- The parent article reports no statistical validation, replicate counts, or quantitative performance metrics for the scoring strategy
- Identifying in vivo components remains difficult despite MS data post-processing

## Evidence

- [intro] Metworks base reaction set: "We based the known predictable metabolic responses provided by Metworks 1.3.0.200."
- [intro] MDRB integration of m/z, MS2 fragments, mass defect: "we integrated the mass spectrometry data of m/z, MS2 fragment matching and mass defect of the difference limit, to establish a novel mass dataset relevance bridging (MDRB) data post-processing"
- [abstract] Scoring approach rationale: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [abstract] Three data dimensions analyzed: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
