---
name: mass-defect-range-optimization
description: Decision support for tuning the mass-defect tolerance in MDRB prototype-metabolite
  bridging when matching unpredictable metabolites and needing to trade off pathway
  coverage against false positives.
when_to_use_negative:
- The matching already uses a validated mass-defect range and the goal is routine
  metabolite bridging rather than parameter calibration.
- Metabolites are handled as predictable metabolic reactions (e.g., via Metworks 1.3.0.200
  transformation rules) rather than unpredictable metabolites requiring mass-defect
  bridging.
- The dataset lacks MS1/MS2 mass-defect information or MS2 fragment similarity data,
  so the three-dimensional MDRB scoring cannot be evaluated per range.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging platform used to run the matching
    procedure at each candidate mass-defect range and score bridged pairs
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Language of the MDRB data post-processing platform in which the range sweep
    and metric tabulation are implemented
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Produces the prototype database and the unpredictable-metabolite feature table
    that feed the range optimization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:b779093b705cf5e5b1fee84c6efb94e89795eb91bf7884d468f08d9008d203c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-defect-range-optimization

## Summary

An optimization procedure within the MDRB strategy that empirically selects the MS1/MS2 mass-defect tolerance window (±25 mmu in the paper) for bridging unpredictable metabolites to prototypes, balancing pathway coverage, positivity rate, and false-positive bridged pairs.

## When to use

Decision support for tuning the mass-defect tolerance in MDRB prototype-metabolite bridging when matching unpredictable metabolites and needing to trade off pathway coverage against false positives.

## When NOT to use

- The matching already uses a validated mass-defect range and the goal is routine metabolite bridging rather than parameter calibration.
- Metabolites are handled as predictable metabolic reactions (e.g., via Metworks 1.3.0.200 transformation rules) rather than unpredictable metabolites requiring mass-defect bridging.
- The dataset lacks MS1/MS2 mass-defect information or MS2 fragment similarity data, so the three-dimensional MDRB scoring cannot be evaluated per range.

## Inputs

- Prototype database produced by PATBS (in vitro TCM component structures)
- Unpredictable-metabolite feature table from in vivo MS data (m/z, MS1/MS2 mass defect, MS2 fragments)
- Candidate mass-defect ranges (e.g., ±10, ±25, ±50, ±75 mmu)

## Outputs

- mass_defect_optimization.csv with per-range metrics (pathway coverage, positivity rate, false-positive bridged pairs)
- Selected optimal mass-defect range (±25 mmu in the reference study)

## How to apply

Load the PATBS-derived prototype database and the unpredictable-metabolite feature table into the R-based MDRB post-processing platform. Score metabolite-prototype pairs by sequentially integrating the three dimensions of mass spectral data — m/z, MS1/MS2 mass defect, and MS2 fragment similarity — using structural similarity and minimal mass defect variations. Re-run the matching with the MS1/MS2 mass-defect range set to ±10, ±25, ±50, and ±75 mmu. For each range, tabulate pathway coverage, positivity rate, and the number of false-positive bridged pairs. Select the range that best balances these metrics: in the paper a narrower window reduced covered metabolic pathways while a wider window raised false positives, so ±25 mmu was chosen for both MS1 and MS2 unpredictable pathway matching.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging platform used to run the matching procedure at each candidate mass-defect range and score bridged pairs) — https://github.com/933ZhangDD/MDRB
- **R** (Language of the MDRB data post-processing platform in which the range sweep and metric tabulation are implemented) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Produces the prototype database and the unpredictable-metabolite feature table that feed the range optimization)

## Evaluation signals

- All four ranges (±10, ±25, ±50, ±75 mmu) were run through the identical MDRB scoring pipeline so the comparison is confounded only by the range itself.
- mass_defect_optimization.csv contains, for every range, all three metrics: pathway coverage, positivity rate, and number of false-positive bridged pairs.
- The selected optimum should fall between the extremes: a narrower range (±10 mmu) shows reduced covered metabolic pathways while a wider range (±50/±75 mmu) shows elevated false positives.
- The final selected range (±25 mmu) is applied consistently to both MS1 and MS2 mass-defect dimensions in subsequent unpredictable pathway matching.

## Limitations

- The optimum (±25 mmu) is empirical and dataset-specific; no software version or fixed default is stated for the MDRB platform.
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the optimization in the article.
- Identification of in vivo components remains difficult despite MS data post-processing techniques.

## Evidence

- [abstract] scoring integrates three MS dimensions: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [intro] mass defect bridges prototypes and metabolites: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [discussion] residual difficulty in component identification: "there are still more difficulties in the identification of components."
