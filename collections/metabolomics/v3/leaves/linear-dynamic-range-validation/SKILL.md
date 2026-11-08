---
name: linear-dynamic-range-validation
description: Use when processing lipidomics peak tables (e.g., LipiDex 2 Peak Finder/Spectral
  Searcher results) where quantitation confidence must be validated against a dilution-series-derived
  linear dynamic range before downstream statistics.
when_to_use_negative:
- Input is already a fully QC-filtered feature table that has passed Degreaser's 7
  filtering criteria
- Analysis uses raw MS data without peak tables or spectral matches from the LipiDex
  2 pipeline — Degreaser takes in results from the peak identification and spectral
  matching steps
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: Executes dilution series linear dynamic range modeling and removes biosample
    quantitations outside the linear range
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Parent pipeline whose Peak Finder and Spectral Searcher outputs are the inputs
    to this filter
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:c3d9bc92869c114c6a066f7e7077ca0491fe79281b193374c7cddff0a4799af7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# linear-dynamic-range-validation

## Summary

A QC filter within the LipiDex 2 Degreaser module that models linear dynamic range from a dilution series and removes lipid quantitations falling outside the established linear range, improving identification-rate confidence in LC-MS lipidomics.

## When to use

Use when processing lipidomics peak tables (e.g., LipiDex 2 Peak Finder/Spectral Searcher results) where quantitation confidence must be validated against a dilution-series-derived linear dynamic range before downstream statistics.

## When NOT to use

- Input is already a fully QC-filtered feature table that has passed Degreaser's 7 filtering criteria
- Analysis uses raw MS data without peak tables or spectral matches from the LipiDex 2 pipeline — Degreaser takes in results from the peak identification and spectral matching steps

## Inputs

- LipiDex 2 Peak Finder filtered results (.csv peak table)
- LipiDex 2 Spectral Searcher spectral match results for biosample experimental runs
- Dilution series data for linear dynamic range modeling
- Replicate quality control sample quantitations

## Outputs

- Filtered lipid identification and feature table retaining only quantitations within the linear range
- Single-filter attribution summary counting identifications removed by each criterion
- Updated identification-rate metrics (e.g., 5% pre-filter to 19% post-filter on MSV000091033)

## How to apply

Load Peak Finder filtered results and Spectral Searcher spectral matches into the LipiDex 2 Degreaser QC module. Degreaser performs dilution series linear dynamic range modeling and retains only biosample quantitations that fall within the modeled linear range, as one of its six automated evidence types. This filter is applied alongside the ECN vs. RT model, replicate QC-sample %RSD check (flagging features with >30% RSD), and Compound Discoverer Peak Quality Factor (PQF) criteria; results are aggregated across 7 filtering criteria to produce the final filtered table. On the MSV000091033 Nile rat plasma dataset (40 experimental samples), applying these filters reduced results to 461 identifications (254 identified lipids) and 2443 unidentified features, raising the identification rate from 5% to 19%. Rationale: quantitations outside the linear range are unreliable for relative comparison, so removing them reduces spurious and redundant features before statistical analysis.

## Related tools

- **LipiDex 2 Degreaser QC module** (Executes dilution series linear dynamic range modeling and removes biosample quantitations outside the linear range) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Parent pipeline whose Peak Finder and Spectral Searcher outputs are the inputs to this filter) — https://github.com/coongroup/LipiDex-2

## Evaluation signals

- Filtered output matches expected counts for the MSV000091033 Nile rat plasma dataset: 254 identified lipids, 461 identifications, 2443 unidentified features
- Final identification rate is 19%, up from 5% pre-filter
- Single-filter attribution table is consistent with 173-of-254 attributions
- Features with >30% RSD in replicate QC samples are flagged for removal
- All remaining biosample quantitations fall within the modeled dilution-series linear range

## Limitations

- Requires a dilution series to establish the linear dynamic range; datasets lacking dilution-series data cannot support this filter
- Degreaser is a hybrid workflow — automated threshold filtering may need follow-up manual validation by the mass spectrometrist
- Filtering removes identified lipids (254 removed on the Nile rat plasma dataset) as the cost of raising the identification rate

## Evidence
