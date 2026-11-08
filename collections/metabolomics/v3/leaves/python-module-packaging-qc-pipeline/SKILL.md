---
name: python-module-packaging-qc-pipeline
description: Reference for packaging a Python LC-MS/MS QC pipeline as a pip-installable
  module with auto-installed Python dependencies and manually installed external tools
  (MSConvert, MS-DIAL).
when_to_use_negative:
- The pipeline's non-Python dependencies (MSConvert, MS-DIAL) must be installed manually
  — do not attempt to include them as pip dependencies.
- Rapid QC-MS was designed for Windows platforms because of MSConvert and MS-DIAL
  dependencies; packaging for other OSes requires reimplementing or substituting those
  tools.
- Only Thermo acquisition sequences and Thermo RAW files have been tested extensively;
  packaging a claim of universal vendor support would overstate the validated scope.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: 'The packaged QC pipeline itself: distributed as a pip-installable module
    that provides automated LC-MS/MS quality control, dashboard updates, and Slack/email
    notifications.'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Non-Python dependency installed manually; converts vendor raw data to open
    mzML format before routing to the data processing pipeline.
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Non-Python dependency installed manually; performs data processing and identification
    after conversion.
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: Python
  role: Runtime for the packaged module (compatible with versions 3.8–3.11); dependencies
    are installed automatically during setup.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:30d916866c83baec00dae2631e278b5080ad057928b58ac485be5b6e0af75aa3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - Google API
  - MS-DIAL
  - MSConvert
  - Python
  - Rapid QC-MS
  - Slack API
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# python-module-packaging-qc-pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Package a Python QC pipeline as a distributable module (pip install) whose dependencies (Pandas, SQLAlchemy, Plotly Dash, Watchdog, Google API, Slack API) are installed automatically, while non-Python dependencies (MSConvert, MS-DIAL) are documented for manual installation. Used by Rapid QC-MS to deliver an automated LC-MS/MS quality control tool that runs on Windows.

## When to use

Reference for packaging a Python LC-MS/MS QC pipeline as a pip-installable module with auto-installed Python dependencies and manually installed external tools (MSConvert, MS-DIAL).

## When NOT to use

- The pipeline's non-Python dependencies (MSConvert, MS-DIAL) must be installed manually — do not attempt to include them as pip dependencies.
- Rapid QC-MS was designed for Windows platforms because of MSConvert and MS-DIAL dependencies; packaging for other OSes requires reimplementing or substituting those tools.
- Only Thermo acquisition sequences and Thermo RAW files have been tested extensively; packaging a claim of universal vendor support would overstate the validated scope.

## Inputs

- Python 3.8–3.11 environment
- pip-installable Python package (rapid-qc-ms)
- Python dependencies: Pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API, Slack API
- MSConvert installation for vendor raw data conversion
- MS-DIAL installation for data processing and identification

## Outputs

- Pip-installable rapid-qc-ms package with auto-installed Python dependencies
- CLI entry point `rapidqcms`
- Automated LC-MS/MS quality control pipeline for Windows platforms
- Browser-viewable QC dashboard launched from the package

## How to apply

Structure the QC pipeline (data conversion via MSConvert to mzML, processing and identification via MS-DIAL, metric evaluation, dashboard and notifications) as a Python package compatible with Python 3.8–3.11 and declare its Python dependencies — Pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API, Slack API — so they install automatically during setup, e.g. via `py -m pip install rapid-qc-ms`. Because the pipeline depends on non-Python vendor-conversion and identification software, do not attempt to install those via pip; instead document MSConvert and MS-DIAL as manual prerequisites, referencing an installation guide for details (including optional virtual-environment deployment to isolate installs on shared instrument computers). The package should expose a simple CLI entry point (`rapidqcms`) so users start monitoring without further configuration. Validate the packaging by confirming a fresh install launches the dashboard and that vendor-format conversion (MSConvert → mzML) routes to the data processing pipeline as intended.

## Related tools

- **Rapid QC-MS** (The packaged QC pipeline itself: distributed as a pip-installable module that provides automated LC-MS/MS quality control, dashboard updates, and Slack/email notifications.) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Non-Python dependency installed manually; converts vendor raw data to open mzML format before routing to the data processing pipeline.) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Non-Python dependency installed manually; performs data processing and identification after conversion.) — http://prime.psc.riken.jp/compms/msdial/main.html
- **Python** (Runtime for the packaged module (compatible with versions 3.8–3.11); dependencies are installed automatically during setup.)

## Examples

```
py -m pip install rapid-qc-ms && rapidqcms
```

## Evaluation signals

- Fresh pip install (`py -m pip install rapid-qc-ms`) completes with Python dependencies (Pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API, Slack API) installed automatically.
- Running `rapidqcms` starts the interactive dashboard without further setup.
- MSConvert and MS-DIAL are documented as manual prerequisites, not resolved by pip.
- Package is declared compatible with Python 3.8–3.11 and designed for Windows platforms.
- Converted vendor raw data (mzML) routes correctly to the MS-DIAL processing pipeline.

## Limitations

- MSConvert and MS-DIAL must be installed manually; pip does not resolve them.
- Rapid QC-MS has only been tested extensively on Thermo Fisher instruments, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs.
- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL.
- Installation on MS instrument computers is possible but optional virtual-environment deployment is recommended for extra security.

## Evidence
