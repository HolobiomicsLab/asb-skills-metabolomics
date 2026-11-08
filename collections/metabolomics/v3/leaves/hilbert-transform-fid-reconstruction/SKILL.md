---
name: hilbert-transform-fid-reconstruction
description: Use when a 1D frequency-domain NMR spectrum has been partitioned into
  frequency bins and each bin must be converted into complex time-domain FID data
  for downstream time-domain modeling.
when_to_use_negative:
- Input is already time-domain FID data or a processed peak/feature table — the reconstruction
  step assumes frequency-domain binned spectra
- Spectra that are not high-resolution metabolomics 1D NMR spectra — SAND is currently
  tuned only for high-resolution metabolomics spectra and other cases are not yet
  accommodated
- When bin boundaries or apodization have not been established; applying Hilbert/IFT
  to un-binned or un-apodized spectra reintroduces spectral leakage across bin edges
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Hosts the adaptive binning pipeline (binning, baseline removal, raised-cosine
    banding, Hilbert transform, inverse FT) as base functions runnable on test examples
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Implementation language for the callable adaptive binning module (src/adaptive_binning.m)
    wrapping the Hilbert/IFT bin-to-FID conversion
- name: NMRbox
  role: Cloud environment where updated SAND modules must run and pass test examples
    before merge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:15e2e14ded8ab96b744416eae1056a7e9d6293ed34646ad6306e12d8813199af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# hilbert-transform-fid-reconstruction

## Summary

Converts each adaptively binned region of a 1D metabolomics NMR spectrum back into a time-domain FID via a Hilbert transform (to form the analytic/imaginary component) followed by an inverse Fourier transform, as implemented in SAND's adaptive binning stage.

## When to use

Use when a 1D frequency-domain NMR spectrum has been partitioned into frequency bins and each bin must be converted into complex time-domain FID data for downstream time-domain modeling.

## When NOT to use

- Input is already time-domain FID data or a processed peak/feature table — the reconstruction step assumes frequency-domain binned spectra
- Spectra that are not high-resolution metabolomics 1D NMR spectra — SAND is currently tuned only for high-resolution metabolomics spectra and other cases are not yet accommodated
- When bin boundaries or apodization have not been established; applying Hilbert/IFT to un-binned or un-apodized spectra reintroduces spectral leakage across bin edges

## Inputs

- 1D frequency-domain NMR spectrum with mapped ppm axis (e.g. from the SAND repository test examples)
- Adaptively binned spectral regions (nominal 0.002 ppm buckets, slackness 0.99)
- Baseline-corrected, raised-cosine-apodized bin intensities (0.02 ppm edge roll-off)

## Outputs

- Complex per-bin time-domain FID data suitable for SAND time-domain modeling
- Callable binning module (MATLAB, e.g. src/adaptive_binning.m) following SAND base-function interface conventions

## How to apply

First partition the ppm axis of the 1D spectrum into adaptive buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where the criterion is met. Within each binned region, apply minimum-shift baseline removal to correct intensity offsets, then apply a raised-cosine band with 0.02 ppm roll-off at the bin edges to suppress spectral leakage. Next, apply a Hilbert transform to the band-limited bin to generate the imaginary (analytic) component of the signal, and finally apply an inverse Fourier transform to convert the binned region into a complex per-bin time-domain FID. In SAND this sequence is wrapped as a callable MATLAB module (e.g. src/adaptive_binning.m) following the interface conventions of the SAND base functions, and the per-bin FIDs feed SAND's time-domain decomposition (Markov chain Monte Carlo modeling with time-domain subsampling).

## Related tools

- **SAND** (Hosts the adaptive binning pipeline (binning, baseline removal, raised-cosine banding, Hilbert transform, inverse FT) as base functions runnable on test examples) — https://github.com/edisonomics/SAND
- **MATLAB** (Implementation language for the callable adaptive binning module (src/adaptive_binning.m) wrapping the Hilbert/IFT bin-to-FID conversion)
- **NMRbox** (Cloud environment where updated SAND modules must run and pass test examples before merge)

## Evaluation signals

- Run the wrapped module on the SAND repository test examples; acceptance is that the test examples pass ('pass the test before merge')
- Verify each binned region yields a complex (not purely real) time-domain FID after the Hilbert + inverse FT step
- Check bin partition respects nominal 0.002 ppm size with slackness 0.99 and merges of three adjacent bins only where the criterion is met
- Confirm raised-cosine roll-off of 0.02 ppm is present at both edges of each bin region to suppress leakage
- Downstream check: SAND's subsequent decomposition on the reconstructed FIDs should reproduce per-bin spectral content consistent with the original binned intensities

## Limitations

- No quantitative metric is reported for this stage in isolation; validation relies on the SAND test-harness pass/fail criterion
- The MATLAB implementation is slow and not yet parallelizable; a faster C/C++ version is planned
- The current version of SAND is tuned for high-resolution metabolomics spectra; other spectral types are not yet accommodated
- Bin-to-FID conversion quality depends on upstream binning (0.002 ppm nominal size, slackness 0.99) and raised-cosine apodization; errors in those stages propagate into the reconstructed FIDs

## Evidence

- [results] The band-selected bin is processed by Hilbert transform to generate imaginary data and then inverse Fourier-processed to produce an FID for each bin.: [evidence span withheld — anchor status: not_found]
- [methods] Partition the spectrum adaptively into buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where the criterion is met.: [evidence span withheld — anchor status: not_found]
- [methods] Apply a raised-cosine band with 0.02 ppm roll-off at the edges of each bin region to suppress spectral leakage.: [evidence span withheld — anchor status: not_found]
- [methods] Any updtes is expected to run NMRBox environemt and pass the test before merge.: "Any updtes is expected to run NMRBox environemt and pass the test before merge."
- [readme] SAND automatic process and deconvolute 1D NMR spectra into peak tables. Similar workflow works for simulated and expermental datasets.: "SAND automatic process and deconvolute 1D NMR spectra into peak tables. Similar workflow works for simulated and expermental datasets."
