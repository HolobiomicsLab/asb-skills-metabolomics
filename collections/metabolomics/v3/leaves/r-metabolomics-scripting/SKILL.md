---
name: r-metabolomics-scripting
description: Use when raw GC/MS chromatograms (e.g. GC-TOF MS plasma or gcspikelite
  spike-in data) must be deconvolved, aligned across samples, recovered for missing
  compounds, and identified by spectral library matching entirely within an R workflow.
when_to_use_negative:
- When compounds have already been deconvolved and only spectral alignment or library
  matching is needed — re-running the full deconvolution step would be redundant
- When the MS library is in NIST format, since the NIST library can only be read with
  NIST software and must first be exported to MSP
- When downstream processing must go through peak-picking tools like XCMS, MetAlign,
  or ChromaTOF that output m/z peaks rather than deconvolved compound spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: erah
  role: R package providing the five core functions (deconvolveComp, alignComp, recMissComp,
    identifyComp, export utilities) used to script the whole GC/MS workflow
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which the erah tool is written and in which all pipeline
    steps are scripted
- name: gcspikelite
  role: Spike-in example dataset used to demonstrate the erah pipeline, including
    a nine-sample recMissComp run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:2239abe8ef7465d93f9ac536f02311e2d3b8812e4212d914f85509eb5a6d9430
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - gcspikelite
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- r-script-execution-for-metabolomics
- r-scripting-for-metabolomics
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-metabolomics-scripting

## Summary

Scripting GC/MS untargeted metabolomics end-to-end in R via the eRah package: load raw chromatograms into a MetaboSet, run deconvolution (deconvolveComp), alignment (alignComp), missing compound recovery (recMissComp), and library matching (identifyComp) from a single R session.

## When to use

Use when raw GC/MS chromatograms (e.g. GC-TOF MS plasma or gcspikelite spike-in data) must be deconvolved, aligned across samples, recovered for missing compounds, and identified by spectral library matching entirely within an R workflow.

## When NOT to use

- When compounds have already been deconvolved and only spectral alignment or library matching is needed — re-running the full deconvolution step would be redundant
- When the MS library is in NIST format, since the NIST library can only be read with NIST software and must first be exported to MSP
- When downstream processing must go through peak-picking tools like XCMS, MetAlign, or ChromaTOF that output m/z peaks rather than deconvolved compound spectra

## Inputs

- Raw GC/MS chromatogram files (GC-TOF MS or GC-EI MS profile data)
- An .msp spectral library file (e.g. a custom MassBank library or an export from the Golm Metabolome Database, Fiehn, or HMDB)
- MetaboSet object holding aligned compounds (output of erah::alignComp)

## Outputs

- Alignment table of compounds across samples (alignList output)
- Final table with compound names, matching scores, and integrated area of compounds for each sample (dataList output)
- Recovered compound-by-sample intensity/area table with no missing (zero) values for recovered compounds

## How to apply

Install from CRAN (`install.packages('erah')`) or the development version (`remotes::install_github('xdomingoal/erah-devel')`). Load raw GC/MS samples into a MetaboSet object, then execute the five core functions in order: noise filtering/baseline removal; compound deconvolution via deconvolveComp() with parameters from setDecPar(); alignment of mass spectra across samples via alignComp() with setAlPar(); missing compound recovery via recMissComp() (e.g. min.samples = 6 for a nine-sample gcspikelite demo); and identification by identifyComp() against a library loaded with importMSP(). Export the aligned table with alignList(ex) and the final compound-by-sample intensity/area table with dataList(ex). Because recMissComp changes compound spectra, always re-run identifyComp after recovery, then confirm the recovered table has no zero/missing entries for recovered compounds.

## Related tools

- **erah** (R package providing the five core functions (deconvolveComp, alignComp, recMissComp, identifyComp, export utilities) used to script the whole GC/MS workflow) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which the erah tool is written and in which all pipeline steps are scripted)
- **gcspikelite** (Spike-in example dataset used to demonstrate the erah pipeline, including a nine-sample recMissComp run)

## Examples

```
install.packages('erah'); ex <- deconvolveComp(data, setDecPar()); ex <- alignComp(ex, setAlPar()); ex <- recMissComp(ex, min.samples = 6); ex <- identifyComp(ex, library = importMSP('library.msp')); dataList(ex)
```

## Evaluation signals

- dataList(ex) output contains compound names, matching scores, and integrated area for every sample
- After recMissComp with min.samples = 6 on a nine-sample experiment, the recovered table contains no missing (zero) values for recovered compounds
- Identification scores to reference MS spectra in GMD/Mass Bank exceed 80.0 similarity match factor
- Compounds are re-identified (identifyComp re-run) after executing the missing compound recovery step, confirming spectra have changed
- alignList(ex) returns an aligned compound list across samples before identification

## Limitations

- NIST library cannot be read directly by erah — it must first be exported to MSP format
- Detailed methods for the HIAE comparison processing are deferred to supporting methods, not in the main text
- No specific version numbers stated for eRah, XCMS, MetAlign, or ChromaTOF in the article
- Missing compound recovery re-searches compounds found in at least min.samples; setting this threshold too low or too high alters which compounds are recovered

## Evidence

- [abstract] erah workflow: "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
- [abstract] alignment step: "(iii) alignment of mass spectra across samples"
- [abstract] recovery step: "(iv) missing compound recovery"
- [abstract] identification step: "(v) identification of metabolites by spectral library matching using publicly available mass spectra"
- [methods] deconvolveComp usage: "The compounds in data are deconvolved using `deconvolveComp()`"
- [methods] setDecPar usage: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [methods] recMissComp follow-up: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [methods] final output: "And the final data list can be returned using: dataList(ex)"
- [results] data table guarantee: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [results] NIST limitation: "Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format."
- [discussion] library availability: "e Rah is a free computational tool written in the open language R and de- posited at a public and permanent repository (http://CRAN.R-project.org/package=erah)"
- [methods] example dataset: "example samples from a spike-in experiment designed for interrogating data processing methods from the **[gcspikelite](https://bioconductor.org/packages/release/data/experiment/html/gcspikelite.html)*"
