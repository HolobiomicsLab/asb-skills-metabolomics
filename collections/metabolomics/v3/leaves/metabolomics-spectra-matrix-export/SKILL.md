---
name: metabolomics-spectra-matrix-export
description: Use when a set of calibrated 1D 1H NMR spectra (Bruker/Agilent FID, Bruker
  1r files, or nmrML) has been baseline-corrected and realigned across selected ppm
  areas in NMRProcFlow and a spectral data matrix must be produced for statistical
  analysis.
when_to_use_negative:
- The input spectra are 2D NMR or non-NMR metabolomics data — NMRProcFlow is dedicated
  to 1D spectra processing (1H & 13C) for NMR-based metabolomics.
- The FID spectra have not yet been baseline-corrected and ppm-scale calibrated —
  the matrix export presumes the upstream processing stages in NMRspec are complete.
- The goal is quantification of crowded regions with heavy peak overlap via local
  signal deconvolution — the article notes such a method is only planned for a future
  version of NMRProcFlow, so the exported matrix will not provide it.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRProcFlow
  role: GUI tool whose interactive session performs the ppm-area realignment (Least-Square
    or Parametric Time Warping) and from which the realigned spectra are exported
    as the data matrix
  repo: https://github.com/INRA/NMRProcFlow
- name: NMRspec
  role: Spectra processing application (implemented mainly in R, with C++ algorithms
    via Rcpp in Rnmr1D) in which calibration, baseline correction, realignment and
    binning run before export
- name: NMRviewer
  role: Client–server spectra viewer used to re-inspect the exported spectra and verify
    resonances are aligned across the selected ppm areas
- name: R
  role: Language in which NMRspec is implemented; hosts the Rnmr1D processing module
    that executes the realignment before matrix export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:4fa581b43e0c1c510bb4670a943ef4def4d62061a1ab63ac06d9908b3efa75db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - NMRspec
  - NMRviewer
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolomics-spectra-matrix-export

## Summary

Exports realigned, bucketed 1D 1H NMR spectra from NMRProcFlow's NMRspec application as a spectral data matrix (e.g. aligned_spectra.csv), the endpoint of the interactive processing chain (ppm calibration, baseline correction, realignment, binning) and the bridge to downstream metabolomics statistics.

## When to use

Use when a set of calibrated 1D 1H NMR spectra (Bruker/Agilent FID, Bruker 1r files, or nmrML) has been baseline-corrected and realigned across selected ppm areas in NMRProcFlow and a spectral data matrix must be produced for statistical analysis.

## When NOT to use

- The input spectra are 2D NMR or non-NMR metabolomics data — NMRProcFlow is dedicated to 1D spectra processing (1H & 13C) for NMR-based metabolomics.
- The FID spectra have not yet been baseline-corrected and ppm-scale calibrated — the matrix export presumes the upstream processing stages in NMRspec are complete.
- The goal is quantification of crowded regions with heavy peak overlap via local signal deconvolution — the article notes such a method is only planned for a future version of NMRProcFlow, so the exported matrix will not provide it.

## Inputs

- Set of calibrated 1D 1H NMR spectra loaded into NMRspec (Bruker or Agilent/Varian FID, Bruker 1r folder structure, or nmrML format)
- Spectra already baseline-corrected and ppm-scale calibrated within NMRProcFlow
- User-selected ppm areas exhibiting uncontrolled chemical-shift variations between spectra

## Outputs

- aligned_spectra.csv — data matrix of realigned spectra with one spectral row per input spectrum
- Bucket table (Intelligent, variable-size, or fixed-width bins), optionally filtered by signal-to-noise ratio at the output step

## How to apply

In NMRProcFlow's NMRspec application (R GUI, with C++ processing algorithms in the Rnmr1D module via Rcpp), first complete the upstream chain: ppm-scale calibration, local or full baseline correction, then realignment of the user-selected ppm areas whose resonances show uncontrolled chemical-shift variation between spectra. For each area, apply the Least-Square realignment algorithm; where regions contain intertwined peaks and Least-Square does not fit, switch to Parametric Time Warping (Bloemberg et al. 2010), which performs well provided all spectra contain the same number of peaks in that region. Bin the realigned spectra using Intelligent (Adaptive Intelligent Binning), variable-size, or fixed-width binning to define the matrix columns, and optionally filter buckets on signal-to-noise ratio at the output step. Export the realigned spectra as a data matrix saved to aligned_spectra.csv, with one spectral row per input spectrum. Validate by confirming the CSV has the same number of spectral rows as the input set and by re-inspecting the overlaid spectra in the NMR spectra viewer to confirm resonances within the selected ppm areas are aligned across spectra.

## Related tools

- **NMRProcFlow** (GUI tool whose interactive session performs the ppm-area realignment (Least-Square or Parametric Time Warping) and from which the realigned spectra are exported as the data matrix) — https://github.com/INRA/NMRProcFlow
- **NMRspec** (Spectra processing application (implemented mainly in R, with C++ algorithms via Rcpp in Rnmr1D) in which calibration, baseline correction, realignment and binning run before export)
- **NMRviewer** (Client–server spectra viewer used to re-inspect the exported spectra and verify resonances are aligned across the selected ppm areas)
- **R** (Language in which NMRspec is implemented; hosts the Rnmr1D processing module that executes the realignment before matrix export)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- aligned_spectra.csv exists and contains exactly the same number of spectral rows as the number of input spectra in the set.
- On re-inspection in the NMR spectra viewer, resonances within every selected ppm area are aligned across all spectra.
- Each realigned region retains the same number of peaks per spectrum — the precondition under which Parametric Time Warping (Bloemberg et al. 2010) yields better results.
- The bucket columns of the exported matrix reflect the chosen binning method (Intelligent, variable size or fixed width) and any SNR-based bucket filter applied at the output step.
- The ppm scale from the earlier calibration step is preserved in the exported matrix, consistent with what was inspected in the viewer.

## Limitations

- Parametric Time Warping produces a better result only provided all spectra have the same amount of peaks in the realigned region; mismatched peak counts break the alignment.
- The Least-Square realignment does not fit spectral regions with intertwined peaks, so those regions require the Parametric Time Warping route instead.
- Automated batch-mode 1D NMR spectra processing workflows regarded as a black box have not yet reached full maturity for non-expert users; interactive processing with the spectra viewer is still the best approach, so blind batch export without visual inspection is discouraged.
- Quantification based on local signal deconvolution for crowded regions with heavy peak overlap is planned only in a future version of NMRProcFlow and is not available at export time.

## Evidence

- [abstract] covers all spectra processing steps including baseline correction, chemical shift calibration and alignment.: "covers all spectra processing steps including baseline correction, chemical shift calibration and alignment."
- [intro] re-alignment of resonances having uncontrolled variations in chemical shifts between spectra: "re-alignment of resonances having uncontrolled variations in chemical shifts between spectra"
- [methods] NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R: "NMRspec, the NMR spectra processing application has been implemented using mainly the open source software language R"
- [results] in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step.: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [readme] The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application.: "The 'NMR spectra viewer' is the central tool of NMRProcFlow and the core of the application."
- [readme] the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width): "the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
