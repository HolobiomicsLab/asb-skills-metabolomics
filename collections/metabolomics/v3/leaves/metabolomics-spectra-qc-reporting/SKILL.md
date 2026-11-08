---
name: metabolomics-spectra-qc-reporting
description: Use when a metabolomics pipeline holds MS2 peak lists for public spectra
  and must decide, before compound identification, which spectra are reliable enough
  to carry into MS2 spectral comparison and annotation.
when_to_use_negative:
- The input is raw MS1 feature data or a feature table with no MS2 peak lists — the
  CMM quality-assessment service consumes MS2 peak lists.
- The goal is metabolite query filtering/scoring via CMM's expert system on experimental
  setup and compound properties (the CMM 2.0-style filter/score step), not spectrum
  reliability assessment.
- The spectra are in-house query spectra being matched against libraries, not public
  spectra whose reliability must be vetted before compound identification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3292
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the spectral quality-assessment service invoked per MS2 peak list
    through its RESTful API to return a reliability verdict and numeric quality score
    for compound identification
  repo: albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:43ad9ebc6d7998c584787a1fae8dac1603dda334e08fad271b2b97bc734be9be
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolomics-spectra-qc-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A QC-reporting step for metabolomics annotation workflows: each MS2 peak list of a public spectrum is submitted to the CEU Mass Mediator (CMM) 3.0 spectral quality-assessment service through its RESTful API, yielding a per-spectrum reliability verdict and numeric quality score for compound identification.

## When to use

Use when a metabolomics pipeline holds MS2 peak lists for public spectra and must decide, before compound identification, which spectra are reliable enough to carry into MS2 spectral comparison and annotation.

## When NOT to use

- The input is raw MS1 feature data or a feature table with no MS2 peak lists — the CMM quality-assessment service consumes MS2 peak lists.
- The goal is metabolite query filtering/scoring via CMM's expert system on experimental setup and compound properties (the CMM 2.0-style filter/score step), not spectrum reliability assessment.
- The spectra are in-house query spectra being matched against libraries, not public spectra whose reliability must be vetted before compound identification.

## Inputs

- MS2 peak lists of public spectra (one peak list per spectrum) destined for CMM 3.0's MS2 spectral comparison service
- Reachable CMM 3.0 RESTful API endpoint (tool hosted at http://ceumass.eps.uspceu.es/)

## Outputs

- spectrum_quality_report.csv with exactly one row per input spectrum, each row carrying a reliability verdict for compound identification and a numeric quality score

## How to apply

Load the MS2 peak lists of the public spectra that feed CMM 3.0's MS2 spectral comparison service. Submit each peak list to the CMM 3.0 spectral quality-assessment service via its RESTful API (the tool is hosted at http://ceumass.eps.uspceu.es/, and the same RESTful API has been integrated into HMDB). Capture the per-spectrum reliability verdict for compound identification and the numeric quality score returned by the service; no universal score cutoff is published in the article, so gate downstream annotation on the verdict/score semantics the service returns. Assemble the per-spectrum verdicts and scores into a single table and write it as spectrum_quality_report.csv, positioned as a QC gate ahead of the MS2 spectral comparison and metabolite annotation/identification workflow. Validate the report: it must contain exactly one row per input spectrum, each row carrying a reliability verdict field and a numeric quality score; re-submit or exclude any spectrum lacking a verdict rather than passing it to identification.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the spectral quality-assessment service invoked per MS2 peak list through its RESTful API to return a reliability verdict and numeric quality score for compound identification) — albertogilf/ceuMassMediator

## Evaluation signals

- spectrum_quality_report.csv exists and its row count exactly equals the number of MS2 peak lists submitted to the service.
- Every row contains a populated reliability verdict field and a numeric quality score that parses as a number.
- There is a strict 1:1 mapping between submitted spectra and report rows — no duplicated or missing spectra.
- Verdicts and scores originate from the CMM 3.0 RESTful API responses rather than placeholder or default values.
- Downstream MS2 spectral comparison/annotation consumes only spectra whose verdict marks them reliable for compound identification.

## Limitations

- Using CMM requires access to an internal CEU database — access must be requested via <maintainer e-mail, see the repository README> — plus a data-source configured on the app server with the provided credentials.
- Deployment additionally requires the javax.faces-2.2.12.jar library on the app server.
- The article describes the service only at abstract level; no numeric score cutoffs or verdict-category definitions are given in the provided context, so thresholds must come from the API response or service documentation.
- Verdicts assess reliability for compound identification; they are not a substitute for processing-level QC of raw mass-spectrometry data.

## Evidence
