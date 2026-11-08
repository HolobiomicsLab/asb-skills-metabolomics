---
name: gcms-spectral-deconvolution
description: Use when raw GC/MS chromatograms (netCDF or mzXML) contain co-eluting
  compounds with extensive EI fragmentation and you need pure compound spectra and
  integrated areas per sample rather than fragment-ion peak picking.
when_to_use_negative:
- When the goal is generic fragment-ion peak picking and alignment without compound-level
  spectra — peak-picking approaches (XCMS centWave, MetAlign, ChromaTOF) are the alternative;
  this skill exists precisely because those methods quantitate on m/z value, retention
  time window and area of fragment ion peaks and leave identification as the bottleneck
- When the input is already a deconvolved MetaboSet or a compound-by-sample area table
  — then the downstream skills (alignment with alignComp(), missing compound recovery
  with recMissComp(), identification with identifyComp()) apply instead
- When the mass spectral library is only readable with NIST software — eRah requires
  MS libraries in non-binary format importable via importMSP(); export found spectra
  to MSP and use MassBank/GMD/HMDB libraries instead
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: R package implementing the CMLC + OSD spectral deconvolution, LAD-based quantification,
    alignment and library matching pipeline
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Language in which eRah is written and executed
  repo: http://CRAN.R-project.org/package=erah
- name: GC-TOF MS
  role: 'Instrument data source: plasma GC-TOF MS chromatograms used to demonstrate
    the deconvolution'
- name: XCMS
  role: Peak-picking baseline comparator (centWave algorithm) whose results eRah's
    deconvolution is benchmarked against
- name: MetAlign
  role: Peak-picking comparator for quantitative evaluation
- name: ChromaTOF
  role: Commercial peak-picking comparator for quantitative evaluation
- name: gcspikelite
  role: Spike-in example samples for interrogating the deconvolution workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:5c95e8f0139b242a4512e6272fd720da2537d3ea91d9a6f06946aa3c1376433b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChromaTOF
  - GC-TOF MS
  - MetAlign
  - R
  - XCMS
  - erah
  - gcspikelite
  - osd
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gcms-spectral-deconvolution

## Summary

Blind-source-separation-based spectral deconvolution of GC-MS chromatograms in R: the eRah pipeline applies compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD) to resolve co-eluting compounds in electron ionization data into pure compound spectra plus per-sample integrated areas.

## When to use

Use when raw GC/MS chromatograms (netCDF or mzXML) contain co-eluting compounds with extensive EI fragmentation and you need pure compound spectra and integrated areas per sample rather than fragment-ion peak picking.

## When NOT to use

- When the goal is generic fragment-ion peak picking and alignment without compound-level spectra — peak-picking approaches (XCMS centWave, MetAlign, ChromaTOF) are the alternative; this skill exists precisely because those methods quantitate on m/z value, retention time window and area of fragment ion peaks and leave identification as the bottleneck
- When the input is already a deconvolved MetaboSet or a compound-by-sample area table — then the downstream skills (alignment with alignComp(), missing compound recovery with recMissComp(), identification with identifyComp()) apply instead
- When the mass spectral library is only readable with NIST software — eRah requires MS libraries in non-binary format importable via importMSP(); export found spectra to MSP and use MassBank/GMD/HMDB libraries instead

## Inputs

- Cleaned GC/MS chromatograms in netCDF or mzXML format (output of noise filtering and baseline removal)
- Instrumental and phenotype metadata tables for newExp()
- Deconvolution parameter object built with setDecPar() (min.peak.width, min.peak.height, noise.threshold, avoid.processing.mz)
- Optionally a parallel backend (e.g. future::multisession)

## Outputs

- Deconvolved MetaboSet object containing pure compound spectra and integrated areas for each sample
- Saved R data file (e.g. deconvolved_experiment.rda)
- Per-sample compound profile from LAD regression (compound area across samples)

## How to apply

Load cleaned chromatograms into an eRah MetaboSet with newExp(), then run deconvolveComp() after building parameters with setDecPar(min.peak.width, min.peak.height, noise.threshold, avoid.processing.mz) — e.g. min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149) to suppress columns/solvent artifacts. deconvolveComp() first applies CMLC, which exploits covariance patterns of co-eluting ion redundancy to detect compounds, then OSD (a blind source separation multivariate technique) retrieves the pure compound spectrum given the elution profile; a least absolute deviation (LAD) regression yields the quantitative compound profile and integrated area for each sample. Optionally register a parallel backend (plan(future::multisession, workers=2)) for large sample sets; a 25-sample benchmark ran in under 30 minutes on a 2.4 GHz Intel Core i7. Save the deconvolved MetaboSet with save(ex, file='deconvolved_experiment.rda') before downstream alignment and library matching. This deconvolution step is what distinguishes eRah from peak-picking tools (XCMS centWave, MetAlign, ChromaTOF), which quantitate on m/z / retention-time windows and leave compound identification as the main bottleneck.

## Related tools

- **erah** (R package implementing the CMLC + OSD spectral deconvolution, LAD-based quantification, alignment and library matching pipeline) — http://CRAN.R-project.org/package=erah
- **R** (Language in which eRah is written and executed) — http://CRAN.R-project.org/package=erah
- **GC-TOF MS** (Instrument data source: plasma GC-TOF MS chromatograms used to demonstrate the deconvolution)
- **XCMS** (Peak-picking baseline comparator (centWave algorithm) whose results eRah's deconvolution is benchmarked against)
- **MetAlign** (Peak-picking comparator for quantitative evaluation)
- **ChromaTOF** (Commercial peak-picking comparator for quantitative evaluation)
- **gcspikelite** (Spike-in example samples for interrogating the deconvolution workflow)

## Examples

```
ex <- newExp(...); dec.par <- setDecPar(min.peak.width=1, min.peak.height=2000, noise.threshold=500, avoid.processing.mz=c(73:75, 147:149)); ex <- deconvolveComp(ex, dec.par); save(ex, file='deconvolved_experiment.rda')
```

## Evaluation signals

- Deconvolved MetaboSet contains pure compound spectra and integrated areas for every sample (verified via save()/load round-trip of the .rda)
- Compounds resolved from co-eluted regions show high similarity match factor (>80.0) to reference MS spectra in GMD or MassBank
- Quantitative results comparable to XCMS centWave, MetAlign and ChromaTOF on the same dataset (the article's validation design)
- Compound areas consistent across replicate samples and responsive in spike-in experiments (gcspikelite) — dysregulated metabolites later validated by pure standards and targeted GC-QqQ / LC-QqQ / NMR analysis
- Runtime sanity: 25 samples deconvolved in under 30 minutes on a 2.4 GHz Intel Core i7

## Limitations

- eRah processes and compares against tools without stated versions; no explicit version number is given for the package in the article text
- NIST library cannot be read directly — eRah requires libraries in non-binary readable format (MassBank/GMD/HMDB/Fiehn or internal); eRah can export found spectra to MSP for use with other tools
- Validation scope in the article was serum samples analyzed by GC-TOF MS; behavior on other instruments/matrices is extrapolation
- Co-elution of compounds from complex biological samples in GC along with extensive fragmentation of molecular ions by EI ionization produce large and complex datasets — deconvolution quality depends on ion covariance structure and is challenged by heavy co-elution and extensive fragmentation of molecular ions by hard electron ionization
- Column bleed / solvent ions must be excluded via avoid.processing.mz (e.g. 73:75, 147:149, optionally 35:69); wrong exclusions degrade deconvolution or waste compute

## Evidence

- [methods] CMLC detects compounds by exploiting ion redundancy covariance patterns, then OSD retrieves the pure compound spectrum given the elution profile, and a LAD regression yields the quantitative compound profile.: "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
- [intro] Peak-picking approaches lack compound-level spectra and quantitate on m/z/RT windows, making identification the main bottleneck.: [evidence span withheld — anchor status: not_found]
- [methods] Deconvolution parameters are set with setDecPar and the compounds are deconvolved with deconvolveComp().: "The compounds in data are deconvolved using `deconvolveComp()` ... This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [readme] eRah outputs a table with compound names, matching scores and the integrated area of the compound for each sample.: "Outputs a table with compound names, matching scores and the integrated area of the compound for each sample."
- [results] NIST library can only be read with NIST software; eRah can export spectra to MSP format.: "Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format."
