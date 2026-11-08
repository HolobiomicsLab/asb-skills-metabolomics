---
name: compound-identification-reliability-scoring
description: Use when you have MS2 peak lists that need vetting before metabolite
  annotation/identification and you need a per-spectrum reliability verdict plus a
  numeric quality score.
when_to_use_negative:
- Input is already a metabolite annotation/identification result table and no per-spectrum
  MS2 peak lists are available.
- The data are MS1-only (no MS2 peak lists) — the service assesses quality of spectra
  for the MS2 spectral comparison workflow.
- The goal is filtering/scoring candidate query results by experimental setup and
  compound properties — that is CMM 2.0's expert-system filter step, not spectral
  quality assessment.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM 3.0)
  role: Provides the spectral quality-assessment service, accessed via its RESTful
    API, that yields reliability verdicts and numeric quality scores for MS2 spectra.
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:69a09e168ca69712c2a9da2b9d244a9f19656a07d628e3615bf2704098a6503e
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
  - CEU Mass Mediator (CMM 3.0)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# compound-identification-reliability-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the reliability of MS2 spectra for compound identification purposes using CMM 3.0's spectral quality-assessment service, exposed via its RESTful API, producing per-spectrum reliability verdicts and numeric quality scores.

## When to use

Use when you have MS2 peak lists that need vetting before metabolite annotation/identification and you need a per-spectrum reliability verdict plus a numeric quality score.

## When NOT to use

- Input is already a metabolite annotation/identification result table and no per-spectrum MS2 peak lists are available.
- The data are MS1-only (no MS2 peak lists) — the service assesses quality of spectra for the MS2 spectral comparison workflow.
- The goal is filtering/scoring candidate query results by experimental setup and compound properties — that is CMM 2.0's expert-system filter step, not spectral quality assessment.

## Inputs

- MS2 peak lists of public spectra (input to the spectral comparison service)

## Outputs

- spectrum_quality_report.csv — one row per input spectrum, each with a reliability verdict field and a numeric quality score

## How to apply

Load the MS2 peak lists of the public spectra that feed the spectral comparison service and submit each peak list to the CMM 3.0 spectral quality-assessment service through its RESTful API. The service, operating alongside the MS2 spectral comparison service in the metabolite annotation and identification workflow, returns a reliability verdict for compound identification purposes and a numeric quality score per spectrum. Assemble the per-spectrum verdicts and scores into a table and write it as spectrum_quality_report.csv. The quality-assessment step sits downstream of expert-system filtering/scoring of query results but precedes committing to compound annotation, so spectra deemed unreliable should not be used for identification claims.

## Related tools

- **CEU Mass Mediator (CMM 3.0)** (Provides the spectral quality-assessment service, accessed via its RESTful API, that yields reliability verdicts and numeric quality scores for MS2 spectra.) — https://github.com/albertogilf/ceuMassMediator

## Evaluation signals

- spectrum_quality_report.csv contains exactly one row per input spectrum (row count equals number of submitted peak lists).
- Every row has a non-empty reliability verdict field.
- Every row has a numeric quality score (parseable as a number).
- Per-spectrum verdicts and scores were obtained by submitting each MS2 peak list to the CMM 3.0 RESTful API, not fabricated or copied from the spectral comparison service output.
- Spectra with unreliable verdicts are excluded from downstream compound identification claims.

## Limitations

- Deployment/use of CMM requires access to an internal CEU database, which must be requested via email (<maintainer e-mail, see the repository README>), plus configuration of a data-source on the app server with provided credentials and the javax.faces-2.2.12.jar dependency.
- The abstract-level evidence describes the service's purpose but does not document numeric score ranges, verdict thresholds, or API request/response schemas.
- Quality verdicts apply to the reliability of spectra for compound identification purposes; they do not themselves annotate metabolites — annotation proceeds via the MS2 spectral comparison and annotation services.

## Evidence

- [abstract] a spectral quality-assessment service to determine the reliability of a spectrum for compound identification purposes: "a spectral quality-assessment service to determine the reliability of a spectrum for compound identification purposes"
- [abstract] a RESTful API has been created, and it has already been integrated into the HMDB (Human Metabolome Database): "a RESTful API has been created, and it has already been integrated into the HMDB (Human Metabolome Database)"
- [abstract] a service for spectral comparison from MS 2 data: "a service for spectral comparison from MS 2 data"
- [readme] It is necessary to access an internal database of CEU. Request access to <maintainer e-mail, see the repository README>.: "It is necessary to access an internal database of CEU. Request access to <maintainer e-mail, see the repository README>."
