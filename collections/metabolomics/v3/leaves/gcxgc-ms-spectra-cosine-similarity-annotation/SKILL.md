---
name: gcxgc-ms-spectra-cosine-similarity-annotation
description: Use when Module III (deconvolution) has produced consensus spectra ($cor_spectra
  in DuoResults) for GC × GC–TOFMS features and you need compound identifications
  from an in-house MSP library before PARAFAC2 data integration.
when_to_use_negative:
- Input is still raw CDF data — annotation must wait until after Module II ROI selection
  and Module III deconvolution have produced consensus spectra
- The compounds of interest are absent from the available MSP library — the paper
  reports features with no library match above cosine similarity 0.7 receive no annotation
- Only small chromatographic signals with too few characteristic fragments are present
  — these cannot be deconvoluted into consensus spectra suitable for library matching
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package providing duoID() for cosine-similarity annotation and getMSP/spectramatrix/rt2ri/getRI
    helpers for library import and RI refinement
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Environment in which the annotation workflow (duoID, library dictionary assembly,
    CSV export) is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:d8b4942812f3f372ad2fab1e6fc741f0cebae6150f24e9c2019fb7f4b73a215e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - GcDUO
  - R
  - duoID
  - getMSP
  - getRI
  - id_rev
  - libID
  - rt2ri
  - spectramatrix
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 25
---

# gcxgc-ms-spectra-cosine-similarity-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate GC × GC–MS consensus spectra (GcDUO Module IV) by matching them against an imported MSP spectral library using cosine similarity, with optional Kovats retention-index refinement, accepting identifications only above a user-defined match_cutoff (e.g. 0.7).

## When to use

Use when Module III (deconvolution) has produced consensus spectra ($cor_spectra in DuoResults) for GC × GC–TOFMS features and you need compound identifications from an in-house MSP library before PARAFAC2 data integration.

## When NOT to use

- Input is still raw CDF data — annotation must wait until after Module II ROI selection and Module III deconvolution have produced consensus spectra
- The compounds of interest are absent from the available MSP library — the paper reports features with no library match above cosine similarity 0.7 receive no annotation
- Only small chromatographic signals with too few characteristic fragments are present — these cannot be deconvoluted into consensus spectra suitable for library matching

## Inputs

- DuoResults object from GcDUO Module III containing $cor_spectra (consensus spectra of common features between samples)
- In-house MSP spectral library file (imported via getMSP)
- Optional: Kovats RT–RI alkane table for retention-index refinement

## Outputs

- DuoResults object updated with $id_peak — compound annotations for each feature passing the match_cutoff
- annotated_peak_table.csv (features with no cosine similarity above the cutoff are left unannotated)

## How to apply

Import the in-house MSP library with getMSP() and build the spectra matrix with spectramatrix(), then assemble a library dictionary (compound name, exact mass, formula, InChIKey, and Kovats RI when available) as required by the annotation function. Run duoID() on the Module III DuoResults object with match_cutoff set to the user threshold (0.7 in the paper's fruitybeer validation), so identifications are returned only where the cosine similarity between the consensus spectrum and a library spectrum exceeds the cutoff. If retention-index data are available, supply the RT–RI alkane values via the RI argument and set RIrange (e.g. 30) so RI-based refinement is applied through rt2ri/getRI. Extract the resulting $id_peak annotations per feature; features below the cutoff receive no identification, as in the fruitybeer case where no library compound exceeded 0.7.

## Related tools

- **GcDUO** (R package providing duoID() for cosine-similarity annotation and getMSP/spectramatrix/rt2ri/getRI helpers for library import and RI refinement) — https://github.com/mariallr/GcDuo
- **R** (Environment in which the annotation workflow (duoID, library dictionary assembly, CSV export) is executed)

## Examples

```
DuoResults <- GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = NULL, RIrange = 30)
```

## Evaluation signals

- Every returned $id_peak annotation corresponds to a library compound whose cosine similarity with the feature's consensus spectrum exceeds match_cutoff (0.7 in the paper)
- Features with all library cosine similarities ≤ 0.7 remain unannotated (no false identifications returned), mirroring the fruitybeer validation
- $id_peak covers every feature in $ind_peak (either an annotation or NA/empty), with no dropouts
- When RI refinement is enabled, accepted matches also fall within RIrange (e.g. ±30 RI units) of the library Kovats RI
- Spot-check a peak with SpectraView() against the library spectrum to confirm the match spectrum matches the feature's consensus spectrum

## Limitations

- Annotation quality depends entirely on the in-house MSP library; in the fruitybeer validation one compound was missing from the library and three detected peaks had no library compound above cosine similarity 0.7, so GcDUO returned no result despite spectra showing characteristic fragments of the expected compounds
- GcDUO is implemented in R, which is not optimized for large memory loads or parallel processing across multiple cores; high-performance computing environments may be needed
- No software version for GcDUO/R is stated in the article

## Evidence
