---
name: chromatographic-peak-smoothing-and-noise-estimation
description: Use when performing semi-targeted LC-MS peak quantification on converted
  Aird data (e.g. TripleTOF or QE HF acquisitions) and a smoothing or noise-estimation
  strategy must be selected among MetaPro's 4 smoothing and 3 noise-estimation options
  before peak selection and integration.
when_to_use_negative:
- Input is already a feature table or peak list with no raw chromatographic signal
  — smoothing and noise estimation have nothing to operate on
- Data was not acquired with the data-dependent acquisition (DDA) method supported
  by MetaPro
- The goal is untargeted feature discovery rather than semi-targeted quantification
  against an mz/rt library
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaPro
  role: Web application whose quantification pipeline hosts the four EIC smoothing
    methods and three EIC noise estimation methods, selectable via the Method object;
    started via MetaproStart scripts with Redis and MongoDB
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor files to the Aird computational format (.aird + . index) whose
    rearranged, compressed, indexed spectra enable rapid EIC extraction prior to smoothing
    and noise estimation
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Started alongside MetaPro (MetaproStart.sh/Redis) as part of the local MetaPro
    runtime that serves the analysis pipeline
- name: MongoDB
  role: Started alongside MetaPro (MetaproStart.sh/MongoDB) as part of the local MetaPro
    runtime that stores project/method/analysis state
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:f7e9871fbee5a004dc4fe19f78584c8afb87fa034e90898fd9808d9b2f0ed9e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chromatographic-peak-smoothing-and-noise-estimation

## Summary

Configurable EIC smoothing and noise estimation as implemented in MetaPro's eight-step quantification pipeline, which exposes four EIC smoothing methods and three EIC noise estimation methods selectable through the Method parameter set before peak selection and integration.

## When to use

Use when performing semi-targeted LC-MS peak quantification on converted Aird data (e.g. TripleTOF or QE HF acquisitions) and a smoothing or noise-estimation strategy must be selected among MetaPro's 4 smoothing and 3 noise-estimation options before peak selection and integration.

## When NOT to use

- Input is already a feature table or peak list with no raw chromatographic signal — smoothing and noise estimation have nothing to operate on
- Data was not acquired with the data-dependent acquisition (DDA) method supported by MetaPro
- The goal is untargeted feature discovery rather than semi-targeted quantification against an mz/rt library

## Inputs

- Vendor LC-MS instrument files (e.g. TripleTOF, QE HF) converted to .aird data files with . index files
- Analyte and internal standard library tables (.txt/.tsv with at least mz and rt columns)
- MetaPro Method parameter set (smoothing method, noise estimation method, peak selection and integration options)

## Outputs

- Smoothed EIC traces with per-peak noise estimates
- Per-compound peak boundaries ready for two-point re-integration
- Quantification results exported to a named .xlsx report per overview

## How to apply

First convert vendor files to the Aird computational format with AirdPro, since MetaPro extracts EICs against Aird's rearranged, compressed, and indexed spectra for rapid MS data access. Load the analyte and internal standard libraries (.txt/.tsv tables containing at least mz and rt columns) and create a Method on the Method page, choosing one of the four EIC smoothing methods and one of the three EIC noise estimation methods — these are selectable via a method-like parameter set analogous to the MetaPro Method object. Run smoothing and noise estimation as steps 2–3 of the eight-step workflow (EIC extraction, smoothing, noise estimation, peak selection, integration, screening, identification, batch optimization), then proceed to peak selection using one of the three peak selection methods and integration with one of the two integration methods. If peak boundaries are wrong, use the manual hooks — 'Reset integration params', 'Reset TargetRT', and 'Manual Integration' (choosing two data points for two-point start/end re-integration) — and apply corrections batch-wide. Export the per-batch quantification of an overview into a named .xlsx report.

## Related tools

- **MetaPro** (Web application whose quantification pipeline hosts the four EIC smoothing methods and three EIC noise estimation methods, selectable via the Method object; started via MetaproStart scripts with Redis and MongoDB) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor files to the Aird computational format (.aird + . index) whose rearranged, compressed, indexed spectra enable rapid EIC extraction prior to smoothing and noise estimation) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Started alongside MetaPro (MetaproStart.sh/Redis) as part of the local MetaPro runtime that serves the analysis pipeline)
- **MongoDB** (Started alongside MetaPro (MetaproStart.sh/MongoDB) as part of the local MetaPro runtime that stores project/method/analysis state)

## Examples

```
bash MetaproStart.sh  # start MetaPro (Redis + MongoDB + web app), then browse localhost:8080 to configure the Method and run Target Search
```

## Evaluation signals

- The Method object exposes exactly the required method counts: 4 EIC smoothing methods, 3 EIC noise estimation methods, 3 peak selection methods, and 2 integration methods
- The pipeline runs end-to-end (script_runs) on the TripleTOF and QE HF tutorial datasets loaded as .aird/. files
- A non-empty .xlsx quantification report is produced per overview (file_exists / file_format_is)
- Manual corrections via 'Reset integration params', 'Reset TargetRT', and two-point 'Manual Integration' reproduce the same peak boundaries when re-applied batch-wide

## Limitations

- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method only
- The MetaPro release used is for academic use only; commercial use requires contacting the authors
- MetaPro is a semi-targeted workflow combining artificial judgment and algorithm inference — manual inspection of peak boundaries (two-point re-integration) remains indispensable for verifying quantification

## Evidence

- [supplementary] MetaPro uses in-memory caching technology to speed up the analysis of MS data.: "MetaPro uses in-memory caching technology to speed up the analysis of MS data."
- [methods] Set Method parameters: "Enter Method page, click `Create` at the top."
- [methods] Manual Integration: "Click on `Manual Integration`, you can choose two data points to conduct manual integration functions."
- [readme] AirdPro conversion role: [evidence span withheld — anchor status: external_doc]
- [methods] Target Search start: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
