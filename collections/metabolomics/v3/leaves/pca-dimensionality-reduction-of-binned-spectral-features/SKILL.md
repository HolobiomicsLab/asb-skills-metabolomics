---
name: pca-dimensionality-reduction-of-binned-spectral-features
description: Use when you have ≥30 MS/MS spectra per compound from spectral libraries
  (e.g. NIST 17, MoNA) with instrument-type and collision-energy metadata and want
  to inspect whether spectra cluster by acquisition conditions before training a fingerprint
  model.
when_to_use_negative:
- The dataset already exists as a feature table of intensity values per compound,
  so no spectrum-to-bin conversion is needed.
- You need absolute quantification of specific peaks rather than an exploratory similarity/clustering
  view across many spectra.
- Spectra were acquired with widely differing mass ranges or de-isotoping settings
  such that integer-bin accumulation would merge chemically unrelated peaks.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Performs PCA dimensionality reduction on the binned spectral feature matrix
    before clustering.
- name: MoNA
  role: Source of downloaded MS/MS spectra with instrument and collision-energy metadata
    used to build the binned feature matrix.
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Second spectral library from which training spectra (≥30 per compound, 916
    compounds total) were obtained.
- name: InChIKey
  role: Compound identifier used to merge multiple MS/MS spectra with different collision
    energies acquired from the same compound.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:ae5a6ba30e5c0ea44c09fe3b9db621868064ac9ad189d7e7f1fecf6a21dd3a36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MoNA
  - NIST 17
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pca-dimensionality-reduction-of-binned-spectral-features

## Summary

Reduce a library of MS/MS spectra (HCD, QqQ, Q-TOF, IT) to a 2-D PCA map after binning m/z into integer bins, to reveal how instrument type and collision energy drive clustering; in MetFID this supported choosing a 30 eV low/high collision-energy split.

## When to use

Use when you have ≥30 MS/MS spectra per compound from spectral libraries (e.g. NIST 17, MoNA) with instrument-type and collision-energy metadata and want to inspect whether spectra cluster by acquisition conditions before training a fingerprint model.

## When NOT to use

- The dataset already exists as a feature table of intensity values per compound, so no spectrum-to-bin conversion is needed.
- You need absolute quantification of specific peaks rather than an exploratory similarity/clustering view across many spectra.
- Spectra were acquired with widely differing mass ranges or de-isotoping settings such that integer-bin accumulation would merge chemically unrelated peaks.

## Inputs

- MS/MS spectra (peak list: m/z, intensity) with instrument type (HCD, QqQ, Q-TOF, IT) and collision energy metadata
- Compound identifier (InChIKey) linking spectra from the same compound
- Spectral library entries from MoNA or NIST 17

## Outputs

- PCA cluster plot annotated by instrument type and collision energy (e.g. pca_cluster_plot.png)
- Per-spectrum cluster assignments (e.g. cluster_assignments.csv)

## How to apply

Collect spectra that share compound identity (merged by InChIKey), keeping instrument type (HCD, QqQ, Q-TOF, IT) and collision energy per spectrum. Scale peak intensities to the 0–100 range, drop spectra with fewer than five peaks above 2% relative intensity, and bin each spectrum's m/z range into continuous integer bins using accumulated intensities as feature values. Reduce the binned feature matrix with PCA (e.g. scikit-learn) and cluster/annotate the result by instrument type, applying a 30 eV cutoff when separating low- vs high-collision-energy spectra. Export the PCA cluster figure and per-spectrum cluster assignments so the map can be re-inspected. The rationale is that collision-cell instruments (HCD, QqQ) produce similar fragmentation whereas IT fragments via a different mechanism, so the PCA map reveals whether acquisition conditions dominate spectral similarity.

## Related tools

- **scikit-learn** (Performs PCA dimensionality reduction on the binned spectral feature matrix before clustering.)
- **MoNA** (Source of downloaded MS/MS spectra with instrument and collision-energy metadata used to build the binned feature matrix.) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Second spectral library from which training spectra (≥30 per compound, 916 compounds total) were obtained.)
- **InChIKey** (Compound identifier used to merge multiple MS/MS spectra with different collision energies acquired from the same compound.)

## Examples

```
python3 main.py  # from MetFID folder, after placing MetFID_CNN_40088_5618.h5 in the root directory
```

## Evaluation signals

- Spectra acquired by HCD and QqQ (both collision-cell instruments) fall into the same cluster in the PCA map.
- Spectra acquired by IT also cluster with spectra from collision-cell instruments despite the different fragmentation mechanism.
- The 30 eV cutoff cleanly separates low- vs high-collision-energy spectra within each cluster view.
- Every retained spectrum has at least five peaks with relative intensity above 2%; fewer than that were removed before PCA.
- Peak intensities are scaled to the 0–100 range before binning; no spectrum exceeds this range.

## Limitations

- Collision energy was binned into a coarse low/high split (30 eV cutoff), so finer energy-dependent spectral variation is collapsed.
- IT spectra cluster with collision-cell spectra even though IT uses a completely different fragmentation mechanism, so cluster membership alone does not prove mechanistic similarity.
- The analysis was restricted to spectra acquired in positive mode and to instruments using collision cell or ion trap; other acquisition modes are not covered.

## Evidence
