---
name: collision-energy-metadata-curation
description: Use when preparing MS/MS training data from spectral libraries (MoNA,
  NIST 17) where spectra for the same compound were acquired with heterogeneous collision
  energies or instrument types, and you need energy-aware filtering, merging, or train/test
  splits.
when_to_use_negative:
- Spectra lack collision energy or instrument-type metadata — cannot assign low/high
  energy labels or instrument-aware splits
- All training spectra come from a single instrument and a single fixed collision
  energy, so the low/high (30 eV) and IT/HCD distinction is moot
- The downstream model tolerates heterogeneous energies without loss (MetFID showed
  combining IT and HCD did not affect performance), so pooling spectra is preferable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MoNA
  role: Source of MS/MS spectra with collision-energy and instrument-type metadata
    for filtering and merging
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Library source used to obtain spectra at diverse collision energies for the
    low/high energy split
- name: InChIKey
  role: Compound identifier used to merge multiple spectra of the same compound acquired
    at different collision energies
- name: scikit-learn (PCA, K-means)
  role: PCA dimensionality reduction and K-means clustering of binned spectra to validate
    that similar collision energies cluster together
- name: MetFID
  role: Downstream ANN fingerprint predictor that benefits from energy-specific training
    sets
  repo: https://github.com/aohongyu/MetFID
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:2624342bc65b2e05c3df61e8aff7e34509b57b7cd266cc3274936b6bc653fa5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  - scikit-learn (PCA, K-means)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# collision-energy-metadata-curation

## Summary

Curate and split MS/MS spectra by collision energy before modeling, because spectra at very different collision energies fragment differently even for the same compound. In MetFID, spectra were retained only if their collision energy metadata (HCD/QqQ/Q-TOF vs IT, low ≤30 eV vs high ≥30 eV) was consistent, enabling separate low- and high-energy ANN models.

## When to use

Use when preparing MS/MS training data from spectral libraries (MoNA, NIST 17) where spectra for the same compound were acquired with heterogeneous collision energies or instrument types, and you need energy-aware filtering, merging, or train/test splits.

## When NOT to use

- Spectra lack collision energy or instrument-type metadata — cannot assign low/high energy labels or instrument-aware splits
- All training spectra come from a single instrument and a single fixed collision energy, so the low/high (30 eV) and IT/HCD distinction is moot
- The downstream model tolerates heterogeneous energies without loss (MetFID showed combining IT and HCD did not affect performance), so pooling spectra is preferable

## Inputs

- MS/MS spectra from MoNA libraries (Vaniya/Fiehn Natural Products Library, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) and NIST 17
- Per-spectrum metadata: ionization mode, instrument type (HCD, QqQ, Q-TOF, IT), and collision energy (eV)
- InChIKey compound identifiers for merging spectra of the same compound

## Outputs

- Filtered, merged per-compound spectra retaining instrument type and low/high collision-energy labels (30 eV cutoff)
- Separate training sets for low-energy (≤30 eV) and high-energy (≥30 eV) ANN models
- Cluster assignments and PCA plot confirming that similar collision energies co-cluster (e.g. cluster_assignments.csv, pca_cluster_plot.png)

## How to apply

First, select spectra with usable metadata: in MetFID only positive-mode spectra from HCD, QqQ, Q-TOF (collision cell) or IT were retained. Use InChIKey to group multiple spectra of the same compound, scale intensities to 0–100, drop spectra with fewer than five peaks above 2% relative intensity, and merge peaks with Δm/z ≤ 0.1 when combining different collision energies. To detect whether collision energy affects fragmentation, scale peak intensities, bin m/z into integer bins, reduce with PCA, and cluster (K-means, three clusters) — spectra at similar collision energies co-clustered, and 30 eV emerged as a reasonable low/high cutoff. Train separate ANNs for low (≤30 eV) and high (≥30 eV) energies; a combined model showed slightly lower accuracy on low-energy spectra and lower F1 scores than the energy-specific models. For IT vs HCD, combining did not affect prediction performance, so IT spectra may be pooled with collision-cell spectra.

## Related tools

- **MoNA** (Source of MS/MS spectra with collision-energy and instrument-type metadata for filtering and merging) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Library source used to obtain spectra at diverse collision energies for the low/high energy split)
- **InChIKey** (Compound identifier used to merge multiple spectra of the same compound acquired at different collision energies)
- **scikit-learn (PCA, K-means)** (PCA dimensionality reduction and K-means clustering of binned spectra to validate that similar collision energies cluster together)
- **MetFID** (Downstream ANN fingerprint predictor that benefits from energy-specific training sets) — https://github.com/aohongyu/MetFID

## Examples

```
python main.py  # after curating spectra: scale intensities to 0–100, filter ≥5 peaks >2%, merge same-InChIKey spectra with Δm/z ≤ 0.1, split at 30 eV
```

## Evaluation signals

- After filtering, every retained spectrum carries ionization mode (positive), instrument type (HCD/QqQ/Q-TOF/IT), and a low/high energy label relative to 30 eV
- Cluster plot shows HCD and QqQ spectra co-cluster, and IT spectra co-cluster with collision-cell spectra; spectra at similar collision energies group together
- Accuracy and F1 of separate low/high-energy ANN models exceed those of a single combined model on low-energy spectra
- No spectrum with fewer than five peaks above 2% relative intensity survives the filter; intensities scaled to 0–100
- Merged spectra share one InChIKey with peaks merged when Δm/z ≤ 0.1

## Limitations

- The 30 eV low/high cutoff is empirical and may not transfer to other instruments or libraries
- PCA/K-means clustering with integer m/z binning is coarse and cannot quantify how fragmentation similarity varies continuously with energy
- Combining IT and HCD did not affect prediction performance in MetFID, so instrument-level splitting may be unnecessary in some settings
- Collision energy is defined relative to a 30 eV cutoff; very low or very high values within a bin are not distinguished further
- Metadata for some public spectra (e.g. non-positive mode, non-collision-cell instruments) is excluded, potentially reducing training-set size

## Evidence
