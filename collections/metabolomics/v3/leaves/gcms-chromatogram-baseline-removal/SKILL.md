---
name: gcms-chromatogram-baseline-removal
description: Use when raw GC/MS chromatogram files (e.g. CDF/NetCDF) contain baseline
  drift and noise that must be removed before compound deconvolution, alignment, or
  metabolite identification in an untargeted metabolomics pipeline.
when_to_use_negative:
- The input is already a peak/feature table or a table of compound names, matching
  scores and integrated areas — i.e. stages (ii)–(v) have already run
- The goal is spectral deconvolution (deconvolveComp), alignment (alignComp) or library
  identification (identifyComp) rather than chromatogram cleaning
- The data are not GC/MS chromatograms (e.g. LC-QqQ or NMR data), which this stage
  of the eRah GC/MS workflow is not designed for
tools:
- name: erah
  role: R package implementing the stage (i) noise filtering and baseline removal
    pre-processing of GC/MS chromatograms within its five-step workflow
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which eRah is written and in which the pre-processing is
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:eef872831815e27d8b0bfcc6693da2495a10512cb8f94b0c5ef9f7cb13eb29dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gcms-chromatogram-baseline-removal

## Summary

Stage (i) of the eRah GC/MS workflow: correcting baseline drift with a moving-minimum filter and smoothing noise with a Savitzky-Golay filter, converting raw GC/MS chromatograms (e.g. NetCDF/CDF files) into cleaned chromatograms ready for spectral deconvolution.

## When to use

Use when raw GC/MS chromatogram files (e.g. CDF/NetCDF) contain baseline drift and noise that must be removed before compound deconvolution, alignment, or metabolite identification in an untargeted metabolomics pipeline.

## When NOT to use

- The input is already a peak/feature table or a table of compound names, matching scores and integrated areas — i.e. stages (ii)–(v) have already run
- The goal is spectral deconvolution (deconvolveComp), alignment (alignComp) or library identification (identifyComp) rather than chromatogram cleaning
- The data are not GC/MS chromatograms (e.g. LC-QqQ or NMR data), which this stage of the eRah GC/MS workflow is not designed for

## Inputs

- Raw GC/MS chromatogram files (NetCDF/CDF), e.g. the nine gcspikelite spike-in chromatograms
- Sample metadata: instrumental table (createInstrumentalTable) and phenotype table (createPhenoTable) with class labels (e.g. targets$Group ordered by FileName)

## Outputs

- A MetaboSet experiment object holding smoothed, baseline-corrected chromatograms for every sample (e.g. preprocessed_chromatograms.rda)
- Chromatogram plots (plotChr) confirming cleaning was applied to each sample

## How to apply

In R, load the erah package and assemble the experiment from raw chromatogram files (for the gcspikelite spike-in demo, list CDF files via list.files(system.file('data', package='gcspikelite'), full.names=TRUE)). Build the instrumental and phenotype tables with createInstrumentalTable(files) and createPhenoTable(files, cls), then instantiate the MetaboSet with newExp(instrumental, phenotype, info). Define processing parameters with setDecPar — for the gcspikelite data: min.peak.width=1, min.peak.height=2000, noise.threshold=500, and avoid.processing.mz=c(35:69, 73:75, 147:149) to mask the ubiquitous trimethylsilyl fragments m/z 73-75, 147-149 and low-mass fragments 35-69. Run only the stage (i) pre-processing (noise filtering and baseline removal) over each chromatogram, stopping before deconvolveComp() so that only cleaned, smoothed chromatograms are retained; inspect via sampleInfo and plotChr before saving with save().

## Related tools

- **erah** (R package implementing the stage (i) noise filtering and baseline removal pre-processing of GC/MS chromatograms within its five-step workflow) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which eRah is written and in which the pre-processing is executed)

## Examples

```
ex <- newExp(createInstrumentalTable(files), createPhenoTable(files, cls), info='DEMO Experiment')
```

## Evaluation signals

- The saved RData file (e.g. preprocessed_chromatograms.rda) loads and contains a MetaboSet with all 9 gcspikelite samples across the 3 triplicate classes
- Chromatogram signals exist for every sample and plotChr output shows smoothed chromatograms rather than raw drift
- The excluded m/z channels (35-69, 73-75, 147-149) are masked in the processed experiment
- Pre-processing stopped before deconvolution: no deconvolveComp/alignment output tables are present in the object
- Sample metadata (sampleInfo) correctly pairs each of the nine CDF file paths with its class from targets$Group

## Limitations

- Pre-processing parameters are set via setDecPar prior to running, and the article does not state specific eRah version numbers used
- Validation scope in the paper is limited to GC-TOF MS serum data and the gcspikelite spike-in example; other chromatogram types may need parameter tuning
- This stage alone does not identify or quantify metabolites — it only cleans chromatograms; downstream stages (deconvolution, alignment, recovery, identification) are required for a compound table

## Evidence

- [abstract] Stage (i) of the eRah GC/MS workflow is noise filtering and baseline removal: "(i) noise filtering and baseline removal of GC/MS chromatograms"
- [methods] The gcspikelite spike-in experiment is the recommended example dataset for interrogating eRah data processing: "example samples from a spike-in experiment designed for interrogating data processing methods from the"
- [readme] eRah is installed from CRAN and documented online: [evidence span withheld — anchor status: external_doc]
- [intro] Motivation: a free, open-source tool integrating all GC/MS data-processing steps was needed: "there is a need for a free and open source software that integrates all the necessary steps for data processing in GC–MS-based untargeted metabolomics"
