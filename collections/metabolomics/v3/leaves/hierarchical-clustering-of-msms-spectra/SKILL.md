---
name: hierarchical-clustering-of-msms-spectra
description: Use when a set of curated MS/MS spectra (e.g., NIST 17 or MoNA entries)
  carries instrument type and collision-energy metadata and the goal is to determine
  whether spectral similarity is driven by acquisition conditions (HCD, QqQ, Q-TOF,
  IT) rather than by the underlying compounds.
when_to_use_negative:
- When the input is a feature table of already-derived spectral descriptors rather
  than raw MS/MS peak lists — binning and PCA must run on the m/z intensity vectors.
- When spectra lack instrument type or collision energy metadata — cluster patterns
  cannot be interpreted as the paper does (30 eV cutoff, collision-cell vs IT).
- When the aim is fingerprint prediction or candidate ranking for metabolite annotation
  (MetFID's primary goal) rather than diagnosing acquisition-driven spectral clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MoNA
  role: Source of MS/MS spectra libraries with instrument and collision-energy metadata
    to be clustered
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Source of MS/MS spectra used in the paper's cluster analysis of 916 compounds
- name: InChIKey
  role: Compound identifier used to merge spectra acquired from the same compound
    at different collision energies before clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:f1f64294a3fe6bd6dbf326f81b39085a30ec2d54684be70dd71b90b8f95eb26d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MoNA
  - NIST 17
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hierarchical-clustering-of-msms-spectra

## Summary

Cluster MS/MS spectra by instrument type and collision energy using PCA dimensionality reduction on binned m/z intensity vectors followed by a clustering method (K-means) with spectral-acquisition metadata as labels, to reveal how spectral similarity groups by acquisition conditions rather than by chemistry.

## When to use

Use when a set of curated MS/MS spectra (e.g., NIST 17 or MoNA entries) carries instrument type and collision-energy metadata and the goal is to determine whether spectral similarity is driven by acquisition conditions (HCD, QqQ, Q-TOF, IT) rather than by the underlying compounds.

## When NOT to use

- When the input is a feature table of already-derived spectral descriptors rather than raw MS/MS peak lists — binning and PCA must run on the m/z intensity vectors.
- When spectra lack instrument type or collision energy metadata — cluster patterns cannot be interpreted as the paper does (30 eV cutoff, collision-cell vs IT).
- When the aim is fingerprint prediction or candidate ranking for metabolite annotation (MetFID's primary goal) rather than diagnosing acquisition-driven spectral clustering.

## Inputs

- MS/MS spectra (e.g., from NIST 17 or MoNA) with per-spectrum instrument type (HCD, QqQ, Q-TOF, IT) and collision energy metadata
- Compound identifiers (InChIKey) for merging spectra acquired from the same compound at different collision energies
- A minimum of 30 spectra per compound used for the curated subset

## Outputs

- pca_cluster_plot.png — 2-D projection of spectra colored/labeled by instrument type and collision energy
- cluster_assignments.csv — per-spectrum cluster IDs and acquisition metadata

## How to apply

First, curate the spectral set to compounds with >30 spectra each (916 compounds in the paper), keep positive-mode spectra from collision-cell instruments (HCD, QqQ, Q-TOF) and IT, merge spectra of the same compound (InChIKey as identifier) with peaks differing by ≤0.1 m/z, and scale peak intensities to the 0–100 range. Denoise by removing peaks with intensity below 10 and drop spectra with fewer than five peaks above 2% relative intensity. Bin the m/z range of each spectrum into continuous integer bins, using accumulated intensity per bin as feature values, and remove bins that are all-zero across the training set. Reduce the binned feature matrix with PCA (e.g., via scikit-learn) to leading components, then cluster (the paper used K-means on three clusters) and annotate the plot by instrument type, using a 30 eV cutoff to split high vs low collision energies when comparing cluster patterns. The rationale is that acquisition metadata — not chemistry — can dominate spectral similarity; this is validated by observing that HCD and QqQ (collision-cell) spectra co-cluster and that IT spectra also fall in the collision-cell cluster despite a completely different fragmentation mechanism.

## Related tools

- **MoNA** (Source of MS/MS spectra libraries with instrument and collision-energy metadata to be clustered) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Source of MS/MS spectra used in the paper's cluster analysis of 916 compounds)
- **InChIKey** (Compound identifier used to merge spectra acquired from the same compound at different collision energies before clustering)

## Examples

```
python3 main.py  # after downloading MetFID code, produces testing_compound_prediction.txt in _files; for cluster analysis use scikit-learn PCA then KMeans(n_clusters=3) on binned m/z intensity vectors
```

## Evaluation signals

- Spectra acquired by HCD and QqQ (collision-cell instruments) fall in the same cluster, matching the paper's reported observation.
- Spectra acquired by IT cluster together with collision-cell spectra despite IT's different fragmentation mechanism.
- Similar collision energies group together, with a 30 eV cutoff used as a reasonable low/high split.
- Each spectrum carries instrument-type and collision-energy metadata so clusters can be annotated — missing metadata makes the result uninterpretable.
- The curated set has >30 spectra per compound (916 compounds in the paper), sufficient for stable grouping.

## Limitations

- Clustering reflects acquisition conditions (instrument type and collision energy) more than the chemical nature of the compounds, so clusters should not be interpreted as chemistry-driven groups.
- Spectra with fewer than five peaks above 2% relative intensity must be removed; retaining them distorts PCA and clustering.
- The 30 eV low/high collision-energy cutoff is an empirical choice from the 916-compound analysis and may not transfer to other datasets.
- Collision-cell instruments were grouped by mechanism rather than resolved individually; finer distinctions between HCD, QqQ, and Q-TOF are not claimed.

## Evidence
