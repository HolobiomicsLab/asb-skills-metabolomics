---
name: metabolite-prototype-bridging
description: Use when LC-MS feature tables from non-targeted metabolite identification
  must be linked back to prototype database entries, especially for TCM herbal components
  whose metabolic transformations (e.g. among components sharing the same parent nucleus)
  need bridging.
when_to_use_negative:
- When matching only a single metabolite against the prototype database using MS2
  map similarity — the article notes MTSF technology covers this narrower case and
  MDRB is intended to go beyond it
- When the input is raw MS data not yet processed by PATBS — MDRB is a data post-processing
  technique that consumes already-identified feature tables
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging post-processing platform that performs
    the metabolite-prototype bridging and builds the component relationship network
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Non-targeted precise-and-thorough background-subtraction technique that generates
    the prototype database and unpredictable-metabolite feature table consumed by
    MDRB
- name: R
  role: Language in which the MDRB data post-processing technique is implemented and
    run
  repo: https://github.com/933ZhangDD/MDRB
- name: Metworks
  role: Provides known predictable metabolic responses (version 1.3.0.200) used alongside
    MDRB's unpredictable-pathway matching
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:6fc0ecf3613c95c565da4ef86c3a259212836893c1dd77402122f58687327a90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-prototype-bridging

## Summary

Bridges in vivo metabolites to their prototype (parent) components by sequentially integrating three dimensions of mass spectral data — m/z, MS1/MS2 mass defect, and MS2 fragment similarity — using the R-based MDRB post-processing platform, with a ±25 mmu mass defect range optimized for unpredictable pathway matching.

## When to use

Use when LC-MS feature tables from non-targeted metabolite identification must be linked back to prototype database entries, especially for TCM herbal components whose metabolic transformations (e.g. among components sharing the same parent nucleus) need bridging.

## When NOT to use

- When matching only a single metabolite against the prototype database using MS2 map similarity — the article notes MTSF technology covers this narrower case and MDRB is intended to go beyond it
- When the input is raw MS data not yet processed by PATBS — MDRB is a data post-processing technique that consumes already-identified feature tables

## Inputs

- Prototype database (in vitro component structures) from PATBS non-targeted identification
- Unpredictable-metabolite feature table (in vivo MS1/MS2 features) from PATBS
- Metworks 1.3.0.200-provided known predictable metabolic responses (for predictable pathway handling)

## Outputs

- Bridged metabolite-prototype pairs with m/z, MS1/MS2 mass defect, and MS2 fragment similarity scores
- Component relationship network linking in vivo metabolites to in vitro prototype components
- Per-range mass defect metrics (pathway coverage, positivity rate, false positives) written to mass_defect_optimization.csv

## How to apply

Load the prototype database and unpredictable-metabolite feature table produced by PATBS into the R-based MDRB data post-processing platform. Score metabolite-prototype pairs by sequentially analyzing the three dimensions of mass spectral data: m/z, mass defect of MS1 and MS2, and similarity of MS2 fragments, based on structural similarity and minimal mass defect variations. Run the matching procedure with the MS1/MS2 mass defect range set to ±25 mmu — the optimized setting from a ±10/±25/±50/±75 mmu comparison (Fig. S3), where narrower ranges reduce covered metabolic pathways and wider ranges raise false positives. Construct a component relationship network that bridges prototypes and metabolites, enabling correlation of in vitro and in vivo data. Verify by tabulating pathway coverage, positivity rate, and false-positive bridged pairs at each mass defect range before fixing the optimum.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging post-processing platform that performs the metabolite-prototype bridging and builds the component relationship network) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Non-targeted precise-and-thorough background-subtraction technique that generates the prototype database and unpredictable-metabolite feature table consumed by MDRB)
- **R** (Language in which the MDRB data post-processing technique is implemented and run) — https://github.com/933ZhangDD/MDRB
- **Metworks** (Provides known predictable metabolic responses (version 1.3.0.200) used alongside MDRB's unpredictable-pathway matching)

## Examples

```
Rscript MDRB_bridging.R --prototype_db prototypes.csv --features unpredictable_metabolites.csv --mass_defect_range 25 --output mass_defect_optimization.csv
```

## Evaluation signals

- Mass defect range sensitivity check: compare ±10, ±25, ±50, ±75 mmu runs on pathway coverage, positivity rate, and false-positive bridged pairs; ±25 mmu should offer the best balance
- Bridged pairs must be supported by minimal mass defect variations and structural similarity between metabolite and prototype
- The output component relationship network should connect metabolites sharing the same parent nucleus structure to their prototypes
- In vitro and in vivo data should be correlatable through the bridged pairs (exposure/metabolism/disposition workflow completes)
- mass_defect_optimization.csv should contain per-range metrics and the selected optimum

## Limitations

- Identification of in vivo components remains difficult despite MS data post-processing techniques
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy
- No software versions are stated for the MDRB technique or its data-processing platform
- No specific mass spectrometry instrument model or data acquisition parameters are named, limiting reproducibility of acquisition-side parameters

## Evidence

- [abstract] The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [intro] mass defect of MS2 fragments of prototypes and metabolites is within a certain range: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [intro] MDRB technique to characterize metabolites and bridge correlation between prototype and metabolites: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [discussion] we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
- [abstract] We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes"
