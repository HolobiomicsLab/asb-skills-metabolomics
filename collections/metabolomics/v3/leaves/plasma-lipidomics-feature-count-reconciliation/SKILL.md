---
name: plasma-lipidomics-feature-count-reconciliation
description: Use when a hybrid lipidomics QC workflow (automated threshold filtering
  plus manual validation, e.g. Degreaser) has been run on plasma lipid identifications
  and you must reconcile the automated-filter accepted count against the manually
  re-evaluated count.
when_to_use_negative:
- The dataset has not passed through the LipiDex 2 peak identification and spectral
  matching pipeline, so Degreaser has no per-identification filter flags to reconcile
- The workflow used fully automated filtering only with no manual-validation step,
  so there is no second set of filter statuses to compare
- The input is a raw peak table or unfiltered feature list rather than Degreaser-filtered
  identification results
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: LipiDex 2 Degreaser QC module
  role: Provides the hybrid automated-threshold-filtering plus optional manual-validation
    workflow whose filter statuses are reconciled by this skill
  repo: https://github.com/coongroup/LipiDex-2
- name: LipiDex 2
  role: Upstream pipeline (peak identification, spectral matching) that produces the
    identification results Degreaser filters and manual validation re-evaluates
  repo: https://github.com/coongroup/LipiDex-2
- name: Metaboanalyst
  role: Downstream statistical analysis target for the reconciled, filtered identification
    results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:89815ec4b0c3674e0038f36c656ece303e09a27941eaaf5dbfc1b6125fc05c5d
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# plasma-lipidomics-feature-count-reconciliation

## Summary

Reconciles lipid identification counts between Degreaser's automated threshold filtering and post-hoc manual validation, tracking how many filter statuses change (e.g. 46 in the Nile rat plasma MSV000091033 dataset) and how many identifications are recovered (19, from 461 automated to 480 total).

## When to use

Use when a hybrid lipidomics QC workflow (automated threshold filtering plus manual validation, e.g. Degreaser) has been run on plasma lipid identifications and you must reconcile the automated-filter accepted count against the manually re-evaluated count.

## When NOT to use

- The dataset has not passed through the LipiDex 2 peak identification and spectral matching pipeline, so Degreaser has no per-identification filter flags to reconcile
- The workflow used fully automated filtering only with no manual-validation step, so there is no second set of filter statuses to compare
- The input is a raw peak table or unfiltered feature list rather than Degreaser-filtered identification results

## Inputs

- Degreaser-filtered lipid identification results for MSV000091033 (40 Nile rat plasma samples reprocessed with LipiDex 2)
- Per-identification automated filter flags from the six Degreaser evidence types (>30% RSD in QC samples, ECN vs. RT, dilution series, linear range, spectral match validation, PQF)
- Degreaser dashboard QC plots for manual review of flagged identifications

## Outputs

- manual_validation_summary.csv with per-identification filter status decisions
- Reconciled accepted-identification count (461 automated -> 480 after manual validation)
- Tally of identifications whose filter status changed under manual evaluation (46)

## How to apply

Load the Degreaser-filtered results for the reprocessed dataset (e.g. the 40-sample Nile rat plasma dataset MSV000091033 processed with LipiDex 2) with each identification's automated filter outcome from the six evidence types: >30% RSD in QC samples, ECN vs. RT deviation, dilution-series linear dynamic range, quantitations within linear range, spectral match validation, and Compound Discoverer PQF. Re-evaluate, as a mass spectrometrist would in the Degreaser dashboard, each identification flagged by an automated cutoff, because some cutoffs are too stringent or miss nuances visible only in the QC plots. Tally identifications whose filter status changes (the reproduction target is 46 changed statuses), then update the accepted-identification count by the net recovered identifications (target +19, reaching 480 total from the automated count). Write a summary table (e.g. manual_validation_summary.csv) with per-identification decisions and the final counts, and validate against the reported 46 changed / 480 total.

## Related tools

- **LipiDex 2 Degreaser QC module** (Provides the hybrid automated-threshold-filtering plus optional manual-validation workflow whose filter statuses are reconciled by this skill) — https://github.com/coongroup/LipiDex-2
- **LipiDex 2** (Upstream pipeline (peak identification, spectral matching) that produces the identification results Degreaser filters and manual validation re-evaluates) — https://github.com/coongroup/LipiDex-2
- **Metaboanalyst** (Downstream statistical analysis target for the reconciled, filtered identification results)

## Evaluation signals

- Number of identifications with changed filter status after manual evaluation equals 46
- Final accepted-identification count equals 480, exactly 19 above the automated-filter count
- manual_validation_summary.csv contains a per-identification filter status decision for every flagged identification with no missing entries
- Automated-filter baseline count is 461 (480 minus 19 recovered)
- Changes are concentrated among identifications flagged by over-stringent automated cutoffs (e.g. >30% RSD in QC samples, ECN vs. RT deviation)

## Limitations

- Manual validation outcomes depend on mass spectrometrist judgment of the QC plots, so reproduction requires expert-style re-evaluation rather than pure rule application
- The 46/480 figures are specific to the MSV000091033 Nile rat plasma dataset reprocessed with LipiDex 2; other datasets will yield different counts
- LipiDex 2's Degreaser module requires results from the LipiDex 2 peak identification and spectral matching steps, and Peak Finder requires Compound Discoverer 3.3 .cdResult files

## Evidence
