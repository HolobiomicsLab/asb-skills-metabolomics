---
name: lipid-identification-filter-status-validation
description: Use when LipiDex 2 Degreaser automated threshold filtering has flagged
  lipid identifications for removal and a mass spectrometrist must manually validate
  those statuses via QC plots, or when reproducing a manual-validation QC count against
  reported totals.
when_to_use_negative:
- Input is raw spectra or a peak table that has not yet passed through the LipiDex
  2 automated threshold filtering step — manual validation presupposes automated filter
  statuses to re-evaluate
- The goal is fully automated filtering with no expert curation (e.g. outputs only
  need to be Metaboanalyst-ready without manual review)
- The data were not produced by the LipiDex 2 pipeline — Degreaser takes results specifically
  from LipiDex 2 peak identification and spectral matching steps
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: 'Provides the hybrid workflow: automated threshold filtering followed by optional
    manual validation, with interactive QC visualizations for re-deciding filter statuses'
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Upstream pipeline whose peak identification and spectral matching results
    feed Degreaser; produces the filtered identification table being manually validated
  repo: https://github.com/coongroup/LipiDex-2
- name: Metaboanalyst
  role: Downstream statistical analysis target for the validated identification results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:2a987e18df25038b001482e3c9ab331ac63386d65603ee2d9d05347146037be2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - Metaboanalyst
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# lipid-identification-filter-status-validation

## Summary

Manual re-evaluation of Degreaser's automated lipid identification filter decisions using interactive QC visualizations (ECN-vs-RT, %RSD, spectral match validation), changing filter statuses where automated cutoffs were too stringent or missed nuances. Bridges automated threshold filtering and expert curation in the LipiDex 2 hybrid QC workflow.

## When to use

Use when LipiDex 2 Degreaser automated threshold filtering has flagged lipid identifications for removal and a mass spectrometrist must manually validate those statuses via QC plots, or when reproducing a manual-validation QC count against reported totals.

## When NOT to use

- Input is raw spectra or a peak table that has not yet passed through the LipiDex 2 automated threshold filtering step — manual validation presupposes automated filter statuses to re-evaluate
- The goal is fully automated filtering with no expert curation (e.g. outputs only need to be Metaboanalyst-ready without manual review)
- The data were not produced by the LipiDex 2 pipeline — Degreaser takes results specifically from LipiDex 2 peak identification and spectral matching steps

## Inputs

- Degreaser-filtered lipid identification results (.csv) from the LipiDex 2 peak identification and spectral matching pipeline
- Reprocessed LC-MS lipidomics dataset (e.g. MSV000091033, 40 Nile rat plasma samples)
- Degreaser QC dashboard visualizations: ECN-vs-RT model, dilution series linear dynamic range, replicate QC %RSD with run order

## Outputs

- manual_validation_summary.csv with per-identification filter status decisions
- Updated accepted-identification count (reported: 480, +19 over automated count)
- Tally of identifications with changed filter status (reported: 46)

## How to apply

Load the Degreaser-filtered lipid identification results (e.g. the 40-sample Nile rat plasma dataset MSV000091033 reprocessed with LipiDex 2) into the Degreaser QC dashboard. For each identification flagged by the automated threshold filters — >30% RSD in QC samples, ECN-vs-RT deviation, spectral-match and dilution-series checks — re-inspect the interactive QC visualizations (ECN vs. RT model, dilution series linear dynamic range, replicate QC %RSD with run order, spectral match validation) and re-decide the filter status as a mass spectrometrist would. Tally identifications whose status changes from the automated outcome (the reported benchmark was 46 changed statuses) and update the accepted-identification count accordingly (19 above the automated count, reaching 480 total). Write a summary table of per-identification filter decisions and final counts; counts outside reported values indicate a failed reproduction.

## Related tools

- **LipiDex 2 Degreaser QC module** (Provides the hybrid workflow: automated threshold filtering followed by optional manual validation, with interactive QC visualizations for re-deciding filter statuses) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Upstream pipeline whose peak identification and spectral matching results feed Degreaser; produces the filtered identification table being manually validated) — https://github.com/coongroup/LipiDex-2
- **Metaboanalyst** (Downstream statistical analysis target for the validated identification results)

## Evaluation signals

- Reproduced count of identifications with changed filter status matches the reported 46
- Updated accepted-identification total matches the reported 480 (19 above the automated-filter count)
- Summary table contains a per-identification filter status decision for every identification flagged by the automated filters
- Final validated output remains compatible with Metaboanalyst import

## Limitations

- Manual validation is inherently subjective — some automatic cutoffs were too stringent or missed nuances only visible in the QC plots, so outcomes depend on the analyst's judgment
- The 46 changed statuses / 480 total benchmarks are specific to the MSV000091033 Nile rat plasma reprocessing; other datasets will yield different counts
- Degreaser requires results from the LipiDex 2 pipeline; Peak Finder additionally requires Compound Discoverer 3.3 .cdResult files, and .NET 7.0 must be installed

## Evidence
