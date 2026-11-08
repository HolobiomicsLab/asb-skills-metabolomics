---
name: gc-ms-spectral-alignment-across-samples
description: Use when deconvolved GC/MS metabolomics data from multiple samples must
  be cross-sample aligned to build a compound × sample quantification table, especially
  when retention indices are unavailable and RT drift plus spectral similarity suffice.
when_to_use_negative:
- Input is raw GC/MS profile data (e.g. NetCDF from MTBLS321) that has not yet been
  deconvolved — run the deconvolution step first
- The task is alignment of fragment-ion peaks by m/z and RT window (peak-picking style,
  e.g. XCMS centWave) rather than alignment of deconvolved compound spectra
- AMDIS-style independent per-sample identification without cross-sample spectral
  alignment is acceptable
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: erah
  role: R package providing setAlPar, alignComp, and alignList to align deconvolved
    compound spectra across samples
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open-source language in which the eRah alignment workflow is implemented
- name: GC-TOF MS
  role: Instrument whose profile data (e.g. serum samples, MTBLS321) supply the deconvolved
    spectra being aligned
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:315396537b4db003328a7d4bdeebffd05cf3ec6c4dea31892d917f668d4b4059
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GC-TOF MS
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gc-ms-spectral-alignment-across-samples

## Summary

Aligns deconvolved GC/MS compound spectra across samples using eRah's alignComp, grouping compounds within a maximum retention time drift (3 s) and minimum spectral correlation (0.90) without requiring retention indices. Produces a per-compound, per-sample integrated area table (AlignId) for downstream quantification.

## When to use

Use when deconvolved GC/MS metabolomics data from multiple samples must be cross-sample aligned to build a compound × sample quantification table, especially when retention indices are unavailable and RT drift plus spectral similarity suffice.

## When NOT to use

- Input is raw GC/MS profile data (e.g. NetCDF from MTBLS321) that has not yet been deconvolved — run the deconvolution step first
- The task is alignment of fragment-ion peaks by m/z and RT window (peak-picking style, e.g. XCMS centWave) rather than alignment of deconvolved compound spectra
- AMDIS-style independent per-sample identification without cross-sample spectral alignment is acceptable

## Inputs

- MetaboSet experiment object from eRah deconvolveComp (one or more GC/MS chromatograms, e.g. GC-TOF MS profile data)
- Alignment parameters object created with setAlPar (min.spectra.cor, max.time.dist, mz.range)

## Outputs

- Aligned compound list via alignList(ex), one row per unique aligned compound with AlignId
- Per-sample integrated area table for each aligned compound (compound names, matching scores, integrated area per sample)
- Aligned MetaboSet object ready for missing compound recovery (recMissComp) and identification (identifyComp)

## How to apply

Load the deconvolved MetaboSet experiment object from eRah's deconvolveComp output and configure alignment parameters with setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); the mz.range restricts spectral comparison to masses 70–600 m/z. Run alignComp(ex, alParameters = ex.al.par), which computes the Euclidean distance between retention-time distance and spectral similarity so compounds appearing across the maximum number of samples with the least RT and spectral distance are grouped. The 0.90 spectral correlation cutoff ensures only highly similar deconvolved spectra merge, and the 3 s RT cap tolerates typical GC/TOF-MS drift. Rationale: unlike peak-picking tools that align fragment-ion peaks by m/z and RT window, eRah aligns compound spectra directly, avoiding the need for internal references such as retention indices. Compounds absent in some samples can later be recovered with recMissComp, which must be followed by re-identification via identifyComp.

## Related tools

- **erah** (R package providing setAlPar, alignComp, and alignList to align deconvolved compound spectra across samples) — http://CRAN.R-project.org/package=erah
- **R** (Open-source language in which the eRah alignment workflow is implemented)
- **GC-TOF MS** (Instrument whose profile data (e.g. serum samples, MTBLS321) supply the deconvolved spectra being aligned)

## Examples

```
ex.al.par <- setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); ex <- alignComp(ex, alParameters = ex.al.par); alignList(ex)
```

## Evaluation signals

- Aligned table contains one row per unique compound with an AlignId and per-sample area columns
- Each grouped compound entry satisfies the alignment criteria: min.spectra.cor = 0.90 and max.time.dist = 3 s
- Co-eluting compounds deconvolved upstream remain distinct rows rather than being erroneously merged
- Missing compounds in some samples are deferred to recMissComp, not silently dropped during alignment
- alignList(ex) returns a populated alignment list and dataList(ex) yields the final aligned data list

## Limitations

- NIST-format mass spectra libraries cannot be imported directly — eRah requires non-binary-readable compound spectra and can export spectra to MSP format instead
- Compounds absent from some samples may be lost at this stage; the optional recMissComp recovery step is needed, and re-identification with identifyComp is required afterward
- Mass comparison is restricted to the user-specified mz.range (e.g. 70:600); fragments outside this window are excluded from similarity scoring
- Alignment criteria must be tuned per dataset; overly loose max.time.dist risks merging co-eluting compounds

## Evidence

- [methods] alignment parameters: [evidence span withheld — anchor status: not_found]
- [methods] alignment criteria: [evidence span withheld — anchor status: not_found]
- [methods] minimum spectral correlation: [evidence span withheld — anchor status: not_found]
- [abstract] no retention indices needed: "(iii) alignment of mass spectra across samples"
- [methods] missing compound recovery: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [abstract] ERah outputs quantitative table: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [methods] alignment list access: [evidence span withheld — anchor status: not_found]
- [results] spectra must be non-binary: "Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format."
- [intro] eRah integrates alignment with deconvolution: [evidence span withheld — anchor status: not_found]
- [readme] eRah is freely available: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
