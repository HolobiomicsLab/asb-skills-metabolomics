---
name: chromatographic-peak-alignment-qc
description: Use when GC/MS chromatograms have already been deconvolved with eRah
  and aligned compound tables contain gaps (compounds found in some samples but not
  others) that must be filled before quantification.
when_to_use_negative:
- Raw GC/MS profile data that has not yet been deconvolved with deconvolveComp() —
  alignment requires deconvolved compound spectra
- Peak tables produced by peak-picking tools such as XCMS centWave, MetAlign, or ChromaTOF,
  which are not based on compound spectra and are outside eRah's recovery workflow
- LC-MS or NMR data — this recovery operation is designed for GC-EI MS data with co-eluting
  compounds and extensive fragmentation
edam_operation: http://edamontology.org/operation_3681
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: erah
  role: Provides alignComp() for spectral alignment across samples, recMissComp()
    for missing compound recovery via LAD regression, identifyComp()/idList() for
    post-recovery re-identification, and dataList() for table export
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which eRah is written and in which the alignment/recovery
    workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:2c07b3d281618546d49fc363ab7d64b8dabad592df2f3efe3ad8c461cdfae72b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: http://edamontology.org/operation_3681
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatographic-peak-alignment-qc

## Summary

Align eRah-deconvolved GC/MS compound spectra across samples and then recover compounds missing from some samples via LAD regression against the aligned compound's mean spectrum, producing a complete compound-by-sample area table.

## When to use

Use when GC/MS chromatograms have already been deconvolved with eRah and aligned compound tables contain gaps (compounds found in some samples but not others) that must be filled before quantification.

## When NOT to use

- Raw GC/MS profile data that has not yet been deconvolved with deconvolveComp() — alignment requires deconvolved compound spectra
- Peak tables produced by peak-picking tools such as XCMS centWave, MetAlign, or ChromaTOF, which are not based on compound spectra and are outside eRah's recovery workflow
- LC-MS or NMR data — this recovery operation is designed for GC-EI MS data with co-eluting compounds and extensive fragmentation

## Inputs

- MetaboSet object holding the aligned compound table produced by erah's alignComp()
- Deconvolution/alignment parameter objects created with setDecPar and setAlPar
- GC/MS (GC-TOF MS) raw chromatogram data underlying the deconvolution, e.g. gcspikelite spike-in samples

## Outputs

- Recovered compound-by-sample intensity/area table with no missing (zero) entries for recovered compounds, exported via dataList()
- Updated MetaboSet object requiring re-identification of compounds via identifyComp() / idList()

## How to apply

Load the MetaboSet object holding the aligned compound table produced by erah's alignComp() output, where alignment of mass spectra across samples has already been executed. Run erah's missing compound recovery with recMissComp(), setting min.samples = 6 as in the gcspikelite nine-sample spike-in demo, so that compounds found in at least that many samples are re-searched in the remaining samples. The recovery step determines a target spectrum from the mean spectrum of each aligned compound, then retrieves the compound chromatographic profile via LAD regression between the target spectrum and a window around the expected elution time in samples where the compound is missing. Because compounds must be matched against the spectral library again after recovery, re-run identification with identifyComp() after executing the missing compound recovery step. Export the final compound-name/area table via dataList().

## Related tools

- **erah** (Provides alignComp() for spectral alignment across samples, recMissComp() for missing compound recovery via LAD regression, identifyComp()/idList() for post-recovery re-identification, and dataList() for table export) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which eRah is written and in which the alignment/recovery workflow is executed)

## Examples

```
ex <- recMissComp(ex, min.samples = 6); ex <- identifyComp(ex); dat <- dataList(ex)
```

## Evaluation signals

- The recovered compound-name/area table contains no missing (zero) values for recovered compounds — the recovery step is designed to eliminate missing entries in the final table
- Only compounds found in at least min.samples = 6 samples (the gcspikelite nine-sample demo setting) are re-searched in the remaining samples
- Compounds are re-identified after recovery: identifyComp() must be re-executed and idList() must reflect updated matching scores
- alignList(ex) returns the alignment list and dataList(ex) returns the final data list without errors, confirming the MetaboSet chain alignComp → recMissComp → identifyComp is intact

## Limitations

- After executing the missing compound recovery step, compounds must always be re-identified (identifyComp()) before final matching scores are valid
- Recovery relies on LAD regression against the mean spectrum within a window around the expected elution time, so compounds absent in nearly all samples (below min.samples) are not recovered
- Spectral libraries in non-binary readable format are required; the NIST library can only be read with NIST software, so MSP-format libraries must be imported via importMSP()
- No explicit version number is stated for the eRah package in the article

## Evidence

- [methods] Alignment is executed with `alignComp()`: "Alignment is executed with `alignComp()`"
- [methods] we always have to re-identify the compounds after executing the missing compound recovery step: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [methods] And the final data list can be returned using: dataList(ex): "And the final data list can be returned using: dataList(ex)"
- [abstract] (iii) alignment of mass spectra across samples: "(iii) alignment of mass spectra across samples"
- [readme] Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in GC-MS untargeted metabolomics: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
