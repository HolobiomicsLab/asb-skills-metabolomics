---
name: gcms-chromatogram-noise-filtering
description: Use when raw GC/MS chromatogram files (e.g. NetCDF/CDF from a spike-in
  or metabolomics experiment) must be baseline-corrected and noise-filtered as the
  first stage of an eRah untargeted metabolomics pipeline.
when_to_use_negative:
- Input is already a deconvolved, aligned compound/feature table (e.g. output of deconvolveComp
  or alignComp) rather than raw chromatograms
- The data are LC-MS, GC-QqQ targeted, or NMR rather than GC-EI MS profile chromatograms;
  eRah's stage (i) is designed for GC/MS chromatogram data
- The goal is downstream stage (ii)-(v) processing (CMLC/OSD deconvolution, alignment,
  missing compound recovery, or spectral library identification) rather than the pre-processing
  stage itself
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: erah
  role: R package implementing the noise filtering (Savitzky-Golay) and baseline removal
    (moving-minimum) pre-processing of GC/MS chromatograms via newExp, setDecPar and
    the MetaboSet framework
  repo: http://CRAN.R-project.org/package=erah
- name: R
  role: Open language in which eRah is written and in which the pre-processing workflow
    is executed
- name: gcspikelite
  role: Bioconductor example dataset of nine CDF spike-in chromatogram files used
    as input for the pre-processing stage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:1c6a5d8c1ebb0e2d5c7feed55f9b0e42d3f91550e22df2a16041d6fcd1e37e03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - erah
  - gcspikelite
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gcms-chromatogram-noise-filtering

## Summary

Pre-processing stage (i) of the eRah GC/MS workflow: correcting baseline drift with a moving-minimum filter and removing noise with a Savitzky-Golay filter, converting raw GC-EI MS chromatograms into smoothed ones ready for spectral deconvolution.

## When to use

Use when raw GC/MS chromatogram files (e.g. NetCDF/CDF from a spike-in or metabolomics experiment) must be baseline-corrected and noise-filtered as the first stage of an eRah untargeted metabolomics pipeline.

## When NOT to use

- Input is already a deconvolved, aligned compound/feature table (e.g. output of deconvolveComp or alignComp) rather than raw chromatograms
- The data are LC-MS, GC-QqQ targeted, or NMR rather than GC-EI MS profile chromatograms; eRah's stage (i) is designed for GC/MS chromatogram data
- The goal is downstream stage (ii)-(v) processing (CMLC/OSD deconvolution, alignment, missing compound recovery, or spectral library identification) rather than the pre-processing stage itself

## Inputs

- Nine raw GC/MS chromatogram files in CDF (NetCDF) format from the gcspikelite spike-in experiment
- Sample class/group metadata (targets$Group) ordered by FileName
- eRah Deconvolution parameters object created with setDecPar (min.peak.width = 1, min.peak.height = 2000, noise.threshold = 500, avoid.processing.mz = c(35:69, 73:75, 147:149))

## Outputs

- A MetaboSet experiment object holding baseline-corrected, Savitzky-Golay-smoothed chromatograms for all 9 samples of the 3 triplicate classes
- Saved RData file (preprocessed_chromatograms.rda) containing the cleaned chromatogram signals ready for the deconvolution stage

## How to apply

Load the gcspikelite example data (data(targets)) and build the file vector with list.files(system.file('data', package='gcspikelite'), full.names=TRUE) filtered on 'CDF' to obtain the nine raw chromatogram files. Assemble sample metadata with createInstrumentalTable(files) and createPhenoTable(files, cls), where cls comes from targets$Group ordered by FileName, then instantiate the MetaboSet experiment with newExp(). Define processing parameters with setDecPar(min.peak.width = 1, min.peak.height = 2000, noise.threshold = 500, avoid.processing.mz = c(35:69, 73:75, 147:149)), matching the paper's exclusion of the ubiquitous trimethylsilyl fragment masses m/z 73-75 and 147-149 and low-mass fragments 35-69. Run the pre-processing stage (noise filtering and baseline removal) over each chromatogram, stopping before spectral deconvolution so only cleaned chromatograms are retained. Inspect the result via the MetaboSet object, sampleInfo, and plotChr chromatogram plots, then save the processed experiment (e.g. preprocessed_chromatograms.rda) with save().

## Related tools

- **erah** (R package implementing the noise filtering (Savitzky-Golay) and baseline removal (moving-minimum) pre-processing of GC/MS chromatograms via newExp, setDecPar and the MetaboSet framework) — http://CRAN.R-project.org/package=erah
- **R** (Open language in which eRah is written and in which the pre-processing workflow is executed)
- **gcspikelite** (Bioconductor example dataset of nine CDF spike-in chromatogram files used as input for the pre-processing stage)

## Examples

```
ex <- newExp(instrumental = createInstrumentalTable(files), phenotype = createPhenoTable(files, cls), info = 'DEMO Experiment')
```

## Evaluation signals

- The saved RData file loads and contains a MetaboSet with all 9 samples spanning the 3 triplicate classes
- Chromatogram signals exist for each sample with the excluded m/z channels (73-75, 147-149, 35-69) masked per avoid.processing.mz
- plotChr chromatogram plots of the pre-processed samples show visibly smoothed, baseline-corrected signals versus the raw CDF traces
- The workflow stops before deconvolution: no deconvolveComp/alignList output is present, only cleaned chromatograms
- The setDecPar parameters actually used match the paper's values (min.peak.width = 1, min.peak.height = 2000, noise.threshold = 500)

## Limitations

- The paper validates the workflow on serum samples analyzed by GC-TOF MS; performance on other instrument types is not demonstrated
- No specific version numbers are stated for the eRah package (or compared tools XCMS, MetAlign, ChromaTOF), so exact filter behavior may vary across versions
- The supporting methods and data files (Supporting Files 1-4) are hosted externally at pubs.acs.org rather than in the main text, so some implementation details are deferred

## Evidence

- [abstract] Stage (i) noise filtering and baseline removal: "(i) noise filtering and baseline removal of GC/MS chromatograms"
- [methods] setDecPar creates the parameters object: "This function requires a `Deconvolution parameters` object, that can be created with `setDecPar` function"
- [readme] eRah is an R tool for GC-MS untargeted metabolomics: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
