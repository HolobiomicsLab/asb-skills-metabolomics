---
name: ms2-spectral-similarity-scoring
description: Use when you have an experimental MS2 peak list (m/z, intensity pairs)
  and need a per-pair similarity score against reference metabolite spectra to support
  compound identification.
when_to_use_negative:
- The input is raw uncentroided LC-MS profile data rather than a peak list of (m/z,
  intensity) pairs.
- The goal is candidate filtering/scoring from an m/z query without MS2 data — use
  CMM's expert-system filtering of query results instead.
- You need to assess the reliability/quality of a spectrum for compound identification
  rather than compare two spectra — use CMM 3.0's spectral quality-assessment service.
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the MS2 spectral-comparison service that pairs experimental and reference
    peaks within an m/z tolerance window and yields a cosine-similarity score per
    spectrum pair.
  repo: https://github.com/albertogilf/ceuMassMediator
- name: MassBank
  role: Source of the reference MS2 spectra that each experimental spectrum is scored
    against.
- name: pandas
  role: Used to load the experimental MS2 peak list (m/z, intensity pairs) from the
    input peak-list file.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:209ef3f634d5c3e708401fa9a409e9b97da67e2e835b36a9ac9ebfaa053e04db
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
  - MassBank
  - R
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ms2-spectral-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a spectral-similarity score in [0, 1] between an experimental MS2 peak list (m/z, intensity pairs) and reference spectra from indexed metabolite databases, using CMM 3.0's spectral-comparison service. This is CMM 3.0's dedicated MS2-based support for metabolite annotation and identification.

## When to use

Use when you have an experimental MS2 peak list (m/z, intensity pairs) and need a per-pair similarity score against reference metabolite spectra to support compound identification.

## When NOT to use

- The input is raw uncentroided LC-MS profile data rather than a peak list of (m/z, intensity) pairs.
- The goal is candidate filtering/scoring from an m/z query without MS2 data — use CMM's expert-system filtering of query results instead.
- You need to assess the reliability/quality of a spectrum for compound identification rather than compare two spectra — use CMM 3.0's spectral quality-assessment service.

## Inputs

- Experimental MS2 peak list file (m/z, intensity pairs)
- Reference MS2 spectra from public MassBank records resolved via the CMM 3.0 spectral-comparison service

## Outputs

- spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score

## How to apply

Load the experimental MS2 peak list (m/z, intensity pairs) with pandas from the input peak-list file. Retrieve the matching reference MS2 spectra from public MassBank records resolved through CMM 3.0's spectral-comparison service, which indexes experimental metabolite spectra. Match experimental and reference peaks using an m/z tolerance window so only corresponding fragments are paired. Compute a cosine-similarity-based score in [0, 1] between each experimental spectrum and its MassBank reference spectrum, as offered by CMM 3.0's MS2 spectral-comparison service. Write the per-pair scores to spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the MS2 spectral-comparison service that pairs experimental and reference peaks within an m/z tolerance window and yields a cosine-similarity score per spectrum pair.) — https://github.com/albertogilf/ceuMassMediator
- **MassBank** (Source of the reference MS2 spectra that each experimental spectrum is scored against.)
- **pandas** (Used to load the experimental MS2 peak list (m/z, intensity pairs) from the input peak-list file.)

## Examples

```
import pandas as pd; peaks = pd.read_csv('ms2_peak_list.csv'); scores = cmm_ms2_compare(peaks, refs='MassBank', tolerance_mz=0.01); scores.to_csv('spectral_similarity_scores.csv', index=False)
```

## Evaluation signals

- Every computed similarity score falls within [0, 1].
- Only fragments paired within the m/z tolerance window contribute to each score — peaks outside the tolerance are not matched.
- Output file spectral_similarity_scores.csv exists and contains columns for the pair identifiers and the similarity score, with one row per experimental/reference pair.
- Each experimental spectrum has been scored against a resolved MassBank reference spectrum retrieved through the CMM 3.0 service.
- Identical or near-identical experimental/reference spectra yield scores near 1, as expected from a cosine-similarity-based metric.

## Limitations

- Scores depend on reference spectra availability in the indexed databases; CMM 3.0's coverage is bounded by its indexed experimental metabolites (332,665 experimental metabolites).
- Deployment of CMM requires access to an internal CEU database and a configured app-server data-source, so the service cannot be trivially re-hosted.
- The article abstract does not specify the exact m/z tolerance value; the tolerance window must be chosen for the instrument's mass accuracy.

## Evidence
