---
name: ion-image-spatial-correlation-analysis
description: Use when evaluating spectral/spatial fidelity of MSI reconstruction methods
  (e.g. MEISTER, FT, linear subspace) against a clean ground-truth dataset with known
  peak centroids.
when_to_use_negative:
- The input is raw mass spectra without an established ground-truth peak centroid
  list, since peak and spatial correlations require matched intensity lists against
  known centroids
- Data lack pixel-level spatial structure (e.g. single-cell spectra without ion images),
  so only peak correlation is meaningful
- Only the noisy reference spectrum is available and no clean/reconstructed alternative
  exists to compare against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Generates the reconstructed transients and transforms them into mass spectra
    and peak intensity lists whose fidelity is scored against ground truth
  repo: https://github.com/richardxie1119/MEISTER
- name: BRAIN algorithm (Python version)
  role: Produces biochemically relevant isotopic distributions for the simulated MSI
    dataset used as the ground truth in the correlation benchmark
- name: MEISTER_simulation.ipynb (multiscale_analysis)
  role: Notebook implementing the simulation setup and the peak/spatial correlation
    evaluation against ground truth
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:d55e1eefd4c11cbab47069a7737999b3a89900edb1499a4365ea9c332a10fb65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - BRAIN algorithm (Python version)
  - MEISTER
  - MEISTER_simulation.ipynb (multiscale_analysis)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ion-image-spatial-correlation-analysis

## Summary

Quantifies reconstruction fidelity in MEISTER mass spectrometry imaging by extracting peak intensity lists at ground-truth peak centroids and computing Pearson correlation-based peak and spatial correlation scores between each method's ion images and ground truth.

## When to use

Use when evaluating spectral/spatial fidelity of MSI reconstruction methods (e.g. MEISTER, FT, linear subspace) against a clean ground-truth dataset with known peak centroids.

## When NOT to use

- The input is raw mass spectra without an established ground-truth peak centroid list, since peak and spatial correlations require matched intensity lists against known centroids
- Data lack pixel-level spatial structure (e.g. single-cell spectra without ion images), so only peak correlation is meaningful
- Only the noisy reference spectrum is available and no clean/reconstructed alternative exists to compare against

## Inputs

- Simulated or experimental MSI transients (e.g. 26,497 pixels x 262,144 temporal points) for ground truth, noisy reference, reduced-input, and reconstructed sets
- Peak centroid list identified from the ground-truth mass spectrum
- Peak intensity lists per pixel for each reconstruction method
- Ground-truth ion images (spatial maps of each peak intensity across pixels)

## Outputs

- Per-method peak correlation and spatial correlation scores (Pearson r) against ground truth
- Comparison table mirroring Extended Data Fig. 2c,d (e.g. simulated_reconstruction_comparison.csv)
- SNR gain (dB) of reconstructed versus noisy reference spectra where applicable

## How to apply

Transform each candidate reconstruction (clean ground truth, noisy reference, reduced-input, and reconstructed transients) into mass spectra, identify peak centroids from the ground-truth spectrum, and extract peak intensity lists at those centroids for every pixel set. Compute Pearson correlation coefficients between each method's peak list and the corresponding ground-truth ion images, aggregating peak and spatial correlation scores per method. In the simulated MEISTER workflow (MEISTER_simulation.ipynb), the benchmark compares standard FT from full transients, a linear subspace approach, and the MEISTER reconstruction across 30 chemical formulas with H+/Na+/K+ adducts and isotopic distributions generated via the Python BRAIN algorithm. Assemble the per-method scores into a comparison table mirroring Extended Data Fig. 2c,d and save it as simulated_reconstruction_comparison.csv.

## Related tools

- **MEISTER** (Generates the reconstructed transients and transforms them into mass spectra and peak intensity lists whose fidelity is scored against ground truth) — https://github.com/richardxie1119/MEISTER
- **BRAIN algorithm (Python version)** (Produces biochemically relevant isotopic distributions for the simulated MSI dataset used as the ground truth in the correlation benchmark)
- **MEISTER_simulation.ipynb (multiscale_analysis)** (Notebook implementing the simulation setup and the peak/spatial correlation evaluation against ground truth) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter nbconvert --to notebook --execute MEISTER_simulation.ipynb --output MEISTER_simulated_reconstruction_comparison.ipynb
```

## Evaluation signals

- Pearson correlation scores between reconstructed peak lists and ground-truth ion images should approach near-ground-truth-level fidelity reported for the simulated dataset
- SNR gain of the MEISTER reconstruction over the noisy reference exceeds 10 dB while using only 5% of the transient data
- Comparison table contains peak and spatial correlation entries for each baseline (standard FT, linear subspace, MEISTER) mirroring Extended Data Fig. 2c,d
- Output CSV (e.g. simulated_reconstruction_comparison.csv) is populated with per-method scores rather than empty or missing columns

## Limitations

- The metric depends on having a reliable ground-truth peak centroid list; mass shifts or inappropriate ppm windows could misalign peaks and distort correlations
- Scores reflect reconstruction fidelity only for peaks present in the ground truth; features outside the simulated 30-formula/adduct set are not evaluated
- The Pearson-based approach assumes linear intensity relationships; nonlinear reconstruction artifacts may not be fully captured

## Evidence

- [other] Transform all transient sets (clean ground truth, noisy reference, noisy reduced 10,000-point transients processed by standard FT/zero-padding, linear-subspace reconstruction, and MEISTER reconstruction) into mass spectra: [evidence span withheld — anchor status: not_found]
- [other] Extract peak intensity lists at peak centroids identified from the ground-truth spectra.: [evidence span withheld — anchor status: not_found]
- [other] Compute peak correlation and spatial correlation scores as Pearson correlation coefficients between each method's peak list and ground-truth ion image pairs: [evidence span withheld — anchor status: not_found]
- [abstract] the deep-learning reconstruction achieved near-ground-truth-level fidelity, with a gain of more than 10 dB in SNR over the noisy reference while using only 5% of the data: "Our method showed near-ground-truth-level fidelity with a gain of more than 10 dB in signal-to-noise ratios (SNRs) over the noisy reference using only 5% data."
- [other] MEISTER reconstruction should show near-ground-truth-level fidelity, consistent with the reported >10 dB SNR gain over the noisy reference using only 5% of the transient data: [evidence span withheld — anchor status: not_found]
