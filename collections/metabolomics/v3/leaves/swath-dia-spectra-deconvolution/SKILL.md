---
name: swath-dia-spectra-deconvolution
description: Use when input is centroid-mode SWATH-DIA mzML data (variable-size windows
  with m/z neighbor overlap) and pseudo-MS2 spectra must be reconstructed by relinking
  MS1 features to their true fragment ions.
when_to_use_negative:
- Input is DDA data with chimeric spectra — use PerformDDADeconvolution instead
- The MS2 spectra are already deconvoluted or come from DDA acquisition where precursor-fragment
  linkage is explicit
- Input is only an MS1 feature table without raw SWATH MS2 windows — deconvolution
  requires the raw sequential MS2 window data
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: Provides PerformDIADeconvolution, the DecoMetDIA-style Rcpp/C++ SWATH-DIA
    deconvolution interface with parallel computing
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR
  role: Loads the centroid mzML data and parses MS1 features and sequential SWATH
    MS2 windows
- name: XCMS
  role: Reference implementation (reconstructChromPeakSpectra) used to benchmark the
    deconvoluted spectra on the same input
- name: R
  role: Environment in which the Rcpp/C++ deconvolution core is invoked
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:f3fbf4e65064cfb05667722b9456e259c8a31062c27a64a464a254a40539b74f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - R
  - XCMS
  - mzR
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# swath-dia-spectra-deconvolution

## Summary

Deconvolves SWATH-DIA MS2 spectra so that co-fragmented precursors are correctly relinked with their fragment ions, following the DecoMetDIA approach re-implemented with an Rcpp/C++ core and parallel computing in MetaboAnalystR 4.0.

## When to use

Use when input is centroid-mode SWATH-DIA mzML data (variable-size windows with m/z neighbor overlap) and pseudo-MS2 spectra must be reconstructed by relinking MS1 features to their true fragment ions.

## When NOT to use

- Input is DDA data with chimeric spectra — use PerformDDADeconvolution instead
- The MS2 spectra are already deconvoluted or come from DDA acquisition where precursor-fragment linkage is explicit
- Input is only an MS1 feature table without raw SWATH MS2 windows — deconvolution requires the raw sequential MS2 window data

## Inputs

- Raw SWATH-DIA metabolomics spectra in centroid mzML format (e.g. Metabolomics Workbench accessions ST002796, ST002798)
- MS1 feature table (e.g. from MetaboAnalystR auto-optimized peak picking) as the target feature list
- SWATH window layout (variable-size windows, ~1.0 m/z neighbor overlap)

## Outputs

- Pseudo-MS2 spectra per MS1 feature (cluster containing the original MS feature), written to an .msp file
- Deconvoluted spectra suitable for downstream consensus building (PerformSpectrumConsenus) and database searching

## How to apply

Load centroid-mode mzML files with the mzR package, parsing the MS1 feature table plus the sequential SWATH MS2 windows (e.g. 10 variable-size windows with 1.0 m/z neighbor overlap and ~0.9 s cycle time). For each target MS1 feature, extract all EICs of MS2 peaks from its corresponding SWATH window, then cluster the EICs by chromatographic peak-shape similarity and retention-time coelution, selecting one model peak per cluster to decompose all EICs. Reconstruct the MS2 cluster composition from the decomposed components and export the cluster containing the original MS1 feature as a pseudo-MS2 spectrum, via the PerformDIADeconvolution interface backed by the Rcpp/C++ DecoMetDIA-style algorithm with parallel computing to overcome the original implementation's computational bottleneck. Benchmark the output against XCMS's reconstructChromPeakSpectra on the same input and write the deconvoluted spectra to an .msp file.

## Related tools

- **MetaboAnalystR 4.0** (Provides PerformDIADeconvolution, the DecoMetDIA-style Rcpp/C++ SWATH-DIA deconvolution interface with parallel computing) — https://github.com/xia-lab/MetaboAnalystR
- **mzR** (Loads the centroid mzML data and parses MS1 features and sequential SWATH MS2 windows)
- **XCMS** (Reference implementation (reconstructChromPeakSpectra) used to benchmark the deconvoluted spectra on the same input)
- **R** (Environment in which the Rcpp/C++ deconvolution core is invoked)

## Examples

```
res <- PerformDIADeconvolution(mSetObj, format="mzML", file="swath_centroid.mzML", ncores=4)
```

## Evaluation signals

- The cluster containing each original MS1 feature is exported as a pseudo-MS2 spectrum in the .msp output
- Deconvoluted spectra are consistent with XCMS reconstructChromPeakSpectra run on the same SWATH-DIA dataset
- Downstream MS2-based identification after deconvolution exceeds the non-deconvoluted baseline (~5% more identifications reported in the benchmark)
- All SWATH windows are parsed correctly (e.g. 10 variable-size windows with 1.0 m/z neighbor overlap, ~0.9 s cycle time) before EIC clustering
- Exactly one model peak is selected per EIC cluster and used to decompose all EICs

## Limitations

- SWATH-DIA deconvolution is specific to SWATH-style DIA acquisitions with defined sequential windows; other DIA schemes are not covered by this card
- The DecoMetDIA-style algorithm is computationally intensive; it was re-implemented in Rcpp/C++ with parallel computing specifically to address this bottleneck, but large datasets may still be slow
- Deconvolution quality depends on chromatographic peak-shape similarity and RT coelution clustering; co-eluting isomers with near-identical peak shapes may be mis-relinked

## Evidence

- [methods] The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution.: "The deconvolution of SWATH-DIA data can be achieved with the function, PerformDIADeconvolution."
- [intro] DIA deconvolution relinks precursors: "spectra deconvolution is essential to relink precursors with fragment ions"
- [methods] we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data.: "we utilized the function, reconstructChromPeakSpectra, to deconvolve SWATH-DIA data."
- [readme] README describes the MS/MS deconvolution module scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] README benchmark claim on identification gains: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
