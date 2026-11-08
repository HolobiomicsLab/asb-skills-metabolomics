---
name: cosine-similarity-peak-vector-comparison
description: Use when you have an experimental MS2 peak list and need per-compound
  spectral similarity scores against database reference spectra (e.g., MassBank) for
  metabolite annotation via CMM 3.0.
when_to_use_negative:
- Input is raw instrument data (e.g., mzML/NetCDF) rather than an already peak-picked
  MS2 m/z–intensity list
- The task is spectral quality assessment rather than comparison — CMM 3.0 provides
  a separate spectral quality-assessment service for judging spectrum reliability
  for compound identification
- The query target is annotation filtering/scoring by experimental setup and compound
  properties, which CMM 3.0 handles through its expert-system filter rather than MS2
  spectral comparison
edam_operation: http://edamontology.org/operation_3254
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Provides the MS2 spectral-comparison service that retrieves MassBank reference
    spectra and computes the cosine-similarity-based score against the experimental
    peak list
  repo: https://github.com/albertogilf/ceuMassMediator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:4d738c739de020e2421dcd241a6ac19370f99f7b46fc066bd319e441058c408e
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
  edam_operation: http://edamontology.org/operation_3254
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cosine-similarity-peak-vector-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares an experimental MS2 peak list (m/z, intensity pairs) against MassBank reference spectra using the CMM 3.0 MS2 spectral-comparison service, yielding a cosine-similarity-based score per spectrum pair to support metabolite annotation.

## When to use

Use when you have an experimental MS2 peak list and need per-compound spectral similarity scores against database reference spectra (e.g., MassBank) for metabolite annotation via CMM 3.0.

## When NOT to use

- Input is raw instrument data (e.g., mzML/NetCDF) rather than an already peak-picked MS2 m/z–intensity list
- The task is spectral quality assessment rather than comparison — CMM 3.0 provides a separate spectral quality-assessment service for judging spectrum reliability for compound identification
- The query target is annotation filtering/scoring by experimental setup and compound properties, which CMM 3.0 handles through its expert-system filter rather than MS2 spectral comparison

## Inputs

- Experimental MS2 peak list (m/z, intensity pairs) as a peak-list file loadable with pandas
- Reference MS2 spectra from MassBank records retrieved via the CMM 3.0 spectral-comparison service

## Outputs

- spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score (one cosine-based score in [0, 1] per experimental–reference spectrum pair)

## How to apply

Load the experimental MS2 peak list (m/z, intensity pairs) with pandas from the input peak-list file. Retrieve the matching reference MS2 spectra from public MassBank records resolved through the CMM 3.0 spectral-comparison service, which indexes experimental metabolite spectra (332,665 experimental metabolites in CMM 3.0). Pair experimental and reference peaks only within an m/z tolerance window so corresponding fragments are matched. Compute a cosine-similarity-based score in [0, 1] between each experimental spectrum and its MassBank reference spectrum, as offered by CMM 3.0's MS2 spectral-comparison service. Write the per-pair scores to spectral_similarity_scores.csv with columns for the pair identifiers and the similarity score.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Provides the MS2 spectral-comparison service that retrieves MassBank reference spectra and computes the cosine-similarity-based score against the experimental peak list) — https://github.com/albertogilf/ceuMassMediator

## Examples

```
score = float(exp_int @ ref_int) / (np.linalg.norm(exp_int) * np.linalg.norm(ref_int))  # cosine over m/z-matched fragment intensities -> spectral_similarity_scores.csv
```

## Evaluation signals

- Every experimental–reference spectrum pair yields exactly one similarity score bounded in [0, 1]
- Only fragment peaks within the specified m/z tolerance window are paired between the experimental and reference spectra
- Output file spectral_similarity_scores.csv contains the required schema: pair identifier columns plus the similarity score column
- Peak lists loaded as m/z, intensity pairs retain m/z as the matching key and intensity as the vector component used in the cosine computation
- Reference spectra must resolve to public MassBank records via the CMM 3.0 service — pairs with no retrievable MassBank record should be flagged, not scored silently

## Limitations

- The article describes the spectral-comparison service only at abstract level; concrete m/z tolerance values and score thresholds are not specified in the provided context
- Self-deploying CMM requires access to an internal CEU database (requested via email) and a configured app-server data-source with provided credentials
- Deployment additionally requires the javax.faces-2.2.12.jar library on the app server

## Evidence

- [abstract] CMM 3.0 offers a service for spectral comparison from MS2 data: "a service for spectral comparison from MS 2 data"
