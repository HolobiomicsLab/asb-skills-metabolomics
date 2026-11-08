---
name: pcoa-ordination-of-metabolomics-samples
description: Use when you have an aligned MS2 fingerprint similarity matrix (e.g.,
  from MEMO, built by counting MS2 peaks and neutral losses to the precursor) and
  need PCoA ordination plus PERMANOVA pseudo-F/p-values for grouping factors.
when_to_use_negative:
- When the comparison depends on retention-time behavior — MEMO is explicitly a retention
  time (RT) agnostic alignment, so PCoA on MEMO distances will not resolve RT differences
- When samples have strong feature overlap and a conventional RT-aligned feature table
  analysis is appropriate rather than MS2-fingerprint vectorization
- When the input is raw chromatographic (MS1) data without MS2 fragmentation spectra,
  since the fingerprints require MS2 peaks and neutral losses
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: memo-ms
  role: Builds the MS2 fingerprints (counts of MS2 peaks and neutral losses to the
    precursor per sample), aggregates them into a MemoMatrix, and aligns fingerprints
    for distance computation
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: Provides the PCoA ordination and PERMANOVA (with 999+ permutations) used to
    test experiment and composition effects on the MEMO distance matrix
- name: matchms
  role: Underlying package MEMO builds on for handling the MS2 spectra and converting
    them into documents
  repo: https://github.com/matchms/matchms
- name: numpy
  role: Numerical handling of the fingerprint matrices and pairwise distance arrays
    prior to ordination
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:66754e232dd1b0ad4a16a597061bed2e1389035fd3d6f5fc29ade40c621db309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pcoa-ordination-of-metabolomics-samples

## Summary

Ordinate metabolomics samples in PCoA space from an aligned MS2-fingerprint (MEMO) distance matrix and quantify group separation (e.g., MS platform vs sample composition) with scikit-bio PERMANOVA pseudo-F statistics.

## When to use

Use when you have an aligned MS2 fingerprint similarity matrix (e.g., from MEMO, built by counting MS2 peaks and neutral losses to the precursor) and need PCoA ordination plus PERMANOVA pseudo-F/p-values for grouping factors.

## When NOT to use

- When the comparison depends on retention-time behavior — MEMO is explicitly a retention time (RT) agnostic alignment, so PCoA on MEMO distances will not resolve RT differences
- When samples have strong feature overlap and a conventional RT-aligned feature table analysis is appropriate rather than MS2-fingerprint vectorization
- When the input is raw chromatographic (MS1) data without MS2 fragmentation spectra, since the fingerprints require MS2 peaks and neutral losses

## Inputs

- MS2 peak list (MGF) files and feature quantification tables from MassIVE (e.g., MSV000083306 Q-Exactive Orbitrap and Maxis II Q-TOF acquisitions)
- Sample metadata labels (experiment/MS platform and sample composition) for coloring and PERMANOVA grouping
- Aligned, sample-wise-normalized MS2 fingerprint (MemoMatrix) distance/similarity matrix

## Outputs

- PCoA ordination coordinates of samples colored by experiment and composition
- PERMANOVA results table (pseudo_f_verification.csv) with pseudo-F and p-values per factor

## How to apply

Load the MS2 peak lists (MGF) and feature quantification tables (e.g., MassIVE accession MSV000083306, acquired on Q-Exactive Orbitrap and Maxis II Q-TOF) with memo_ms.import_data into SpectraDocuments/FeatureTable objects, then aggregate sample-wise MS2 peaks and neutral losses to the precursor into a MemoMatrix. Align the platform-specific fingerprint matrices on shared peak/loss keys and apply sample-wise normalization so cross-platform matrices are comparable. Compute the pairwise MEMO distance/similarity matrix and ordinate samples with scikit-bio PCoA, coloring samples by experiment and by composition. Run scikit-bio permanova on the MEMO distances twice — once with experiment labels and once with composition labels — using 999+ permutations to obtain pseudo-F and p-values, and write the ordination coordinates and statistics to pseudo_f_verification.csv.

## Related tools

- **memo-ms** (Builds the MS2 fingerprints (counts of MS2 peaks and neutral losses to the precursor per sample), aggregates them into a MemoMatrix, and aligns fingerprints for distance computation) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (Provides the PCoA ordination and PERMANOVA (with 999+ permutations) used to test experiment and composition effects on the MEMO distance matrix)
- **matchms** (Underlying package MEMO builds on for handling the MS2 spectra and converting them into documents) — https://github.com/matchms/matchms
- **numpy** (Numerical handling of the fingerprint matrices and pairwise distance arrays prior to ordination)

## Examples

```
conda install -c conda-forge scikit-bio
```

## Evaluation signals

- On MSV000083306, PERMANOVA pseudo-F for the platform (experiment) effect should reproduce ~213.35 and for composition ~5.19, both with p = 0.001 using 999+ permutations
- The PCoA should show the ordination dominated by MS platform (major axis) while composition resolves on PC2/PC3
- The pairwise MEMO distance matrix must be symmetric with a zero diagonal after sample-wise normalization and cross-platform alignment
- Fingerprint counts include both MS2 peaks and neutral losses to the precursor for every sample before alignment
- pseudo_f_verification.csv contains ordination coordinates plus both PERMANOVA factor results matching the reported values

## Limitations

- PERMANOVA p-values with 999 permutations have a floor of ~0.001, so borderline significance cannot be resolved below that level
- MEMO ordination is retention time agnostic, so RT-driven separation is invisible in the resulting PCoA
- Cross-platform comparison requires explicit alignment on shared peak/loss keys and sample-wise normalization before distances are comparable
- Environment constraints: the memo conda environment was created with Python 3.8 (and the Python requirement was later lowered from >3.8 to >3.7 for TMAP compatibility)

## Evidence

- [intro] MEMO suits particularly well to compare chemodiverse samples, ie with a poor features overlap, or to compare samples with a strong RT shift, acquired using different LC methods or even different mass spectrometer technologies (Maxis Q-ToF vs Q-Exactive Orbitrap).: "MEMO suits particularly well to compare chemodiverse samples, ie with a poor features overlap, or to compare samples with a strong RT shift,"
- [intro] Filtering and visualization options for aligned fingerprints.: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [methods] Installation of scikit-bio for ordination and PERMANOVA.: "conda install -c conda-forge scikit-bio"
- [readme] The publication-examples repository provides code and data to reproduce the MEMO analyses, including this PCoA/PERMANOVA workflow.: [evidence span withheld — anchor status: not_found]
