---
name: molecular-mass-filtering
description: Use when performing library matching of an EI mass spectrum against a
  large reference library (e.g., the NIST 2017 library) and the molecular mass of
  the query sample is known with reasonable accuracy.
when_to_use_negative:
- The molecular mass of the query sample cannot be estimated with some accuracy, so
  the filter cannot prune candidates reliably
- The reference library is already small enough that filtering adds no benefit or
  risks excluding the true candidate beyond the tolerance window
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NEIMS
  role: Provides the library-matching search over the augmented spectral library with
    the mass filtering option applied before similarity computation
  repo: github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:c37045f3914edb35e3a519bc838df5a48c9cde74fc950b462772f0fa6fa46dfe
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Vizier
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-mass-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Restricts the spectral reference library to candidates whose molecular mass is within a tolerance (5 Da) of the query molecule's mass before cosine-similarity library matching, as used in NEIMS library matching. This shrinks the candidate space (median 6696 spectra per query) and improves recall@10.

## When to use

Use when performing library matching of an EI mass spectrum against a large reference library (e.g., the NIST 2017 library) and the molecular mass of the query sample is known with reasonable accuracy.

## When NOT to use

- The molecular mass of the query sample cannot be estimated with some accuracy, so the filter cannot prune candidates reliably
- The reference library is already small enough that filtering adds no benefit or risks excluding the true candidate beyond the tolerance window

## Inputs

- Query electron-ionization mass spectrum with known molecular mass of the query molecule
- Reference spectral library (e.g., NIST 2017 Mass Spectral Main Library, optionally augmented with NEIMS-predicted spectra)
- Mass tolerance parameter (5 Da in the NEIMS evaluation)

## Outputs

- Filtered candidate library (median 6696 spectra per query at 5 Da tolerance)
- Ranked list of candidate spectra by weighted cosine similarity to the query
- Recall@10 (rank ≤ 10) accuracy over the query set

## How to apply

During the library match search, enable the mass filtering option so the library only includes spectra from molecular candidates whose molecular mass differs from the query molecule's mass by no more than a set tolerance. In the NEIMS evaluation this tolerance was set to 5 Da of the query molecule's mass, which reduced the search space to a median of 6696 spectra per query molecule and contributed an additional ~6 percentage points of recall@10 on top of the 91.8% baseline. Match the filtered library using the weighted cosine similarity between the query spectrum and each candidate spectrum, then record the rank of the correct spectrum. Apply this filter after constructing the augmented reference library (observed spectra plus NEIMS predictions) and before computing similarities; it is appropriate whenever precursor or molecular mass can be measured for the sample.

## Related tools

- **NEIMS** (Provides the library-matching search over the augmented spectral library with the mass filtering option applied before similarity computation) — github.com/brain-research/deep-molecular-massspec

## Evaluation signals

- Filtered library size should drop to a median of ~6696 spectra per query molecule at a 5 Da tolerance
- Recall@10 on the held-out test half should reach ~91.7–91.8% with mass filtering, ~6 percentage points above matching without the filter
- Every retained candidate spectrum must have a molecular mass within ±5 Da of the query molecule's mass
- Correct spectrum ranks must be recomputable by weighted cosine similarity (eq 1) over the filtered library

## Limitations

- Mass filtering is only applicable for experimental setups where the molecular mass of the sample can be known with some accuracy
- If the true candidate's mass differs from the measured query mass by more than the tolerance (5 Da), correct identification becomes impossible

## Evidence

- [readme] Task framing: match against augmented library: [evidence span withheld — anchor status: external_doc]
