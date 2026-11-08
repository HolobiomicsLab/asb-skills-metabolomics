---
name: mass-spectra-peak-centroiding
description: Use when raw profile-mode spectra or transients from MALDI FT-ICR MS
  (or reconstructed high-resolution equivalents) must be reduced to peak lists before
  m/z binning, TIC normalization, feature counting, or SNR comparison.
when_to_use_negative:
- When the input is already a centroided peak/imzML feature table (e.g., peak_data_sc_regional_decode.imzML)
  — binning and normalization, not centroiding, is the next step
- When spectra come from ion-trap or orbitrap instruments where MEISTER's transient-reconstruction
  model was not trained and peak positions/patterns differ substantially
- When the goal is spectral visualization rather than feature extraction, since centroiding
  discards isotopic fine structure in profile data
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Deep-learning regressor-decoder pipeline that reconstructs high-resolution
    transients from low-resolution FT-ICR data; the reconstructed spectra then undergo
    peak centroid extraction as part of SNR evaluation
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Instrument generating the raw profile transients (64k/1M points) that are
    transformed to spectra and centroided
- name: ftms Control (v.2.1.0, Bruker)
  role: Vendor acquisition software whose output transients are the centroiding input;
    also drives target automation via microMS coordinate files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:a60f6665c8ec44f16dd6bd4bdaeb06e05ee7a9b2290dbe409241ff9f5cf4b2a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  - ftms Control (v.2.1.0, Bruker)
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mass-spectra-peak-centroiding

## Summary

Converts continuous FT-ICR transient-based mass spectra into discrete peak lists by detecting local maxima above noise and reporting centroid m/z positions, which is a prerequisite for downstream feature alignment, SNR quantification, and cross-annotation in the MEISTER multiscale MSI/SCMS pipeline.

## When to use

Use when raw profile-mode spectra or transients from MALDI FT-ICR MS (or reconstructed high-resolution equivalents) must be reduced to peak lists before m/z binning, TIC normalization, feature counting, or SNR comparison.

## When NOT to use

- When the input is already a centroided peak/imzML feature table (e.g., peak_data_sc_regional_decode.imzML) — binning and normalization, not centroiding, is the next step
- When spectra come from ion-trap or orbitrap instruments where MEISTER's transient-reconstruction model was not trained and peak positions/patterns differ substantially
- When the goal is spectral visualization rather than feature extraction, since centroiding discards isotopic fine structure in profile data

## Inputs

- Raw transient time-domain data from MALDI FT-ICR MS (64k low-resolution or 1M high-resolution .zip archives from Illinois Data Bank 10.13012/B2IDB-9740536_V1 / MassIVE task 2a823e133be648b2b0ae982f40e7f41b)
- Deep-learning-reconstructed high-resolution transients from MEISTER decoder output
- Zero-padded reduced-transient spectra and high-resolution reference average spectra for peak position anchoring

## Outputs

- Peak list of centroid m/z values with apex intensities per pixel/cell
- Per-peak SNR table (e.g., snr_comparison.csv) for MEISTER-reconstructed vs. FT-processed raw data
- Binned feature matrix (3-ppm m/z bins) ready for TIC normalization and downstream UMAP/registration analysis

## How to apply

Feed each transient (e.g., 64k low-resolution rat-brain transients or the 1M-point high-resolution reference) through the Fourier transform to obtain a mass spectrum, then detect peaks as local maxima exceeding the local noise floor; record centroid m/z and apex intensity. In MEISTER this peak extraction follows the FT transformation of reconstructed, zero-padded, and reference transients, and peak lists are subsequently processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift. Peaks are used for per-pixel TIC normalization, feature prevalence filtering (features present in <5% of cells discarded), and cross-annotation of SCMS data against tissue MSI features searched against LIPID MAPS with a ±0.005 m/z (≈3 ppm) threshold. SNR per peak is computed as signal intensity divided by the noise standard deviation measured over a signal-free spectral region, which requires a reliable centroid m/z coordinate for each peak. Correct application is judged by peak positions matching reference centroid values (e.g., m/z 813.4838 and m/z 813.6009) within the 3-ppm binning window and by reproduced SNR gains over traditional FT analysis.

## Related tools

- **MEISTER** (Deep-learning regressor-decoder pipeline that reconstructs high-resolution transients from low-resolution FT-ICR data; the reconstructed spectra then undergo peak centroid extraction as part of SNR evaluation) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Instrument generating the raw profile transients (64k/1M points) that are transformed to spectra and centroided)
- **ftms Control (v.2.1.0, Bruker)** (Vendor acquisition software whose output transients are the centroiding input; also drives target automation via microMS coordinate files)

## Examples

```
python -c "import numpy as np; from meister.eval import transform_to_spectrum, centroid_peaks; peaks = centroid_peaks(transform_to_spectrum(reconstructed_transient), noise_region=(0,100))"
```

## Evaluation signals

- Centroid m/z values of known lipids (e.g., m/z 813.4838, m/z 813.6009) fall within the 3-ppm binning tolerance across serial sections
- Per-peak SNR of MEISTER-reconstructed spectra exceeds raw FT-processed data by >10× (e.g., 100-fold at m/z 813.4838)
- Number of features per cell after centroiding + 5% prevalence filter matches reported count (13,566 cells profiled; features in <5% of cells discarded)
- Peak intensity at reference centroid positions in reconstructed data correlates with the high-resolution reference average spectra used for peak extraction

## Limitations

- Mass shifts between acquired and reference spectra can falsely cross-annotate features if the ppm window is inappropriate; m/z binning in 3-ppm increments is the paper's mitigation
- Centroiding quality depends on the transient length — low-resolution 64k transients must be reconstructed by MEISTER before peak positions reach reference fidelity
- SNR estimation requires a signal-free region; peaks in crowded regions may have noise floors contaminated by neighboring signals

## Evidence

- [results] peak extraction after FT transformation of reconstructed data: [evidence span withheld — anchor status: not_found]
- [methods] 3-ppm m/z binning of peak lists: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [results] SNR computation against signal-free noise region: [evidence span withheld — anchor status: not_found]
- [results] reported SNR gain over raw FT data: "Reconstruction improved SNRs as shown in SNR distributions for signals at m/z 813.4838 (top, 100-fold higher)"
