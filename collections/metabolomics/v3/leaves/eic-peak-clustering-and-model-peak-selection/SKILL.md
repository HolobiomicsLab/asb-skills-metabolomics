---
name: eic-peak-clustering-and-model-peak-selection
description: Use when processing centroided SWATH-DIA mzML data where all ions in
  a wide m/z range are co-fragmented, producing chimeric MS2 spectra that must be
  relinked to their originating MS1 precursors before compound identification.
when_to_use_negative:
- Input is DDA data with chimeric spectra — use the PerformDDADeconvolution workflow
  instead, which targets chimeric DDA spectra
- Input is already a feature table without raw MS2 spectra — deconvolution requires
  raw SWATH MS2 EICs and cannot be run on preprocessed peak tables
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformDIADeconvolution with an Rcpp/C++ core implementing the DecoMetDIA-style
    EIC clustering, model peak selection, and decomposition with parallel computing
    support
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Loads the mzML data and parses the MS1 feature table and sequential SWATH
    MS2 windows
- name: XCMS
  role: Reference implementation (reconstructChromPeakSpectra) used to benchmark the
    deconvoluted SWATH-DIA spectra
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:247c33c28dab66512a8122fd775747eeebb0fc08bbd5532be10039e8207fa6fd
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# EIC peak clustering and model peak selection

## Summary

Deconvolves SWATH-DIA MS2 spectra by extracting MS2 EICs from the corresponding SWATH window for each target MS1 feature, clustering them by chromatographic peak-shape similarity and retention-time coelution, selecting one model peak per cluster, and using model peaks to decompose all EICs so precursors are correctly relinked with fragment ions.

## When to use

Use when processing centroided SWATH-DIA mzML data where all ions in a wide m/z range are co-fragmented, producing chimeric MS2 spectra that must be relinked to their originating MS1 precursors before compound identification.

## When NOT to use

- Input is DDA data with chimeric spectra — use the PerformDDADeconvolution workflow instead, which targets chimeric DDA spectra
- Input is already a feature table without raw MS2 spectra — deconvolution requires raw SWATH MS2 EICs and cannot be run on preprocessed peak tables

## Inputs

- Centroided mzML files containing MS1 and sequential SWATH MS2 windows
- MS1 feature table (m/z, retention time, intensity) from LC-MS1 peak detection
- SWATH window definition (variable-size windows, 1.0 m/z neighbor overlap)

## Outputs

- Pseudo-MS2 spectra per MS1 feature, exported as .msp files
- Deconvoluted EIC cluster compositions (model peaks and decomposed components)

## How to apply

Load centroided SWATH-DIA mzML files with the mzR package and parse the MS1 feature table plus the sequential SWATH MS2 windows (e.g. 10 variable-size windows with 1.0 m/z neighbor overlap, ~0.9 s cycle time). For each target MS1 feature, detect all extracted ion chromatograms (EICs) of MS2 peaks from its corresponding SWATH window. Cluster the EICs by chromatographic peak-shape similarity and retention-time coelution, then select one model peak per cluster and use the model peaks to decompose all EICs in the cluster. Reconstruct the MS2 cluster composition from the decomposed components and export the cluster containing the original MS feature as a pseudo-MS2 spectrum. In MetaboAnalystR 4.0 this is implemented behind the PerformDIADeconvolution interface with a Rcpp/C++ core following the DecoMetDIA approach, optimized for parallel computing to overcome the original implementation's computational bottleneck.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformDIADeconvolution with an Rcpp/C++ core implementing the DecoMetDIA-style EIC clustering, model peak selection, and decomposition with parallel computing support) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Loads the mzML data and parses the MS1 feature table and sequential SWATH MS2 windows)
- **XCMS** (Reference implementation (reconstructChromPeakSpectra) used to benchmark the deconvoluted SWATH-DIA spectra)

## Examples

```
PerformDIADeconvolution(mSet)
```

## Evaluation signals

- The cluster containing each original MS feature is exported as a pseudo-MS2 spectrum in the .msp output
- Deconvoluted spectra are consistent with the XCMS reconstructChromPeakSpectra reference on the same SWATH-DIA dataset
- All EICs in each cluster decompose to components that reconstruct the MS2 cluster composition without orphaned fragments
- One model peak is selected per EIC cluster (no cluster left without a model peak)
- Pipeline runs to completion in practical time thanks to the Rcpp/C++ parallel core, i.e. no computational bottleneck versus the original DecoMetDIA implementation

## Limitations

- Requires centroided mzML input; the workflow explicitly confirms spectra are in centroid mode before loading
- Performance depends on the correctness of the SWATH window definitions (variable-size windows with 1.0 m/z neighbor overlap) and the MS1 feature table quality
- Referenced datasets are metabolomics SWATH-DIA acquisitions (e.g. Metabolomics Workbench ST002796 and ST002798); behavior on other acquisition schemes is not validated here

## Evidence

- [methods] The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution.: "The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution."
- [methods] we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data.: "we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data."
- [intro] spectra deconvolution is essential to relink precursors with fragment ions: "spectra deconvolution is essential to relink precursors with fragment ions"
- [intro] an efficient SWATH-DIA data deconvolution pipeline: "an efficient SWATH-DIA data deconvolution pipeline"
