---
name: pseudo-ms2-spectrum-reconstruction
description: Use when input is centroid-mode SWATH-DIA mzML with sequential variable-size
  MS2 windows (e.g., 10 windows, 1.0 m/z overlap, ~0.9 s cycle time) and pseudo-MS2
  spectra must be reconstructed per MS1 feature.
when_to_use_negative:
- The data are DDA with chimeric spectra — use the auto-optimized DDA deconvolution
  workflow (PerformDDADeconvolution) instead
- Input is already a feature table or processed peak list with no raw SWATH MS2 windows
  available for EIC detection
- MS1-only data with no MS2 spectra, where relinking precursors to fragments is impossible
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides the PerformDIADeconvolution function wrapping the DecoMetDIA-style
    Rcpp/C++ deconvolution algorithm for SWATH-DIA data
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Loads the mzML data and parses the MS1 feature table and sequential SWATH
    MS2 windows
- name: XCMS
  role: Reference implementation (reconstructChromPeakSpectra) used to benchmark the
    deconvoluted spectra
- name: R
  role: Open-source environment in which the deconvolution workflow is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:5f860b433a6f95fb73edafd8589b35005e41efd0908b99676d56fe15cd7392d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - R
  - XCMS
  - mzR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Pseudo-MS2 spectrum reconstruction for SWATH-DIA data

## Summary

Deconvolves SWATH-DIA MS2 spectra so that precursors are correctly relinked with their fragment ions when all ions in a wide m/z range are co-fragmented, exporting a pseudo-MS2 spectrum per target MS1 feature. Implemented in MetaboAnalystR 4.0 as a DecoMetDIA-style algorithm with an Rcpp/C++ core supporting parallel computing.

## When to use

Use when input is centroid-mode SWATH-DIA mzML with sequential variable-size MS2 windows (e.g., 10 windows, 1.0 m/z overlap, ~0.9 s cycle time) and pseudo-MS2 spectra must be reconstructed per MS1 feature.

## When NOT to use

- The data are DDA with chimeric spectra — use the auto-optimized DDA deconvolution workflow (PerformDDADeconvolution) instead
- Input is already a feature table or processed peak list with no raw SWATH MS2 windows available for EIC detection
- MS1-only data with no MS2 spectra, where relinking precursors to fragments is impossible

## Inputs

- Centroid-mode mzML files of SWATH-DIA LC-MS runs (e.g., Metabolomics Workbench ST002796 / ST002798)
- MS1 feature table parsed from the mzML data
- SWATH window definition (10 variable-size windows with 1.0 m/z neighbor overlap, ~0.9 s cycle time)

## Outputs

- Pseudo-MS2 spectra, one per target MS1 feature (the cluster containing the original feature)
- Deconvoluted spectra written to an .msp file
- Benchmark comparison against XCMS reconstructChromPeakSpectra on the same input

## How to apply

Retrieve raw SWATH-DIA metabolomics spectra (e.g., Metabolomics Workbench accessions ST002796 and ST002798, centroid mzML mode) and load them with the mzR package, parsing the MS1 feature table plus the sequential SWATH MS2 windows. For each target MS feature, detect all extracted ion chromatograms (EICs) of MS2 peaks from its corresponding SWATH window. Cluster the EICs by chromatographic peak-shape similarity and retention-time coelution, select one model peak per cluster, and use the model peaks to decompose all EICs. Reconstruct the MS2 cluster composition from the decomposed components and export the cluster containing the original MS feature as a pseudo-MS2 spectrum, using the Rcpp/C++ core behind the PerformDIADeconvolution interface (the DecoMetDIA approach, re-implemented with parallel computing to fix the original computational bottleneck). Benchmark the module against XCMS's reconstructChromPeakSpectra on the same SWATH-DIA input and write the deconvoluted spectra to an .msp file.

## Related tools

- **MetaboAnalystR 4.0** (Provides the PerformDIADeconvolution function wrapping the DecoMetDIA-style Rcpp/C++ deconvolution algorithm for SWATH-DIA data) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Loads the mzML data and parses the MS1 feature table and sequential SWATH MS2 windows)
- **XCMS** (Reference implementation (reconstructChromPeakSpectra) used to benchmark the deconvoluted spectra)
- **R** (Open-source environment in which the deconvolution workflow is executed)

## Examples

```
mSet <- MetaboAnalystR::PerformDIADeconvolution(mSet)  # SWATH-DIA deconvolution; write deconvoluted pseudo-MS2 spectra to .msp
```

## Evaluation signals

- The cluster containing each original MS feature is exported as a pseudo-MS2 spectrum in the .msp output
- Deconvoluted spectra are consistent with the XCMS reconstructChromPeakSpectra reference on the same SWATH-DIA dataset
- Input mzML files are confirmed to be in centroid mode before loading with mzR
- Each exported pseudo-MS2 spectrum maps back to exactly one target MS1 feature from the SWATH window it co-elutes with
- The run completes in acceptable time, confirming the parallel-computing Rcpp/C++ implementation removes the original computational bottleneck

## Limitations

- The original DecoMetDIA implementation had a computational bottleneck; the re-implemented Rcpp/C++ core with parallel computing exists to address this, so pure-R reference implementations may be slow on large files
- Built-in MS2 reference spectra are compiled from various instruments and experimental conditions, resulting in potentially varying qualities
- The SWATH window scheme (10 variable-size windows with 1.0 m/z neighbor overlap) must match the acquisition; different window layouts are not covered by the cited validation

## Evidence

- [methods] The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution.: "The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution."
- [intro] spectra deconvolution is essential to relink precursors with fragment ions: "spectra deconvolution is essential to relink precursors with fragment ions"
- [results] we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data.: "we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data."
- [readme] a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA);: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA);"
