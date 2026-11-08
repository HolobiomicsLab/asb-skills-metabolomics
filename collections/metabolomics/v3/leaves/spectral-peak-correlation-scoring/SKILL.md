---
name: spectral-peak-correlation-scoring
description: Use when validating deep-learning spectral reconstruction or any denoising
  pipeline on MSI/SCMS data where fidelity must be quantified against ground truth,
  using peak-wise and spatial Pearson correlations and SNR gain (dB) as acceptance
  metrics.
when_to_use_negative:
- Input is already a processed feature table of a single acquisition with no ground-truth
  or noisy-reference counterpart — correlation scoring requires a reference pair.
- Evaluating raw acquisition quality (e.g. FT analysis of transients) before reconstruction;
  this skill judges reconstruction fidelity, not instrument-level acquisition metrics.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MEISTER
  role: Source of the denoising autoencoder/regressor pipeline whose decoder produces
    the reconstructed spectra; also hosts the notebooks (MEISTER_simulation.ipynb,
    MEISTER_eval.ipynb) that execute the correlation and SNR scoring
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks
  role: Provide MEISTER_simulation.ipynb and MEISTER_eval.ipynb that implement the
    transform-to-peak-list and Pearson/SNR metric computation on simulated and experimental
    MSI data
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:ef473de32560d9db5ae7aa86518086ed2a2fe6b2cd356605dca9b517dbbb3018
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis notebooks
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Spectral peak correlation scoring

## Summary

Quantifies reconstruction fidelity of deep-learning-enhanced MSI spectra by transforming clean, noisy, reduced, and reconstructed data into mass spectra and peak intensity lists at ground-truth peak centroids, then computing Pearson correlation-based peak and spatial correlation scores plus SNR gain in dB against the noisy reference.

## When to use

Use when validating deep-learning spectral reconstruction or any denoising pipeline on MSI/SCMS data where fidelity must be quantified against ground truth, using peak-wise and spatial Pearson correlations and SNR gain (dB) as acceptance metrics.

## When NOT to use

- Input is already a processed feature table of a single acquisition with no ground-truth or noisy-reference counterpart — correlation scoring requires a reference pair.
- Evaluating raw acquisition quality (e.g. FT analysis of transients) before reconstruction; this skill judges reconstruction fidelity, not instrument-level acquisition metrics.

## Inputs

- Ground-truth high-resolution mass spectra and peak intensity lists at known centroids
- Noisy reference spectra (full-resolution acquisition with added Gaussian noise)
- Reduced-input spectra reconstructed from short transients (~5% of temporal points)
- Reconstructed spectra from the denoising autoencoder (e.g. MEISTER decoder outputs)

## Outputs

- Peak Pearson correlation score (reconstructed vs ground truth)
- Spatial Pearson correlation score across ion images (reconstructed vs ground truth)
- SNR gain in dB of reconstructed spectra relative to noisy reference
- Metric table (e.g. simulation_reconstruction_metrics.csv) plus trained model weights

## How to apply

First generate the full-quality (ground truth), noisy, reduced-input, and reconstructed datasets, ensuring all four are processed identically so the comparison is fair. Transform each set into mass spectra and peak intensity lists at the same ground-truth peak centroids so peak-to-peak alignment is exact. Compute the Pearson correlation between ground truth and reconstructed peak intensities across the feature list (peak correlation), and separately across ion images to capture spatial fidelity (spatial correlation) — e.g. the article's evaluation used n = 69,847 pixels and n = 574 ion images. Compute SNR gain in dB as the SNR of the reconstructed spectra relative to the noisy reference; acceptance on the simulated MEISTER dataset requires SNR gain >10 dB using only 5% of the data, with high peak and spatial Pearson correlations against ground truth. For experimental SCMS, the same correlation logic applies between full-resolution reference and reconstructed single-cell spectra, where r > 0.95 was expected.

## Related tools

- **MEISTER** (Source of the denoising autoencoder/regressor pipeline whose decoder produces the reconstructed spectra; also hosts the notebooks (MEISTER_simulation.ipynb, MEISTER_eval.ipynb) that execute the correlation and SNR scoring) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks** (Provide MEISTER_simulation.ipynb and MEISTER_eval.ipynb that implement the transform-to-peak-list and Pearson/SNR metric computation on simulated and experimental MSI data) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook multiscale_analysis/MEISTER_simulation.ipynb  # compute peak/spatial Pearson correlations and SNR gain (dB) vs noisy reference
```

## Evaluation signals

- SNR gain of reconstructed vs noisy reference exceeds 10 dB on the simulated MSI dataset using only 5% of the data
- Peak Pearson correlation and spatial Pearson correlation against ground truth are high (near-ground-truth-level fidelity) across the test set (e.g. n = 69,847 pixels, n = 574 ion images in the article's evaluation)
- Experimental single-cell test on 1,000 independent cells yields Pearson r > 0.95 between full-resolution reference and reconstructed spectra
- The metric table (e.g. simulation_reconstruction_metrics.csv) is populated with peak correlation, spatial correlation, and SNR gain columns and saved alongside trained weights
- Reconstructed SNR distributions show the expected fold-improvement over raw FT analysis (e.g. ~100-fold at m/z 813.4838, tenfold at m/z 813.6009)

## Limitations

- Acceptance thresholds (>10 dB SNR gain, near-ground-truth correlations) were demonstrated on a simulated dataset; experimental realizations vary and may fall short of these exact values.
- Peak and spatial correlations depend on the choice of ground-truth peak centroids; misaligned or wrongly selected centroids will distort the correlation score.
- Scores evaluate only reconstruction fidelity at ground-truth centroids — they do not assess chemical annotation accuracy or spatial registration quality.

## Evidence

- [results] On the simulated MSI dataset, the deep-learning reconstruction achieved near-ground-truth-level fidelity, with a gain of more than 10 dB in SNR over the noisy reference while using only 5% of the data.: "Our method showed near-ground-truth-level fidelity with a gain of more than 10 dB in signal-to-noise ratios (SNRs) over the noisy reference using only 5% data."
- [results] Evaluation used test data with n = 69,847 pixels and n = 574 ion images for peak and spatial correlation respectively.: "the test data contain n = 69,847 pixels and n = 574 ion images for peak and spatial correlation, respectively"
- [methods] Transform clean, noisy, reduced and reconstructed sets into mass spectra and peak intensity lists at the ground-truth peak centroids.: [evidence span withheld — anchor status: not_found]
- [readme] MEISTER_simulation.ipynb is the notebook for evaluation of MEISTER models on simulated MSI data sets.: "MEISTER_simulation.ipynb: Evaluation of MEISTER models on simulated MSI data sets."
- [results] Experimental single-cell validation achieved Pearson r > 0.95 on 1,000 independent cells.: [evidence span withheld — anchor status: not_found]
