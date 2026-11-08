---
name: ms2-spectrum-quality-assessment
description: Use when raw MS2 peak lists must be vetted for compound-identification
  reliability before being trusted in a metabolite annotation or identification workflow.
when_to_use_negative:
- The input is not MS2 peak-list data from spectra intended for compound identification.
- A spectral-comparison result (e.g. similarity scores against reference spectra)
  is already available and the question is which reference compound matches, not whether
  the query spectrum itself is reliable.
- The task concerns annotating oxidized glycerophosphocholines, which is a separate
  CMM 3.0 service rather than the quality-assessment service.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Hosts the spectral quality-assessment service that accepts MS2 peak lists
    and returns a reliability verdict plus numeric quality score for compound identification,
    exposed via its RESTful API.
- name: CEU Mass Mediator
  role: Online metabolite annotation tool at http://ceumass.eps.uspceu.es/ whose RESTful
    API provides the quality-assessment endpoint used by this skill.
- name: HMDB (Human Metabolome Database)
  role: 'Downstream consumer: the CMM RESTful API has already been integrated into
    HMDB, so the same quality-assessment call is available through HMDB.'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:85fc677f42ec00804fe31038ed012d6535c7c3bac3abdc9014541377b02fedc6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator
  - CEU Mass Mediator (CMM) 3.0
  - HMDB (Human Metabolome Database)
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ms2-spectrum-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses the reliability of an MS2 spectrum for compound identification using the CEU Mass Mediator (CMM) 3.0 spectral quality-assessment service, which is exposed via the tool's RESTful API alongside its MS2 spectral-comparison service.

## When to use

Use when raw MS2 peak lists must be vetted for compound-identification reliability before being trusted in a metabolite annotation or identification workflow.

## When NOT to use

- The input is not MS2 peak-list data from spectra intended for compound identification.
- A spectral-comparison result (e.g. similarity scores against reference spectra) is already available and the question is which reference compound matches, not whether the query spectrum itself is reliable.
- The task concerns annotating oxidized glycerophosphocholines, which is a separate CMM 3.0 service rather than the quality-assessment service.

## Inputs

- MS2 peak lists of public spectra (one list per spectrum)

## Outputs

- Per-spectrum reliability verdict for compound-identification purposes
- Numeric quality score per spectrum
- spectrum_quality_report.csv with exactly one row per input spectrum, each carrying a verdict and a numeric quality score

## How to apply

Collect the MS2 peak lists of the public spectra that will feed the downstream spectral-comparison step. Submit each peak list to the CMM 3.0 spectral quality-assessment service through its RESTful API (hosted at http://ceumass.eps.uspceu.es/), which returns a reliability verdict for compound-identification purposes plus a numeric quality score per spectrum. Record the verdicts and scores in a per-spectrum table and export it as spectrum_quality_report.csv, keeping one row per input spectrum. Use these results to decide which spectra are reliable enough for compound identification before running the MS2 spectral-comparison service. The rationale is that CMM 3.0 was built with 'new services to support researchers in the metabolite annotation and identification process', and this quality-assessment service is one of those new capabilities in version 3.0, which also increased the number of experimental metabolites to 332,665 and predicted metabolites to 681,198.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Hosts the spectral quality-assessment service that accepts MS2 peak lists and returns a reliability verdict plus numeric quality score for compound identification, exposed via its RESTful API.)
- **CEU Mass Mediator** (Online metabolite annotation tool at http://ceumass.eps.uspceu.es/ whose RESTful API provides the quality-assessment endpoint used by this skill.)
- **HMDB (Human Metabolome Database)** (Downstream consumer: the CMM RESTful API has already been integrated into HMDB, so the same quality-assessment call is available through HMDB.)

## Examples

```
curl -X POST http://ceumass.eps.uspceu.es/ -H 'Content-Type: application/json' -d '{"ms2_peaks":[[m/z, intensity], ...]}'  # returns verdict + quality score
```

## Evaluation signals

- The report spectrum_quality_report.csv exists and contains exactly one row per input spectrum.
- Every row in spectrum_quality_report.csv has a reliability verdict field populated.
- Every row in spectrum_quality_report.csv has a numeric quality score.
- The service used is the CMM 3.0 spectral quality-assessment service reached through its RESTful API, not an ad-hoc heuristic.
- Spectra judged unreliable are excluded from or flagged in the subsequent MS2 spectral-comparison step.

## Limitations

- The service determines reliability for compound-identification purposes only; it does not itself identify the compound.
- It is one of the CMM 3.0 services and therefore requires access to the online tool (or HMDB integration); deployment of the CMM source also requires access to an internal CEU database requested via <maintainer e-mail, see the repository README> and a configured app-server data-source with provided credentials.
- The provided article text does not disclose the scoring thresholds or verdict categories used internally by the service.

## Evidence
