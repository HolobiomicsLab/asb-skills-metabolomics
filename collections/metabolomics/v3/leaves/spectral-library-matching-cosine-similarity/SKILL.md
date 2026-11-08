---
name: spectral-library-matching-cosine-similarity
description: Use when consensus spectra from GC×GC–TOFMS peak picking (e.g. GcDUO
  $cor_spectra) must be assigned compound identities against an MSP library, with
  a cosine-similarity cutoff and optional RI window.
when_to_use_negative:
- The goal is peak detection or deconvolution — annotation requires already-extracted
  consensus spectra from procesData; raw 4D arrays must go through ROI selection first.
- No suitable spectral library exists for the sample matrix (e.g. a compound missing
  from the in-house library, as with one fruitybeer compound) — matching cannot recover
  identities absent from the reference.
- The data are LC-MS or other non-GC×GC–MS acquisitions whose spectra and RI semantics
  do not match the MSP/EI-fragment assumptions of this workflow.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: Provides the duoID annotation module that performs cosine-similarity spectral
    matching with match_cutoff and RIrange parameters
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and the annotation workflow (getMSP,
    spectramatrix, duoID) is executed
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:dd08d383ffc6ff7e5ac1807c516934f84aff523c5d1a9675650c868ad22469f1
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spectral-library-matching-cosine-similarity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate GC × GC–MS features by cosine-similarity matching of consensus spectra against an in-house MSP spectral library, optionally constrained by retention-index tolerance, as implemented in GcDUO's duoID module.

## When to use

Use when consensus spectra from GC×GC–TOFMS peak picking (e.g. GcDUO $cor_spectra) must be assigned compound identities against an MSP library, with a cosine-similarity cutoff and optional RI window.

## When NOT to use

- The goal is peak detection or deconvolution — annotation requires already-extracted consensus spectra from procesData; raw 4D arrays must go through ROI selection first.
- No suitable spectral library exists for the sample matrix (e.g. a compound missing from the in-house library, as with one fruitybeer compound) — matching cannot recover identities absent from the reference.
- The data are LC-MS or other non-GC×GC–MS acquisitions whose spectra and RI semantics do not match the MSP/EI-fragment assumptions of this workflow.

## Inputs

- DuoResults object from GcDuo::procesData containing $cor_spectra consensus spectra of features common across samples
- In-house spectral library in MSP format (compound name, exactmass, formula, inchikey, ri fields plus spectra)
- Optional RT–RI alkane table for Kovats retention-index filtering (RI argument)

## Outputs

- DuoResults with $id_peak: compound annotations for each detected feature
- List of detected-but-unannotated peaks (cosine similarity ≤ 0.7) for manual spectral inspection

## How to apply

Import the in-house MSP library with getMSP, build the spectra matrix with spectramatrix, and assemble the lib list containing a compound dictionary (name, exactmass, formula, inchikey, ri) and the spectra matrix. Run GcDuo::duoID on the DuoResults object with match_cutoff = 0.7 so that only library compounds with cosine similarity above 0.7 return an annotation; leave RI = NULL when no Kovats alkane series is available and set RIrange = 30 to accept ±30 retention-index variability. Compounds whose best match falls below the cutoff stay unannotated even if the peak was detected — in the fruitybeer dataset three such peaks showed characteristic fragments of the expected compounds but no library entry exceeded 0.7. Note that library completeness limits coverage: 16 of 33 in-house fruitybeer library entries lacked a retention index, so RI filtering cannot be applied universally.

## Related tools

- **GcDUO** (Provides the duoID annotation module that performs cosine-similarity spectral matching with match_cutoff and RIrange parameters) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and the annotation workflow (getMSP, spectramatrix, duoID) is executed) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = NULL, RIrange = 30)
```

## Evaluation signals

- In the fruitybeer reproduction, ~85% of the 27 author-reported peaks receive annotations with the expected compound present in matched lists; the unannotated subset should correspond to matches with cosine similarity below 0.7.
- All returned $id_peak annotations must have best-match cosine similarity strictly above the match_cutoff of 0.7 — verify no annotation below cutoff is emitted.
- When RI = NULL, no retention-index filtering is applied; when RI is supplied, retained matches must fall within RIrange = 30 of the library ri value.
- The lib list must be well-formed: compounds lacking exactmass, formula, inchikey, or ri fields appear as NA in libs_dic rather than errors, matching the getMSP/spectramatrix dictionary construction.
- Detected-but-unannotated peaks should be cross-checked manually: spectra should show characteristic fragments of the suspected compound even when cosine similarity ≤ 0.7.

## Limitations

- Library dependence: annotation fails entirely for compounds absent from the in-house MSP library (one fruitybeer compound was missing).
- Three fruitybeer peaks were detected but unannotated because no library compound reached cosine similarity > 0.7, despite spectra showing characteristic fragments of the expected compounds.
- RI-based filtering is limited by library coverage: 16 of 33 in-house library entries lacked a retention index.
- GcDUO did not return a result when no library compound exceeded the 0.7 cosine-similarity threshold — borderline matches are silently dropped rather than flagged.

## Evidence

- [results] RI availability in library: [evidence span withheld — anchor status: not_found]
