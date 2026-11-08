---
name: consensus-spectra-library-matching
description: Use when consensus spectra have been obtained from GC × GC–MS deconvolution
  (e.g. GcDUO Module III DuoResults with $cor_spectra) and each peak feature must
  be assigned a compound identity against an MSP library.
when_to_use_negative:
- Input is raw CDF/GC×GC–MS data that still needs peak picking and deconvolution (run
  GcDUO Modules I–III, e.g. procesData, first)
- The target compound is absent from the library or its spectrum is too small to deconvolute
  — as in the fruitybeer dataset where one compound was missing from the in-house
  library and small signals yielded no annotation
- Data already contain a full annotated feature table (annotation should precede Module
  V data integration, not follow it)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GcDUO
  role: R package providing duoID(), getMSP(), spectramatrix(), getRI, rt2ri, and
    libID for MSP library import and cosine-similarity annotation of consensus spectra
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: runtime environment in which GcDUO (installed via devtools::install_github('mariallr/GcDUO'))
    executes the annotation step
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:a38389c0edf566767cb7968df81dd253f22197f165e6d9fdc5c67e2847d0b70b
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
  - library_import
  - rt2ri
  - spectramatrix
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# consensus-spectra-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates GC × GC–MS consensus spectra (GcDUO Module IV) by matching them against an in-house MSP spectral library using cosine similarity, with optional retention-index (Kovats) refinement. Identifications are accepted only when similarity exceeds a user-defined match_cutoff (e.g. 0.7).

## When to use

Use when consensus spectra have been obtained from GC × GC–MS deconvolution (e.g. GcDUO Module III DuoResults with $cor_spectra) and each peak feature must be assigned a compound identity against an MSP library.

## When NOT to use

- Input is raw CDF/GC×GC–MS data that still needs peak picking and deconvolution (run GcDUO Modules I–III, e.g. procesData, first)
- The target compound is absent from the library or its spectrum is too small to deconvolute — as in the fruitybeer dataset where one compound was missing from the in-house library and small signals yielded no annotation
- Data already contain a full annotated feature table (annotation should precede Module V data integration, not follow it)

## Inputs

- GcDUO Module III object (DuoResults) containing $cor_spectra consensus spectra and $ind_peak/$spec_ind feature tables
- In-house MSP spectral library file (msp_file) with compound name, exactmass, formula, inchikey, and optional ri fields
- Kovats RT–RI alkane values (optional, passed as RI argument)

## Outputs

- DuoResults object updated with $id_peak — compound annotations for each feature
- annotated_peak_table.csv (per-feature identifications)

## How to apply

Import the in-house MSP library into R with GcDUO's getMSP() and build the spectral matrix with spectramatrix(), then assemble a library dictionary of compound name, exact mass, formula, InChIKey, and retention index (NA where absent). Run duoID() on the Module III consensus spectra object with match_cutoff set to the user threshold (0.7 in the paper's fruitybeer test), passing the library as a list with 'data' and 'spectra_matrix' slots. When Kovats retention-index data exist, supply the RT–RI alkane values via the RI argument and set RIrange (e.g. 30) so GcDUO applies RI-based refinement through rt2ri/getRI. Extract $id_peak from the returned DuoResults object to obtain per-feature annotations. The rationale: identification is accepted only when the computed cosine similarity between the consensus spectrum and a library spectrum exceeds the cutoff, mirroring the paper's fruitybeer case where no compound exceeded 0.7 and no annotation was returned.

## Related tools

- **GcDUO** (R package providing duoID(), getMSP(), spectramatrix(), getRI, rt2ri, and libID for MSP library import and cosine-similarity annotation of consensus spectra) — https://github.com/mariallr/GcDuo
- **R** (runtime environment in which GcDUO (installed via devtools::install_github('mariallr/GcDUO')) executes the annotation step) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = NULL, RIrange = 30)
```

## Evaluation signals

- Every accepted annotation corresponds to a library compound whose cosine similarity exceeds match_cutoff (e.g. > 0.7)
- Features with similarity ≤ 0.7 receive no identification in $id_peak — as verified in the fruitybeer dataset where none of the library compounds exceeded 0.7 and GcDUO returned no result
- When RI refinement is enabled, annotations are consistent with the Kovats RT–RI alkane mapping within RIrange (e.g. 30)
- The library dictionary contains complete compound name, exact mass, formula, and InChIKey fields (NA where absent in the MSP)
- Annotations exist for each feature passed from Module III and are available prior to the constrained PARAFAC2 data-integration step

## Limitations

- Compounds absent from the in-house MSP library cannot be annotated (the fruitybeer test had one compound missing from the library)
- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted into consensus spectra and therefore cannot be annotated
- No version number for GcDUO or its PARAFAC/PARAFAC2 implementations is documented, and R is not optimized for large memory loads, which may limit batch annotation scale

## Evidence
