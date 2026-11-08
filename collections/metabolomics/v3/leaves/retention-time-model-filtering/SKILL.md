---
name: retention-time-model-filtering
description: Use when lipid identifications from the LipiDex 2 Peak Finder and Spectral
  Searcher pipeline need QC filtering based on retention-time plausibility, i.e. removing
  identifications whose measured RT deviates substantially from the ECN-predicted
  RT for their lipid class.
when_to_use_negative:
- Input is a raw peak table or .cdResult file that has not yet passed Peak Finder
  and Spectral Searcher — Degreaser takes results from the peak identification and
  spectral matching steps
- A lipid class has fewer than the minimum 5 examples (default), since the class is
  excluded from RT model fitting and its identifications cannot be RT-filtered
- The goal is filtering of unidentified adduct/dimer/in-source-fragment features only
  — that is handled by Peak Finder filtering, not the ECN vs. RT identification model
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipiDex 2 Degreaser QC module
  role: Fits the per-class ECN vs. RT regression and flags/ removes identifications
    with large deviation from predicted RT as one of its automated filters
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Stand-alone executable pipeline whose Peak Finder and Spectral Searcher outputs
    feed the RT model filtering; rewritten in C# .NET 6
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:a17b6577233f6f881ccf868b234531ed382bed5710a185f961888bd6794d7ddd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - C# .NET 6
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# retention-time-model-filtering

## Summary

Applies the LipiDex 2 Degreaser QC module's effective carbon number versus retention time (ECN vs. RT) model, which fits a per-lipid-class regression of RT on fatty acyl carbon count and alkene unsaturations to flag likely incorrect identifications with large RT deviation as part of the 7-criteria filtering workflow.

## When to use

Use when lipid identifications from the LipiDex 2 Peak Finder and Spectral Searcher pipeline need QC filtering based on retention-time plausibility, i.e. removing identifications whose measured RT deviates substantially from the ECN-predicted RT for their lipid class.

## When NOT to use

- Input is a raw peak table or .cdResult file that has not yet passed Peak Finder and Spectral Searcher — Degreaser takes results from the peak identification and spectral matching steps
- A lipid class has fewer than the minimum 5 examples (default), since the class is excluded from RT model fitting and its identifications cannot be RT-filtered
- The goal is filtering of unidentified adduct/dimer/in-source-fragment features only — that is handled by Peak Finder filtering, not the ECN vs. RT identification model

## Inputs

- LipiDex 2 Peak Finder results (peak table with identifications, retention times, fatty acyl compositions; requires Compound Discoverer 3.3 .cdResult-derived peak table upstream)
- LipiDex 2 Spectral Searcher spectral match results (.csv)
- Optional dilution series and replicate QC sample annotations for combined %RSD / linear-range filtering

## Outputs

- Filtered lipid feature table (identified lipids and unidentified features surviving the criteria)
- RT_model_report.csv with per-class ECN vs. RT coefficients, examples used, fit statistics, and model-fitting timing
- Single-filter attribution summary counting removals per criterion (e.g. 173 of 254 removed identifications attributed to a single filter)

## How to apply

Load the Peak Finder filtered results and Spectral Searcher spectral matches (e.g. the 40 Nile rat plasma experimental samples from MSV000091033) into the Degreaser QC module. Parse each lipid identification to extract fatty acyl carbon count and number of alkene unsaturations, then group features by lipid class and fit the ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations) per class, excluding classes with fewer than the user-specified minimum of 5 examples (default). Flag identifications whose measured RT deviates largely from the predicted class RT for removal; this model is one of the 7 criteria Degreaser filters on, alongside dilution-series linear dynamic range, %RSD in replicate QC samples (>30% triggers flagging), and Compound Discoverer Peak Quality Factors. On the LipiDex 2 demo dataset, fitting the underlying data models for ~3000 lipid features completes in under 10 seconds, and the class-level model report (class, coefficients, examples used, fit statistics, timing) is written to RT_model_report.csv. On MSV000091033, applying the automated filters reduced results to 2443 unidentified features and 461 identifications (254 identified lipids filtered), raising the identification rate from 5% to 19%.

## Related tools

- **LipiDex 2 Degreaser QC module** (Fits the per-class ECN vs. RT regression and flags/ removes identifications with large deviation from predicted RT as one of its automated filters) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Stand-alone executable pipeline whose Peak Finder and Spectral Searcher outputs feed the RT model filtering; rewritten in C# .NET 6) — https://github.com/coongroup/LipiDex-2/releases/latest

## Evaluation signals

- RT_model_report.csv is produced containing per-class coefficients, number of examples used, and fit statistics; classes with fewer than 5 examples are excluded from the report
- Model fitting for ~3000 lipid features records an elapsed time under 10 seconds
- Identifications flagged for removal exhibit large deviation from their lipid-class predicted RT; identifications within the class trend are retained
- On MSV000091033, the full automated-filter output contains exactly 254 identified lipids, 461 identifications, 2443 unidentified features, and a 19% final identification rate (up from 5% pre-filter)
- Single-filter attribution table is consistent with 173 of 254 removed identifications attributed to a single filter criterion

## Limitations

- Lipid classes with fewer than the user-specified minimum number of examples (default 5) are excluded from model fitting, so their identifications receive no RT-based filtering
- The model can only filter identifications with fatty acyl composition information; unidentified features are not assessed by the RT model itself
- As one of 7 automated criteria (ECN vs. RT, linear dynamic range, linear-range quantitations, spectral match validation, %RSD with run order, PQF), removals may be confounded across filters and require the attribution summary for interpretation; the LipiDex 2 Wiki documentation is a work in progress (WiP)

## Evidence
