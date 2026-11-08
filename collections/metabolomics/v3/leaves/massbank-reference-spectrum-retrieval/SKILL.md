---
name: massbank-reference-spectrum-retrieval
description: Use when you have an experimental MS2 peak list (m/z, intensity pairs)
  from metabolomics LC-MS/MS data and need reference spectra for candidate metabolites
  plus a per-pair spectral similarity score for annotation.
when_to_use_negative:
- Input data is MS1-only (no MS2 peak list), since the CMM 3.0 service performs spectral
  comparison from MS2 data specifically.
- The goal is filtering/scoring candidate compound lists by experimental setup and
  compound properties — use CMM's expert-system filtering step instead of spectral
  comparison.
- The task is spectral quality assessment (deciding whether a spectrum is reliable
  for compound identification), which is a separate CMM 3.0 service.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the MS2 spectral-comparison service that resolves MassBank reference
    spectra and computes the cosine-similarity score per spectrum pair
  repo: albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:31a0c59c2e01f691d91d2449d65fa9d74d443ecddb37de641e2ad3e356935c57
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
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# massbank-reference-spectrum-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves MassBank reference MS2 spectra through CEU Mass Mediator (CMM) 3.0's spectral-comparison service and computes cosine-similarity scores in [0, 1] between an experimental MS2 peak list and each matched reference spectrum.

## When to use

Use when you have an experimental MS2 peak list (m/z, intensity pairs) from metabolomics LC-MS/MS data and need reference spectra for candidate metabolites plus a per-pair spectral similarity score for annotation.

## When NOT to use

- Input data is MS1-only (no MS2 peak list), since the CMM 3.0 service performs spectral comparison from MS2 data specifically.
- The goal is filtering/scoring candidate compound lists by experimental setup and compound properties — use CMM's expert-system filtering step instead of spectral comparison.
- The task is spectral quality assessment (deciding whether a spectrum is reliable for compound identification), which is a separate CMM 3.0 service.

## Inputs

- Experimental MS2 peak list file (m/z, intensity pairs)
- Candidate metabolite identifiers for which MassBank reference spectra are needed

## Outputs

- spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score (one row per experimental-reference spectrum pair, scores in [0, 1])

## How to apply

Load the experimental MS2 peak list as (m/z, intensity) pairs with pandas from the input peak-list file. Retrieve the matching reference MS2 spectra from public MassBank records, resolved through CMM 3.0's spectral-comparison service, which indexes experimental metabolite spectra (CMM 3.0 covers 332,665 experimental metabolites) and is also exposed via a RESTful API integrated into HMDB. Pair experimental and reference peaks using an m/z tolerance window so only corresponding fragments are matched. Compute the cosine-similarity-based score in [0, 1] between each experimental spectrum and its MassBank reference spectrum as offered by CMM 3.0's MS2 spectral-comparison service. Write the per-pair scores to spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the MS2 spectral-comparison service that resolves MassBank reference spectra and computes the cosine-similarity score per spectrum pair) — albertogilf/ceuMassMediator

## Evaluation signals

- All similarity scores in spectral_similarity_scores.csv fall within [0, 1].
- The output CSV contains columns for pair identifiers and the similarity score, with one row per experimental-reference spectrum pair.
- Only peaks within the m/z tolerance window are paired, so each matched fragment pair corresponds across experimental and reference spectra.
- Each retrieved reference spectrum is a public MassBank record resolvable through the CMM 3.0 spectral-comparison service.
- The number of scored pairs matches the number of candidate metabolites for which reference spectra were retrieved.

## Limitations

- Deploying/local CMM instances require access to an internal CEU database, which must be requested by email, plus configuration of a data-source on the app server with provided credentials (dependency: javax.faces-2.2.12.jar).
- Reference spectra coverage is limited to metabolites indexed by CMM 3.0 (332,665 experimental metabolites); oxidized lipids come from an in-house CMM library.
- Scores depend on the m/z tolerance window used for fragment pairing; overly wide or narrow windows change which fragments are paired and thus the cosine score.

## Evidence
