---
name: ft-spectral-transformation-and-peak-centroid-extraction
description: Use when you need to convert time-domain FT-ICR transients into mass
  spectra and extract peak intensity lists at defined m/z centroids to quantitatively
  compare reconstruction methods (deep-learning, linear subspace, standard FT) on
  MSI data.
when_to_use_negative:
- Input is already a processed peak table or centroided ion image with no time-domain
  transient available — the Fourier transform and centroid extraction are unnecessary.
- The goal is to evaluate reconstruction quality on real experimental data where no
  ground-truth spectrum exists (use MEISTER_eval.ipynb with experimental data instead
  of simulation-based comparison).
- Working with low-resolution or non-FT instruments (e.g. TOF or Orbitrap) where FT-ICR
  transient-to-spectrum transformation is not applicable.
edam_operation: http://edamontology.org/operation_3620
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: MEISTER
  role: Framework that produces the reconstructed transients and provides the computational
    pipeline (including simulation evaluation notebook) in which FT transformation
    and peak extraction are executed
  repo: https://github.com/richardxie1119/MEISTER
- name: BRAIN algorithm (Python version)
  role: Generates isotopic distributions of chemical formulas used to define ground-truth
    peak centroids for simulated MSI data
- name: MEISTER_simulation.ipynb (multiscale_analysis)
  role: Notebook that orchestrates transient loading, spectral transformation, centroid
    extraction, and correlation-based method comparison on simulated MSI data
  repo: https://github.com/richardxie1119/multiscale_analysis
- name: MALDI FT-ICR mass spectrometer
  role: Instrument that acquires the raw time-domain transient data transformed by
    this skill
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:d979f0e4f4ddb1c8870eac591aa59489e25479d9580e7007154661eb78383a1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - BRAIN algorithm (Python version)
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  - MEISTER_simulation.ipynb (multiscale_analysis)
  edam_operation: http://edamontology.org/operation_3620
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ft-spectral-transformation-and-peak-centroid-extraction

## Summary

Transforms raw FT-ICR transient data (e.g. 262,144-point time-domain signals from MALDI MSI) into mass spectra via Fourier transform with zero-padding, then extracts peak intensities at centroids identified from ground-truth or reference spectra for quantitative comparison of reconstruction methods.

## When to use

Use when you need to convert time-domain FT-ICR transients into mass spectra and extract peak intensity lists at defined m/z centroids to quantitatively compare reconstruction methods (deep-learning, linear subspace, standard FT) on MSI data.

## When NOT to use

- Input is already a processed peak table or centroided ion image with no time-domain transient available — the Fourier transform and centroid extraction are unnecessary.
- The goal is to evaluate reconstruction quality on real experimental data where no ground-truth spectrum exists (use MEISTER_eval.ipynb with experimental data instead of simulation-based comparison).
- Working with low-resolution or non-FT instruments (e.g. TOF or Orbitrap) where FT-ICR transient-to-spectrum transformation is not applicable.

## Inputs

- Raw FT-ICR time-domain transients (e.g. .h5/.hdf5 or raw acquisition files, up to 262,144 points per pixel transient)
- Ground-truth or reference peak centroid list (m/z values with isotopic/adduct assignments)
- Reconstructed or noisy spectra to compare (e.g. standard FT, linear subspace, MEISTER deep-learning reconstruction)

## Outputs

- Per-pixel mass spectra from transient Fourier transform with zero-padding
- Peak intensity lists at ground-truth centroids for each spectrum/method
- Peak and spatial Pearson correlation scores per method (e.g. simulated_reconstruction_comparison.csv)

## How to apply

Load raw transients (e.g. the simulated MSI dataset with 30 chemical formulas, H+/Na+/K+ adducts, isotopic distributions via Python BRAIN, 26,497 pixels × 262,144 temporal points, or MEISTER-reconstructed 10,000-point reduced transients) and apply the standard Fourier transform with zero-padding to generate mass spectra per pixel. Separately, identify peak centroids (m/z values) from ground-truth spectra using the same isotopic/adduct definitions, then extract the peak intensity list at each centroid from every method's spectrum. The 3-ppm m/z binning step should be applied to align peaks affected by potential mass shift before further comparison. Compare methods by computing Pearson correlation between each method's peak list and ground-truth ion image pairs (peak and spatial correlation scores), as done in MEISTER_simulation.ipynb. Validation: near-ground-truth-level fidelity should yield >10 dB SNR gain over the noisy reference when using only 5% of transient data.

## Related tools

- **MEISTER** (Framework that produces the reconstructed transients and provides the computational pipeline (including simulation evaluation notebook) in which FT transformation and peak extraction are executed) — https://github.com/richardxie1119/MEISTER
- **BRAIN algorithm (Python version)** (Generates isotopic distributions of chemical formulas used to define ground-truth peak centroids for simulated MSI data)
- **MEISTER_simulation.ipynb (multiscale_analysis)** (Notebook that orchestrates transient loading, spectral transformation, centroid extraction, and correlation-based method comparison on simulated MSI data) — https://github.com/richardxie1119/multiscale_analysis
- **MALDI FT-ICR mass spectrometer** (Instrument that acquires the raw time-domain transient data transformed by this skill)

## Examples

```
python -m MEISTER_simulation --input transients.h5 --ground_truth centroids.csv --method standard_ft,linear_subspace,meister --output simulated_reconstruction_comparison.csv
```

## Evaluation signals

- Peak intensity lists must be extracted at centroids defined from ground-truth spectra, not from each method's own peaks, to ensure fair comparison across methods.
- Pearson correlation (peak and spatial) between each method's output and ground-truth ion images should reflect the expected ranking: deep-learning reconstruction > linear subspace > standard FT from full transients (which suffers denoising loss).
- When using only 5% of transient data, the deep-learning reconstruction should show a signal-to-noise gain of >10 dB over the noisy reference while remaining near ground-truth fidelity.
- m/z binning in 3-ppm increments must be applied before comparing peak lists to account for mass shifts between acquisitions.
- The comparison output should assemble per-method peak and spatial correlation scores into a table mirroring Extended Data Fig. 2c,d (e.g. saved as simulated_reconstruction_comparison.csv).

## Limitations

- Mass shifts between acquisitions can misalign peak centroids, requiring the 3-ppm binning step; without it, extraction may miss true peaks.
- The skill's validation workflow is grounded in simulated MSI data with known ground truth; applying it to experimental data without ground truth requires alternative evaluation (e.g. SNR distributions as in MEISTER_eval.ipynb).
- Reconstruction quality depends on the fraction of transient data available; results degrade when input transients are extremely short or of very low signal-to-noise.

## Evidence

- [methods] Transform all transient sets (clean ground truth, noisy reference, noisy reduced 10,000-point transients processed by standard FT/zero-padding, linear-subspace reconstruction, and MEISTER reconstruction) into mass spectra.: [evidence span withheld — anchor status: not_found]
- [methods] Extract peak intensity lists at peak centroids identified from the ground-truth spectra.: [evidence span withheld — anchor status: not_found]
- [methods] Compute peak correlation and spatial correlation scores as Pearson correlation coefficients between each method's peak list and ground-truth ion image pairs.: [evidence span withheld — anchor status: not_found]
- [methods] The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [readme] MEISTER_simulation.ipynb: Evaluation of MEISTER models on simulated MSI data sets.: "MEISTER_simulation.ipynb: Evaluation of MEISTER models on simulated MSI data sets."
