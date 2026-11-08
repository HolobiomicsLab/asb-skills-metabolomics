---
name: lc-ms-feature-detection
description: Use when raw centroided LC-MS1 spectra (e.g. mzML files from Metabolomics
  Workbench accessions like ST002796/ST002798) must be converted into a quantified
  MS1 feature table as the precursor step to MS2 deconvolution, DB searching, or pathway
  analysis.
when_to_use_negative:
- Input is already a processed feature/peak table — proceed directly to statistical
  analysis or Convert2Mummichog rather than re-detecting peaks
- The primary goal is MS2 spectral deconvolution or compound identification — use
  PerformDDADeconvolution/PerformDIADeconvolution and PerformDBSearchingBatch instead;
  feature detection only supplies the target list
- Data are from stable isotope-labeling experiments, which MetaboAnalystR 4.0 does
  not yet support
edam_operation: http://edamontology.org/operation_3225
tools:
- name: MetaboAnalystR 4.0
  role: Unified R package providing the auto-optimized feature detection and quantification
    module for LC-MS1 spectra processing
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: Backend engine installed alongside MetaboAnalystR for auto-optimized LC-MS1
    peak picking, alignment, and gap filling
  repo: https://github.com/xia-lab/OptiLCMS
- name: ProteoWizard
  role: Used to convert raw LC-MS data to centroid-mode mzML before feature detection
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:a5cde8a1ac3e5b5932d0c5366f2bde44631f10f03bb9010a8867bde9329ace97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MS-DIAL
  - MS-FINDER
  - MZmine
  - MetaboAnalystR 4.0
  - OptiLCMS
  - ProteoWizard
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3225
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lc-ms-feature-detection

## Summary

Auto-optimized LC-MS1 feature detection and quantification in MetaboAnalystR 4.0/OptiLCMS, turning raw centroided mzML spectra into complete MS1 peak/feature tables (peak picking, alignment, gap filling, annotation) that feed compound identification and mummichog functional analysis.

## When to use

Use when raw centroided LC-MS1 spectra (e.g. mzML files from Metabolomics Workbench accessions like ST002796/ST002798) must be converted into a quantified MS1 feature table as the precursor step to MS2 deconvolution, DB searching, or pathway analysis.

## When NOT to use

- Input is already a processed feature/peak table — proceed directly to statistical analysis or Convert2Mummichog rather than re-detecting peaks
- The primary goal is MS2 spectral deconvolution or compound identification — use PerformDDADeconvolution/PerformDIADeconvolution and PerformDBSearchingBatch instead; feature detection only supplies the target list
- Data are from stable isotope-labeling experiments, which MetaboAnalystR 4.0 does not yet support

## Inputs

- Raw centroided LC-MS1 mzML files (e.g. from Metabolomics Workbench ST002796/ST002798)
- R session with MetaboAnalystR 4.0 and OptiLCMS installed

## Outputs

- Complete MS1 feature table with peak m/z, retention time, and per-sample intensities
- MS2 target list of features for subsequent DDA/DIA deconvolution and compound identification
- Feature table usable as unfiltered MS1 input to Convert2Mummichog functional analysis

## How to apply

Install MetaboAnalystR 4.0 and the OptiLCMS backend from GitHub in R (>3.6.1 recommended), then run the auto-optimized workflow on the raw centroided mzML files: peak picking, peak alignment, gap filling, and peak annotation, so no manual parameter tuning is required. The resulting complete MS1 feature table (m/z, retention time, per-sample intensities) serves as the MS2 target list for downstream steps — e.g. it feeds PerformDDADeconvolution/PerformDIADeconvolution and Convert2Mummichog. For SWATH-DIA data, the related MSnbase function reconstructChromPeakSpectra can be used to deconvolve the data after MS1 feature detection. The rationale is that integrated LC-MS1 and MS2 processing was a primary bottleneck in computational efficiency and parameter-setting expertise; the auto-optimized algorithm addresses this and was validated on standards mixtures, dilution series, and clinical metabolomics samples. Convert raw data to centroid mode with ProteoWizard as in the paper before feature detection.

## Related tools

- **MetaboAnalystR 4.0** (Unified R package providing the auto-optimized feature detection and quantification module for LC-MS1 spectra processing) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (Backend engine installed alongside MetaboAnalystR for auto-optimized LC-MS1 peak picking, alignment, and gap filling) — https://github.com/xia-lab/OptiLCMS
- **ProteoWizard** (Used to convert raw LC-MS data to centroid-mode mzML before feature detection)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T); devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- A complete MS1 feature table (with m/z, RT, and intensities after gap filling) is produced and can serve as the MS2 target list for downstream deconvolution
- Feature detection runs without manual parameter tuning — the auto-optimized algorithm selects parameters itself
- Benchmark expectation: MetaboAnalystR 4.0 detects and identifies >10% more high-quality MS and MS/MS features than alternatives on dilution series
- The feature table supports downstream reproduction checks, e.g. the SWATH-DIA MS2-filtered mummichog run recovering four additional significant pathways (phosphatidylinositol phosphate, vitamin D, vitamin C, arachidonic cascades)
- Input files are centroid mode; if profile data are supplied, convert with ProteoWizard first or results will be unreliable

## Limitations

- Requires centroided spectra — the study's data were converted to centroid mode with ProteoWizard; profile-mode input is not covered
- R base >3.6.1 is recommended; compatibility of the latest R version (v4.0.0) was stated to be under evaluation
- System dependencies must be pre-installed (libcairo2-dev, libnetcdf-dev, libxml2, libxt-dev, libssl-dev on Ubuntu; Rtools on Windows; Xcode and GNU Fortran on Mac)
- Integrated LC-MS1 and MS2 processing remains computationally demanding; parameter-setting expertise historically a bottleneck, mitigated but not eliminated by auto-optimization

## Evidence

- [abstract] Auto-optimized LC-MS1 feature detection (abstract): "an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing"
- [readme] Feature detection module in README: "1) an auto-optimized feature detection and quantification module for LC-MS1 spectra processing;"
- [readme] Benchmark claim on feature coverage: "Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features."
- [methods] Install command for OptiLCMS backend: "devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)"
- [methods] SWATH-DIA deconvolution relies on MS1 feature detection: "we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data."
