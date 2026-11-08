---
name: dda-spectral-deconvolution
description: Use when processing centroided DDA mzML data in which many MS2 spectra
  are chimeric (multiple ions co-isolated in the isolation window, 'over half' per
  the paper) and clean, precursor-specific MS2 spectra are needed for downstream library
  search.
when_to_use_negative:
- 'Data are SWATH-DIA: use the DIA/SWATH deconvolution path (PerformDIADeconvolution
  / reconstructChromPeakSpectra) instead, which relinks precursors with fragment ions'
- MS2 spectra are already clean/consensus spectra (e.g., after PerformSpectrumConsenus
  frequency-based filtering) — deconvolution targets chimeric DDA acquisitions
- Only LC-MS1 feature detection and quantification is needed; that is handled by the
  auto-optimized feature detection module, not DDA deconvolution
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboAnalystR 4.0
  role: Provides the PerformDDADeconvolution function that implements chimeric-spectrum
    detection, candidate-spectrum construction, and elastic-net deconvolution
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment in which the DDA deconvolution workflow runs as part
    of the unified LC-MS pipeline
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: mzR / ProteoWizard
  role: Used to load centroided mzML raw MS2 data prior to feature assignment and
    deconvolution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:2e925b8d41ef3e8261ceac1c342346ba77b3d25f922c10462cfe527ea4d6c503
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - mzR
  - mzR / ProteoWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# dda-spectral-deconvolution

## Summary

MetaboAnalystR 4.0's auto-optimized DDA chimeric-spectra deconvolution: it assigns MS2 spectra to MS1 features, detects chimeric acquisitions, and uses a self-tuned elastic-net regression over candidate component spectra to extract a clean spectrum for the main (precursor) ion.

## When to use

Use when processing centroided DDA mzML data in which many MS2 spectra are chimeric (multiple ions co-isolated in the isolation window, 'over half' per the paper) and clean, precursor-specific MS2 spectra are needed for downstream library search.

## When NOT to use

- Data are SWATH-DIA: use the DIA/SWATH deconvolution path (PerformDIADeconvolution / reconstructChromPeakSpectra) instead, which relinks precursors with fragment ions
- MS2 spectra are already clean/consensus spectra (e.g., after PerformSpectrumConsenus frequency-based filtering) — deconvolution targets chimeric DDA acquisitions
- Only LC-MS1 feature detection and quantification is needed; that is handled by the auto-optimized feature detection module, not DDA deconvolution

## Inputs

- Centroided mzML raw data (DDA acquisition)
- Aligned MS1 feature table (peak m/z and retention time per feature)
- User-defined isolation-window and acquisition-threshold parameters

## Outputs

- Deconvoluted, normalized MS2 spectrum per chimeric feature (assigned to the main ion)
- Original 'Spectrum 0' retained/exported for features where regression yields no usable fragments
- Parseable deconvolution output file produced by PerformDDADeconvolution

## How to apply

Load centroided mzML raw data plus the aligned MS1 feature table and assign each MS2 spectrum to its target feature by precursor m/z (ppm tolerance) and retention time (rt_tol), merging replicate spectra MZmine-style. Flag a spectrum as chimeric when multiple centroids fall inside the mz_med-centered isolation window and any intensity exceeds the user-defined acquisition threshold, labeling the target centroid as 'main ion' and others as 'contamination ions'. Build candidate component spectra from four sources: DecoID-style prediction of orphan isotopologues from the MS1 scan, clean spectra of the one or two nearest MS2 scans, the 'Complete library' spectrum with highest dot-product/spectral-entropy similarity to chimeric 'Spectrum 0', and network-predicted spectra of unknown ions (NetID rules, non-propagatable network, with a 0–10 similarity penalty and removal of fragments whose element composition contradicts the unknown ion's formula). Fit a penalized elastic-net regression by permuting 110 alpha–lambda combinations (11 alpha values 0–1 in 0.1 steps, 10 lambda values from response-component correlations) and keep the minimal-residue solution (Solution 0), optimizing alpha and lambda per individual peak. Remove contamination-ion fragments using their beta coefficients, normalize remaining fragments, and export the deconvoluted spectrum; if no fragment remains or the main ion's beta is 0, retain and export the original 'Spectrum 0'.

## Related tools

- **MetaboAnalystR 4.0** (Provides the PerformDDADeconvolution function that implements chimeric-spectrum detection, candidate-spectrum construction, and elastic-net deconvolution) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment in which the DDA deconvolution workflow runs as part of the unified LC-MS pipeline) — https://github.com/xia-lab/MetaboAnalystR
- **mzR / ProteoWizard** (Used to load centroided mzML raw MS2 data prior to feature assignment and deconvolution)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T); mSet <- PerformDDADeconvolution(mSet)
```

## Evaluation signals

- Every chimeric feature yields either a normalized deconvoluted spectrum or a retained original 'Spectrum 0' — no feature is dropped
- Output file is parseable and was produced by the PerformDDADeconvolution routine
- Regression converged over the 110 alpha–lambda permutations with the minimal-residue solution (Solution 0) selected per peak
- Contamination-ion fragments are removed per their beta coefficients and remaining fragments are normalized
- Spectra with no remaining fragment or main-ion beta of 0 fall back to the original Spectrum 0 rather than an empty output

## Limitations

- If no fragment remains after beta-coefficient filtering or the main ion's beta is 0, only the original chimeric 'Spectrum 0' can be exported
- Candidate Spectrum IV depends on abiotic/bio-transformation network neighbors (NetID rules) and a similarity penalty, so deconvolution of truly unknown ions may be limited
- Built-in MS2 reference libraries are compiled from various instruments and experimental conditions, resulting in potentially varying qualities; no per-compound instrument/condition metadata is pinned
- No R version is pinned for the environment (README recommends R base > 3.6.1; compatibility of v4.0.0 was 'under evaluation')

## Evidence

- [methods] The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution.: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [intro] Objective: add an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra: "an auto-optimized DDA data deconvolution workflow to deal with chimeric spectra"
- [intro] over half of them are chimeric and need to be deconvolved: "over half of them are chimeric and need to be deconvolved"
- [readme] a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA): "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [methods] devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T): "devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)"
