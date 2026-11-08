---
name: erah-pipeline-parameter-configuration
description: Use when setting up an eRah (R) processing run on raw GC/MS (GC-TOF or
  GC-QqQ) chromatogram data and you need to instantiate the parameter objects (setDecPar,
  setAlPar) and/or the spectral library before running deconvolveComp, alignComp,
  recMissComp, and identifyComp.
when_to_use_negative:
- Input is already a processed peak/feature table produced by another peak-picking
  tool (e.g. XCMS centWave, MetAlign, ChromaTOF) — this skill configures eRah's own
  deconvolution-first pipeline, not post-hoc alignment of foreign peak lists
- Data are LC-MS or NMR — eRah's workflow is designed for GC/MS-based untargeted metabolomics
  with EI fragmentation and co-elution, not other modalities
- Only a single chromatogram needs processing and no cross-sample alignment, library
  matching, or compound identification is required — eRah's value is in the integrated
  multi-sample pipeline
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: R package whose parameter objects (setDecPar, setAlPar) configure the deconvolution,
    alignment, missing compound recovery, and identification steps
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Programming language/runtime in which the eRah parameter objects are created
    and the pipeline is executed
- name: importMSP
  role: Loads an MS spectral library (e.g. bundled MassBank custom library) whose
    parameters must be set before identifyComp is run
  repo: https://github.com/xdomingoal/erah-devel
- name: alignComp
  role: Executes cross-sample alignment using the alignment parameters defined by
    setAlPar (min.spectra.cor, max.time.dist, mz.range)
  repo: https://github.com/xdomingoal/erah-devel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:7bd72f1869c6bb00faf069fa09b39e1cb0a47d93275dc5aa48a9d2a44173b017
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - alignComp
  - erah
  - importMSP
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# erah-pipeline-parameter-configuration

## Summary

Configures the parameter objects that drive the eRah R package's five-step GC/MS untargeted metabolomics workflow — deconvolution, alignment, missing-compound recovery, and spectral library identification — so that co-eluting compounds are separated and compounds are consistently matched and quantified across samples.

## When to use

Use when setting up an eRah (R) processing run on raw GC/MS (GC-TOF or GC-QqQ) chromatogram data and you need to instantiate the parameter objects (setDecPar, setAlPar) and/or the spectral library before running deconvolveComp, alignComp, recMissComp, and identifyComp.

## When NOT to use

- Input is already a processed peak/feature table produced by another peak-picking tool (e.g. XCMS centWave, MetAlign, ChromaTOF) — this skill configures eRah's own deconvolution-first pipeline, not post-hoc alignment of foreign peak lists
- Data are LC-MS or NMR — eRah's workflow is designed for GC/MS-based untargeted metabolomics with EI fragmentation and co-elution, not other modalities
- Only a single chromatogram needs processing and no cross-sample alignment, library matching, or compound identification is required — eRah's value is in the integrated multi-sample pipeline

## Inputs

- Raw GC/MS chromatogram files (e.g. NetCDF/mzML) from GC-TOF MS or GC-QqQ MS runs
- MetaboSet experiment object built by eRah's load functions
- MS spectral library file (MSP format) importable with importMSP, e.g. the bundled MassBank custom library or Golm Metabolome Database (GMD) export

## Outputs

- MetaboSet object with deconvolved, aligned, and identified compounds
- Aligned compound table (alignList(ex)) with one row per unique aligned compound (AlignId) and per-sample integrated area
- Final data list (dataList(ex)) with compound names, matching scores, and integrated area per sample

## How to apply

Build a MetaboSet from the raw chromatograms and set the processing parameters with the dedicated helper functions: setDecPar for deconvolution and setAlPar for alignment (e.g. min.spectra.cor = 0.90, max.time.dist = 3 s, mz.range = 70:600 for alignment) because alignComp clusters deconvolved compounds within user-defined boundaries of maximum retention time drift and minimum spectral similarity without internal references such as retention indices. Deconvolve the compounds with deconvolveComp using the Deconvolution parameters object created by setDecPar, which implements the compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD) multivariate deconvolution; then align across samples with alignComp (which computes the Euclidean distance between retention time distance and spectral similarity), optionally recover missing compounds with recMissComp, and re-identify compounds afterwards with identifyComp against an imported MS library loaded via importMSP (e.g. the bundled MassBank custom version or the Golm Metabolome Database). Access intermediate and final results with alignList(ex) and dataList(ex). The parameter configuration determines whether co-eluting compounds remain distinct rows and whether compounds appearing across the maximum number of samples with the least retention time and spectral distance are grouped together.

## Related tools

- **erah** (R package whose parameter objects (setDecPar, setAlPar) configure the deconvolution, alignment, missing compound recovery, and identification steps) — http://CRAN.R-project.org/package=erah
- **R** (Programming language/runtime in which the eRah parameter objects are created and the pipeline is executed)
- **importMSP** (Loads an MS spectral library (e.g. bundled MassBank custom library) whose parameters must be set before identifyComp is run) — https://github.com/xdomingoal/erah-devel
- **alignComp** (Executes cross-sample alignment using the alignment parameters defined by setAlPar (min.spectra.cor, max.time.dist, mz.range)) — https://github.com/xdomingoal/erah-devel

## Examples

```
ex.al.par <- setAlPar(min.spectra.cor = 0.90, max.time.dist = 3, mz.range = 70:600); ex <- alignComp(ex, alParameters = ex.al.par)
```

## Evaluation signals

- The aligned table must exist and contain per-compound AlignId and per-sample area columns
- Each grouped compound entry must satisfy the alignment criteria imposed by setAlPar — min.spectra.cor = 0.90 and max.time.dist = 3 s
- Co-eluting compounds deconvolved upstream remain distinct rows in alignList(ex) rather than being merged
- Parameter objects must be created with setDecPar / setAlPar prior to running deconvolveComp / alignComp, and compounds must be re-identified after executing the missing compound recovery step (recMissComp)
- Quantitative output should be a table with compound names, matching scores, and integrated area of compounds for each sample

## Limitations

- NIST library can only be read with NIST software; use eRah's function to export all spectra found in a given experiment to MSP format instead, and import other libraries such as GMD or HMDB
- Compounds absent in some samples are only retrievable via the optional missing compound recovery step (recMissComp), and the compounds must be re-identified after that step
- No specific version number for eRah is stated in the article; the tool is obtained from CRAN or the erah-devel GitHub repo
- The spectral similarity comparison is restricted to the m/z range set in mz.range (e.g. 70:600), so compounds outside that range may not align correctly

## Evidence

- [methods] Parameter configuration drives alignment clustering criteria: [evidence span withheld — anchor status: not_found]
- [methods] setAlPar parameters concrete example: [evidence span withheld — anchor status: not_found]
- [methods] Deconvolution parameters object: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [methods] Re-identification after missing compound recovery: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [abstract] Pipeline outputs: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [results] Library import restriction: "Note that NIST library can only be read with NIST software. However, e Rah has a function to export all the spectra found in a given experiment to MSP format."
- [readme] Package install (README): "From an R console, execute:

```r
install.packages('erah')```"
