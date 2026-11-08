---
name: degreaser-qc-dashboard-interpretation
description: Use when lipid identifications from a LipiDex 2 pipeline run (e.g. .
when_to_use_negative:
- Input is a raw LC-MS/MS file needing spectral library generation or spectral matching
  first — use the LipiDex 2 Library Generator and Spectrum Searcher modules before
  Degreaser
- Peak tables come from tools other than Compound Discoverer 3.3 or pre-converted
  .csv feature tables outside the LipiDex 2 pipeline
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: Core module performing automated threshold filtering and dashboard-based manual
    validation of lipid identifications
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Upstream pipeline (Library Generator, Spectrum Searcher, Peak Finder, Lipid
    Spectral Annotator) whose .csv results Degreaser consumes
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: Metaboanalyst
  role: Downstream statistical analysis target for Degreaser's filtered results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:c996586cdc618738f71a2e9cddd2d7d7c3bcacd96a0d66c24b8e3315df3017de
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - Metaboanalyst
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# degreaser-qc-dashboard-interpretation

## Summary

Interprets results from the LipiDex 2 Degreaser QC module, which combines automated threshold filtering with a data dashboard for optional manual validation of lipid identifications using six evidence types (ECN-vs-RT modeling, dilution-series dynamic range, QC-sample %RSD, spectral match validation, run-order %RSD, and Compound Discoverer Peak Quality Factors).

## When to use

Use when lipid identifications from a LipiDex 2 pipeline run (e.g. .csv results from Spectrum Searcher/Peak Finder) need automated QC filtering plus manual review of flagged features in the Degreaser dashboard, or when reproducing manual filter-status changes such as the 46 changed identifications in MSV000091033.

## When NOT to use

- Input is a raw LC-MS/MS file needing spectral library generation or spectral matching first — use the LipiDex 2 Library Generator and Spectrum Searcher modules before Degreaser
- Peak tables come from tools other than Compound Discoverer 3.3 or pre-converted .csv feature tables outside the LipiDex 2 pipeline

## Inputs

- LipiDex 2 peak identification and spectral matching results (.csv files passed between stand-alone modules)
- Compound Discoverer 3.3 .cdResult files (Peak Finder peak table input)
- Thermo .raw files (Spectrum Searcher input for spectral matching)

## Outputs

- Filtered lipid identification/feature table with automated and manually revised filter statuses
- Manual validation summary with per-identification filter status decisions (e.g. manual_validation_summary.csv)
- Metaboanalyst-compatible results table for downstream statistical analysis

## How to apply

Feed results from the peak identification and spectral matching steps of the LipiDex 2 pipeline into Degreaser, which evaluates six pieces of evidence: effective carbon number versus retention time (ECN vs. RT) modeling, dilution-series linear dynamic range modeling, biosample quantitations within linear range, spectral match validation for all matches in the lipid feature, replicate QC samples' percent relative standard deviation (%RSD) with run order, and Compound Discoverer Peak Quality Factors (PQF). Features with >30% RSD in QC samples are automatically flagged for removal, and an ECN-vs-RT model filters likely incorrect identifications with large deviation from predicted RT. Then perform manual validation in the dashboard: for each identification flagged by the automated threshold filters, re-evaluate its filter status as a mass spectrometrist would, since some automatic cutoffs are too stringent or miss nuances only visible in the QC plots. Tally identifications whose filter status changes and update the accepted-identification count; export results directly compatible with Metaboanalyst without coding.

## Related tools

- **LipiDex 2 Degreaser QC module** (Core module performing automated threshold filtering and dashboard-based manual validation of lipid identifications) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Upstream pipeline (Library Generator, Spectrum Searcher, Peak Finder, Lipid Spectral Annotator) whose .csv results Degreaser consumes) — https://github.com/coongroup/LipiDex-2/releases/latest
- **Metaboanalyst** (Downstream statistical analysis target for Degreaser's filtered results)

## Examples

```
Extract LipiDex-2 release zip and run: LipiDex2.exe (Degreaser module), loading the Spectrum Searcher/Peak Finder .csv results for MSV000091033, applying ECN-vs-RT and >30% QC-sample %RSD filters, then manually reviewing flagged identifications in the dashboard
```

## Evaluation signals

- Manual validation of the MSV000091033 Nile rat plasma dataset should change the filter status of 46 lipid identifications
- Accepted-identification count after manual validation should be 480, i.e. 19 above the automated-filter count
- Features with >30% RSD in QC samples must be flagged for removal by the automatic filter
- Filtered results should have spurious features (adducts, dimers, in-source fragments) removed by Peak Finder filtering
- Output table must be directly loadable into Metaboanalyst without additional coding

## Limitations

- Automated cutoffs can be too stringent or miss nuances only visible in the QC plots, so fully automated filtering without manual validation may discard valid identifications
- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table
- LipiDex 2 requires .NET 7.0; Peak Finder does not accept arbitrary peak-table formats directly
- The LipiDex 2 Wiki instructions were a work in progress ('WiP') at the time of the README

## Evidence

- [other] Manual evaluation outcome on MSV000091033: [evidence span withheld — anchor status: not_found]
