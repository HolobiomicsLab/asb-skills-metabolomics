---
name: lipid-feature-count-reconciliation
description: Use when a LipiDex 2 (or similar) discovery lipidomics run has Peak Finder
  and Spectral Searcher outputs plus Degreaser QC filtering results, and you must
  report how feature counts and identification rate changed after filtering.
when_to_use_negative:
- Input is a single unfiltered peak table with no spectral search or QC filtering
  performed — there are no pre/post stages to reconcile
- The goal is lipid identification or spectral matching itself rather than count bookkeeping
  across filtering stages
- Data were not processed through a LipiDex 2-style pipeline with identifiable filtering
  criteria (e.g., adduct/dimer removal, RT modeling, %RSD), so per-filter attribution
  is undefined
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: Applies the automated QC filters (ECN vs. RT, linear dynamic range, spectral
    match validation, %RSD, PQF) whose pre/post counts are reconciled
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Parent pipeline (Peak Finder and Spectrum Searcher) producing the input feature
    tables and spectral matches
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:c72431de403e14df5a04f7d39a4f3dd02d45a48a1a281247a7f9911f9e27603c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lipid-feature-count-reconciliation

## Summary

Reconciles lipid feature and identification counts across pre- and post-QC-filter stages of the LipiDex 2 Degreaser pipeline, quantifying how many identified lipids, identifications, and unidentified features survive each filtering criterion and the resulting change in identification rate.

## When to use

Use when a LipiDex 2 (or similar) discovery lipidomics run has Peak Finder and Spectral Searcher outputs plus Degreaser QC filtering results, and you must report how feature counts and identification rate changed after filtering.

## When NOT to use

- Input is a single unfiltered peak table with no spectral search or QC filtering performed — there are no pre/post stages to reconcile
- The goal is lipid identification or spectral matching itself rather than count bookkeeping across filtering stages
- Data were not processed through a LipiDex 2-style pipeline with identifiable filtering criteria (e.g., adduct/dimer removal, RT modeling, %RSD), so per-filter attribution is undefined

## Inputs

- LipiDex 2 Peak Finder filtered results table (from Compound Discoverer 3.3 .cdResult peak table)
- Spectrum/Spectral Searcher spectral match results (.csv) for experimental samples
- Replicate quality control sample annotations for %RSD computation
- Compound Discoverer Peak Quality Factor (PQF) metadata

## Outputs

- Final Degreaser-filtered feature table with counts of identified lipids, identifications, and unidentified features
- Pre- vs. post-filter identification rate comparison (e.g., 5% to 19%)
- Single-filter attribution summary (identifications removed per criterion)

## How to apply

Load the Peak Finder filtered results and Spectral Searcher spectral matches for all experimental samples (e.g., the 40 Nile rat plasma samples from MSV000091033) into the LipiDex 2 Degreaser QC module, which applies automated filters including ECN vs. RT modeling, dilution series linear dynamic range modeling, spectral match validation for all matches in each lipid feature, >30% RSD flagging in replicate QC samples, and Compound Discoverer Peak Quality Factor (PQF) criteria. Aggregate results across the 7 filtering criteria to produce a final filtered table, then tally counts of identified lipids, total identifications, and unidentified features before versus after filtering. Compute the identification rate change (in the Nile rat plasma benchmark this rose from 5% pre-filter to 19% post-filter, with 254 identified lipids and 461 identifications removed to leave 2443 unidentified features). Optionally compile a single-filter attribution summary counting how many identifications were removed by each individual criterion (e.g., 173 of 254 by a single filter). Validate by checking that the filtered output's counts and identification rate match the expected reconciled values.

## Related tools

- **LipiDex 2 Degreaser QC module** (Applies the automated QC filters (ECN vs. RT, linear dynamic range, spectral match validation, %RSD, PQF) whose pre/post counts are reconciled) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Parent pipeline (Peak Finder and Spectrum Searcher) producing the input feature tables and spectral matches) — https://github.com/coongroup/LipiDex-2/releases/latest

## Evaluation signals

- Filtered output contains the expected reconciled counts (e.g., 254 identified lipids removed, 461 identifications, 2443 unidentified features on MSV000091033)
- Final identification rate matches expected value (19%, up from 5% pre-filter)
- Single-filter attribution table is internally consistent (e.g., 173 of 254 attributions) and totals reconcile with the overall removed count
- Pre-filter counts must equal post-filter counts plus removed counts for each category (identified lipids, identifications, unidentified features)
- All removed features are attributable to at least one of the 7 filtering criteria

## Limitations

- Counts and identification-rate improvements are dataset-dependent; the 5%→19% figure is specific to the Nile rat plasma dataset (MSV000091033) and will not transfer to other datasets
- Degreaser is a hybrid workflow: automated threshold filtering is followed by optional manual validation, so fully automated reconciliation may omit changes introduced during manual review
- The LipiDex 2 Wiki was a work in progress ('WiP'), so operational details may need to be confirmed from the paper or source

## Evidence
