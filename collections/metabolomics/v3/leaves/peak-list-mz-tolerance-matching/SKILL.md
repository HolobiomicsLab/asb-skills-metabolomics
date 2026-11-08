---
name: peak-list-mz-tolerance-matching
description: Use when an experimental MS2 peak list must be compared against database
  reference spectra (e.g., MassBank records) to score spectral similarity for metabolite
  annotation.
when_to_use_negative:
- Input is MS1-level m/z data only — CMM 3.0's spectral-comparison service targets
  MS2 data specifically.
- The goal is candidate filtering/scoring of database query results by compound properties
  rather than spectrum-vs-spectrum comparison — CMM 2.0-style expert-system filtering
  is the wrong skill.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the MS2 spectral-comparison service that indexes experimental metabolite
    spectra and computes the similarity score between the experimental peak list and
    reference spectra
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:8b5d6fc950f8c5a285031b7b59844c7b6c1792f05d8ca02084bb94c3453f534e
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
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# peak-list-mz-tolerance-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pairs an experimental MS2 peak list (m/z, intensity pairs) against reference spectra retrieved via CEU Mass Mediator (CMM) 3.0's MS2 spectral-comparison service, matching only corresponding fragments within an m/z tolerance window and scoring each pair with a cosine-similarity-based score in [0, 1].

## When to use

Use when an experimental MS2 peak list must be compared against database reference spectra (e.g., MassBank records) to score spectral similarity for metabolite annotation.

## When NOT to use

- Input is MS1-level m/z data only — CMM 3.0's spectral-comparison service targets MS2 data specifically.
- The goal is candidate filtering/scoring of database query results by compound properties rather than spectrum-vs-spectrum comparison — CMM 2.0-style expert-system filtering is the wrong skill.

## Inputs

- experimental MS2 peak list file (m/z, intensity pairs)
- reference MS2 spectra from public MassBank records (resolved through the CMM 3.0 spectral-comparison service)

## Outputs

- spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score (cosine-similarity-based, in [0, 1])

## How to apply

Load the experimental MS2 peak list of m/z and intensity pairs with pandas from the input peak-list file. Retrieve the matching reference MS2 spectra from public MassBank records, resolved through CMM 3.0's MS2 spectral-comparison service (which indexes experimental metabolite spectra — 332,665 experimental metabolites in version 3.0). Match experimental and reference peaks using an m/z tolerance window so that only corresponding fragments are paired. Compute a cosine-similarity-based score in [0, 1] between each experimental spectrum and its reference spectrum, as offered by CMM 3.0's spectral-comparison service. Write the per-pair scores to spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the MS2 spectral-comparison service that indexes experimental metabolite spectra and computes the similarity score between the experimental peak list and reference spectra) — https://github.com/albertogilf/ceuMassMediator

## Examples

```
import pandas as pd; peaks = pd.read_csv('ms2_peaks.csv'); refs = fetch_massbank_via_cmm3(peaks); scores = cosine_similarity_within_mz_tolerance(peaks, refs); scores.to_csv('spectral_similarity_scores.csv', index=False)
```

## Evaluation signals

- Every emitted similarity score falls within [0, 1], as the cosine-similarity-based score is bounded in that range.
- Only peak pairs whose m/z values match within the tolerance window are paired; no cross-matched fragments outside the tolerance window should appear.
- Output file spectral_similarity_scores.csv contains one row per experimental/reference spectrum pair, with pair-identifier columns plus a similarity-score column.
- Each experimental spectrum is scored against the MassBank record retrieved via CMM 3.0, not against an unrelated reference.
- The reference spectra resolved are drawn from CMM 3.0's indexed experimental metabolite spectra (MassBank records), not from unverified local libraries.

## Limitations

- The score quality depends on reference-spectrum reliability; CMM 3.0 separately offers a spectral quality-assessment service to determine the reliability of a spectrum for compound identification purposes.
- Access to the CMM backend requires access to an internal CEU database (request access via <maintainer e-mail, see the repository README>) and configuration of a data-source on the app server with provided credentials.
- CMM 3.0's coverage is limited to its indexed experimental (332,665) and predicted (681,198) metabolites; compounds outside these sources (except the added in-house oxidized-lipid library) cannot be compared.

## Evidence
