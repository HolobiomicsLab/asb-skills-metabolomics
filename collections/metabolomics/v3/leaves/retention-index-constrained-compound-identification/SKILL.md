---
name: retention-index-constrained-compound-identification
description: Use when a GcDUO DuoResults object contains detected peak spectra and
  an MSP spectral library (optionally with Kovats RI values) is available, and each
  feature must be assigned a compound identity with a confidence threshold.
when_to_use_negative:
- Input is an unprocessed 4D CDF array or chromatogram visualization task — peak picking
  (procesData) must run before annotation is possible
- No spectral library is available or the target compounds are outside the library's
  coverage — annotations will simply not be returned (one fruitybeer compound was
  missing from the in-house library)
- Small chromatographic signals with too few characteristic fragments to deconvolute
  — cosine matching will not yield reliable identities
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: Provides getMSP, spectramatrix, and duoID for MSP library import, spectral
    matching, and RI-constrained annotation
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform in which the annotation workflow is executed via devtools-installed
    GcDUO package
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:0f52a48b73e461377099f52b69cd86f90fc2410df65f113dd0193494768d00aa
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
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Retention-index-constrained compound identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate GcDUO-detected GC×GC–MS features by matching consensus spectra against an in-house MSP library with a cosine-similarity cutoff of 0.7, optionally tightening matches with Kovats retention indices (RIrange = 30) when alkane RT–RI calibration is available.

## When to use

Use when a GcDUO DuoResults object contains detected peak spectra and an MSP spectral library (optionally with Kovats RI values) is available, and each feature must be assigned a compound identity with a confidence threshold.

## When NOT to use

- Input is an unprocessed 4D CDF array or chromatogram visualization task — peak picking (procesData) must run before annotation is possible
- No spectral library is available or the target compounds are outside the library's coverage — annotations will simply not be returned (one fruitybeer compound was missing from the in-house library)
- Small chromatographic signals with too few characteristic fragments to deconvolute — cosine matching will not yield reliable identities

## Inputs

- DuoResults object from GcDuo::procesData containing $ind_peak, $spec_ind, and $cor_spectra
- In-house MSP spectral library file (compound names, exact masses, formulas, InChIKeys, and optional Kovats RI values)
- Optional RT–RI alkane series table for retention-index calibration (RI argument; NULL if unused)

## Outputs

- DuoResults object updated with $id_peak: per-feature compound annotations
- libs_dic compound dictionary data.frame (comp, exactmass, formula, inchikey, ri)

## How to apply

First import the library into R: read the MSP with getMSP(msp_file), build a spectra matrix with spectramatrix(libs), and assemble a compound dictionary (comp, exactmass, formula, inchikey, ri) into lib <- list(data = libs_dic, spectra_matrix = spectramatrix_libs). Then run GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = NULL, RIrange = 30), where match_cutoff is the minimum cosine similarity to return an annotation and RI supplies the RT–RI alkane series only if Kovats indices are used; RIrange = 30 defines the accepted RI variability. The function fills DuoResults$id_peak with annotations per feature. Decision points: if no library compound exceeds cosine similarity 0.7 no annotation is returned (as seen for three fruitybeer peaks whose spectra nonetheless matched expected fragments), and compounds absent from the library cannot be identified; RI=NULL is appropriate for the fruitybeer workflow where no alkane calibration is applied.

## Related tools

- **GcDUO** (Provides getMSP, spectramatrix, and duoID for MSP library import, spectral matching, and RI-constrained annotation) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform in which the annotation workflow is executed via devtools-installed GcDUO package)

## Examples

```
DuoResults <- GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = NULL, RIrange = 30)
```

## Evaluation signals

- Every returned annotation has cosine similarity > 0.7 (match_cutoff); features with no library compound above 0.7 remain unannotated rather than force-matched
- When Kovats RI is supplied, accepted matches fall within RIrange = 30 RI units of the library value
- libs_dic contains one row per library compound with correctly populated comp/exactmass/formula/inchikey/ri columns (NA when absent in the MSP)
- Spectra of unannotated peaks can be manually inspected (e.g. SpectraView) and should still show characteristic fragments of the expected compound class when library coverage, not matching, is the limiting factor

## Limitations

- Three peaks in the fruitybeer dataset were detected but not annotated because no library compound exceeded cosine similarity 0.7, even though their spectra showed characteristic fragments of the expected compounds — the cutoff trades recall for specificity
- Annotation is limited to compounds present in the imported in-house MSP library; only one fruitybeer compound was missing from the library, illustrating coverage dependence
- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted or reliably matched; decane was only detected when the signal-to-noise parameter was reduced from 3 to 1

## Evidence
