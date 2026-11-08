---
name: qtof-orbitrap-cross-platform-sample-comparison
description: Use when LC-MS/MS samples were acquired on different mass spectrometers
  (e.g., Q-Exactive Orbitrap vs Maxis II Q-TOF) or with strong RT shifts, and you
  need to compare samples without prior feature alignment.
when_to_use_negative:
- The samples were all acquired on a single platform with well-aligned features —
  a standard feature-table-based ordination suffices and MEMO's cross-platform advantage
  is unneeded
- You need to compare samples using a feature table with prior alignment (e.g., GNPS
  bucket table + CSCS edges); MEMO is specifically designed to avoid the prior feature
  alignment step
- Only MS1-level feature data are available without MS2 fragmentation spectra, since
  MS2 fingerprints are built from MS2 peaks and neutral losses to the precursor
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: Builds MS2 fingerprints (counting MS2 peaks and neutral losses per sample),
    aligns MemoMatrix objects across platforms, and exports the MEMO similarity/distance
    matrix
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: Performs PCoA ordination of the MEMO distance matrix and PERMANOVA testing
    (pseudo-F, p-values) with 999+ permutations
- name: matchms
  role: Underlying package used by MEMO for handling MS2 spectra and converting them
    into documents
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying package used by MEMO for handling MS2 spectra and converting them
    into documents
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Numerical operations for fingerprint counting and matrix normalization/alignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:028efef1fcdc85618f91640c941583e7481daf86140d098443516bbfe3e095ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# qtof-orbitrap-cross-platform-sample-comparison

## Summary

Vectorizes metabolomics samples acquired on different MS platforms (Maxis II Q-TOF vs Q-Exactive Orbitrap) using MEMO MS2 fingerprints, enabling retention-time-agnostic, cross-platform sample comparison via PCoA and PERMANOVA.

## When to use

Use when LC-MS/MS samples were acquired on different mass spectrometers (e.g., Q-Exactive Orbitrap vs Maxis II Q-TOF) or with strong RT shifts, and you need to compare samples without prior feature alignment.

## When NOT to use

- The samples were all acquired on a single platform with well-aligned features — a standard feature-table-based ordination suffices and MEMO's cross-platform advantage is unneeded
- You need to compare samples using a feature table with prior alignment (e.g., GNPS bucket table + CSCS edges); MEMO is specifically designed to avoid the prior feature alignment step
- Only MS1-level feature data are available without MS2 fragmentation spectra, since MS2 fingerprints are built from MS2 peaks and neutral losses to the precursor

## Inputs

- MS2 peak lists in MGF format for each MS platform (e.g., from MassIVE accession MSV000083306)
- Feature quantification tables for each platform acquisition
- Sample metadata labels (experiment/platform and composition) for PERMANOVA grouping

## Outputs

- Aligned, sample-wise-normalized cross-platform MEMO matrix
- Pairwise MEMO distance/similarity matrix
- PCoA ordination coordinates colored by experiment and composition
- PERMANOVA pseudo-F and p-values per grouping variable (e.g., pseudo_f_verification.csv)

## How to apply

Retrieve the MS2 peak lists (MGF) and feature quantification tables for the two platform acquisitions (e.g., MSV000083306) and load them with memo_ms.import_data into SpectraDocuments and FeatureTable objects. For each platform, count the occurrence of MS2 peaks and neutral losses (to the precursor) per sample to build MS2 fingerprints, aggregated in a MemoMatrix. Align the two platform-specific MEMO matrices on their shared peak/loss keys and apply sample-wise normalization so the matrices are comparable across instruments. Compute the pairwise MEMO distance/similarity matrix, ordinate samples with PCoA (scikit-bio), and color by experiment and composition. Run PERMANOVA twice (experiment labels vs composition labels) with 999+ permutations; in the reported Figure 3 analysis, platform dominates PC1 (pseudo-F = 213.35, p = 0.001) while composition is resolved on PC2/PC3 (pseudo-F = 5.19, p = 0.001).

## Related tools

- **memo-ms** (Builds MS2 fingerprints (counting MS2 peaks and neutral losses per sample), aligns MemoMatrix objects across platforms, and exports the MEMO similarity/distance matrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (Performs PCoA ordination of the MEMO distance matrix and PERMANOVA testing (pseudo-F, p-values) with 999+ permutations)
- **matchms** (Underlying package used by MEMO for handling MS2 spectra and converting them into documents) — https://github.com/matchms/matchms
- **spec2vec** (Underlying package used by MEMO for handling MS2 spectra and converting them into documents) — https://github.com/iomega/spec2vec
- **numpy** (Numerical operations for fingerprint counting and matrix normalization/alignment)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms
```

## Evaluation signals

- PERMANOVA on the aligned, sample-wise-normalized MEMO distances reproduces the reported statistics: pseudo-F = 213.35 (p = 0.001) for the platform/experiment effect and pseudo-F = 5.19 (p = 0.001) for composition
- The aligned MEMO matrix contains only shared peak/loss keys across both platforms and both platform sub-matrices are sample-wise normalized before merging
- PCoA of the MEMO distances shows platform (experiment) dominating the first axis and composition separating on PC2/PC3
- Fingerprints contain counted MS2 peaks AND neutral losses to the precursor (not just peaks) for every sample on both instruments
- Ordination coordinates and PERMANOVA results are written out (pseudo_f_verification.csv) and match the reported values within permutation stochasticity (p = 0.001)

## Limitations

- Cross-platform comparison is dominated by the MS platform effect (pseudo-F = 213.35 vs 5.19 for composition); composition differences are only resolved on PC2/PC3, so conclusions about sample composition must account for the platform axis
- PERMANOVA p-values depend on the permutation count (999+ permutations; p = 0.001 floor)
- Python environment constraints: MEMO documentation requires a dedicated conda environment (python=3.8), and the required Python version was later lowered to >3.7 for TMAP compatibility
- Filtering of the fingerprints (e.g., removing peaks/losses from blanks) must be applied carefully; a code fix in classes.py was needed to avoid erasing during filtering

## Evidence

- [intro] MEMO suits particularly well to compare chemodiverse samples, i.e. with a poor features overlap, or to compare samples with a strong RT shift, acquired using different LC methods or even different mass spectrometer technologies (Maxis Q-ToF vs Q-Exactive Orbitrap).: "MEMO suits particularly well to compare chemodiverse samples, ie with a poor features overlap, or to compare samples with a strong RT shift, acquired using different LC methods or even different mass"
- [methods] The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an MS2 fingerprint of the sample.: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [methods] These fingerprints can in a second stage be aligned to compare different samples.: "These
fingerprints can in a second stage be aligned to compare different
samples."
- [methods] different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used"
- [abstract] MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms.: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [methods] MEMO is mainly built on matchms and spec2vec packages for handling the MS2 spectra and converting them into documents.: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra"
- [methods] conda create --name memo python=3.8: "conda create --name memo python=3.8"
