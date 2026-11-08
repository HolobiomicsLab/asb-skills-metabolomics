---
name: spectral-peak-list-parsing
description: Use when MS2 peak lists from public spectra must be loaded and fed to
  CMM 3.0's spectral quality-assessment service to obtain per-spectrum reliability
  verdicts before compound identification or annotation.
when_to_use_negative:
- The input is not per-spectrum MS2 peak lists (e.g. the goal is CMM 3.0's oxidized
  glycerophosphocholine annotation service or general metabolomic database querying
  rather than spectrum quality assessment)
- Spectra have already been scored — re-running the quality-assessment service would
  only duplicate existing verdicts and quality scores
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Hosts the spectral quality-assessment service (RESTful API) that consumes
    each parsed MS2 peak list and returns a reliability verdict and numeric quality
    score for compound identification
  repo: albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:dd6e96bf3b40c04d67f636ca3e12a04fa54532ae6074f4d92c2a70224b4121a0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  repo_url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spectral-peak-list-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares MS2 peak lists from public spectra so each can be submitted to CEU Mass Mediator (CMM) 3.0's RESTful spectral quality-assessment service, which returns a reliability verdict and numeric quality score per spectrum for compound identification.

## When to use

Use when MS2 peak lists from public spectra must be loaded and fed to CMM 3.0's spectral quality-assessment service to obtain per-spectrum reliability verdicts before compound identification or annotation.

## When NOT to use

- The input is not per-spectrum MS2 peak lists (e.g. the goal is CMM 3.0's oxidized glycerophosphocholine annotation service or general metabolomic database querying rather than spectrum quality assessment)
- Spectra have already been scored — re-running the quality-assessment service would only duplicate existing verdicts and quality scores

## Inputs

- MS2 peak lists of public spectra (one peak list per spectrum)
- Access credentials / data-source for the internal CEU database backing CMM 3.0

## Outputs

- Per-spectrum reliability verdict for compound identification
- Numeric quality score per spectrum
- spectrum_quality_report.csv with exactly one row per input spectrum (verdict field plus numeric quality score)

## How to apply

Load the MS2 peak lists of the public spectra that feed the spectral comparison service, so each list is a discrete unit that can be submitted individually. Submit each peak list to the CMM 3.0 spectral quality-assessment service through its RESTful API, which returns a reliability verdict for compound identification and a numeric quality score per spectrum. Run this alongside the CMM 3.0 MS2 spectral comparison service, as both operate as part of the metabolite annotation and identification workflow. Assemble the per-spectrum verdicts and scores into a table and write it as spectrum_quality_report.csv, ensuring the report contains exactly one row per input spectrum.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Hosts the spectral quality-assessment service (RESTful API) that consumes each parsed MS2 peak list and returns a reliability verdict and numeric quality score for compound identification) — albertogilf/ceuMassMediator

## Evaluation signals

- spectrum_quality_report.csv contains exactly one row per input spectrum
- Every row carries both a reliability verdict field and a numeric quality score
- Verdicts/scores are traceable to submissions made via CMM 3.0's RESTful API (one submission per peak list)
- The report pairs correctly with the parallel MS2 spectral comparison service output in the annotation workflow

## Limitations

- Operating CMM 3.0 requires access to an internal CEU database, requested via email (<maintainer e-mail, see the repository README>), plus a data-source configured on the app server with the provided credentials
- Deployment of the CMM source code requires the javax.faces-2.2.12.jar library on the app server

## Evidence
