---
name: qemistree-cscs-metric-evaluation
description: Use when comparing how well MS2-based sample clustering metrics (MEMO,
  Qemistree UniFrac, CSCS) versus feature-table Bray-Curtis suppress a retention-time
  batch effect, quantified by PERMANOVA pseudo-F.
when_to_use_negative:
- The dataset lacks MS/MS fragmentation spectra (feature table only), so MS2 fingerprints
  and cosine-score edges for MEMO/CSCS cannot be computed.
- No GNPS FBMN molecular networking artifacts (bucket table plus edges file with cosine
  scores) are available, which q2-cscs requires as inputs.
- All samples were acquired under the same chromatographic method, so there is no
  RT-shift batch effect to mitigate and the experiment-vs-composition pseudo-F comparison
  is uninformative.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: memo-ms
  role: Counts MS2 peaks and neutral losses per sample (SpectraDocuments) and builds
    aligned/unaligned MEMO distance matrices (MemoMatrix) for the MS2-based branch
    of the benchmark
  repo: github.com/mandelbrot-project/memo
- name: q2-cscs
  role: QIIME 2 plugin computing the chemical structural and compositional similarity
    (CSCS) distance from a GNPS feature table and cosine-score edges file
  repo: madeleineernst/q2-cscs
- name: scikit-bio
  role: Computes the feature-table Bray-Curtis dissimilarity matrix and runs PERMANOVA
    (999 permutations) to obtain pseudo-F statistics
- name: matchms
  role: Underlying package used by MEMO for handling MS2 spectra
  repo: https://github.com/matchms/matchms
- name: qemistree-analyses
  role: Reference analyses repository for computing Qemistree UniFrac-style distances
    on q2-qemistree chemical trees
  repo: knightlab-analyses/qemistree-analyses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:789643c07e69cafdc76e19490ac3ad1184147289914917265ef92dca45e60b89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matchms
  - memo-ms
  - numpy
  - q2-cscs
  - qemistree-analyses
  - scikit-bio
  - spec2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# qemistree-cscs-metric-evaluation

## Summary

Benchmark of six clustering distance metrics (MEMO aligned/unaligned, Qemistree weighted/unweighted UniFrac, CSCS weighted/unweighted, feature-table Bray-Curtis) on the 27-sample Qemistree evaluation dataset, scoring how well each mitigates the artificial C18 RT-shift batch effect using PERMANOVA pseudo-F for 'experiment' and 'composition' factors.

## When to use

Use when comparing how well MS2-based sample clustering metrics (MEMO, Qemistree UniFrac, CSCS) versus feature-table Bray-Curtis suppress a retention-time batch effect, quantified by PERMANOVA pseudo-F.

## When NOT to use

- The dataset lacks MS/MS fragmentation spectra (feature table only), so MS2 fingerprints and cosine-score edges for MEMO/CSCS cannot be computed.
- No GNPS FBMN molecular networking artifacts (bucket table plus edges file with cosine scores) are available, which q2-cscs requires as inputs.
- All samples were acquired under the same chromatographic method, so there is no RT-shift batch effect to mitigate and the experiment-vs-composition pseudo-F comparison is uninformative.

## Inputs

- MS2 spectra and sample metadata from the Qemistree evaluation dataset (27 samples, C18 vs C18 RT-shift, Q-Exactive; MassIVE MSV000083306)
- GNPS FBMN/Qemistree job artifacts: feature abundance table and mass spectral molecular network edges (pairwise cosine scores), e.g. GNPS bucket table (.qza/.tsv) and edges .tsv
- Sample metadata containing 'experiment' (LC method) and 'composition' factors

## Outputs

- Six sample-sample distance matrices (MEMO aligned/unaligned, Qemistree weighted/unweighted UniFrac, CSCS weighted/unweighted, Bray-Curtis)
- PERMANOVA pseudo-F statistics per distance matrix for 'experiment' and 'composition' factors (999 permutations)
- Full comparison table written to permanova_results.csv

## How to apply

Obtain the Qemistree evaluation dataset (27 samples, C18 vs C18 RT-shift LC methods, Q-Exactive Orbitrap; MassIVE MSV000083306 / memo_publication_examples) plus its GNPS FBMN/Qemistree job artifacts. Load MS2 spectra into memo-ms, count MS2 peaks and neutral losses per sample with SpectraDocuments, and build both aligned and unaligned MEMO sample-sample distance matrices via MemoMatrix — MEMO is RT-agnostic because it fingerprints samples by fragmentation spectra, not feature alignment. On the same feature table compute Qemistree weighted/unweighted UniFrac-style distances and CSCS weighted/unweighted cosine-similarity-based distances (q2-cscs integrates the GNPS molecular-network cosine score into the compositional similarity), and compute Bray-Curtis directly from the feature abundance table with scikit-bio. Run PERMANOVA with 999 permutations (scikit-bio permanova) on each of the six distance matrices for both the 'experiment' factor and the 'composition' factor. Judge success against the reported benchmark: MS2-based methods should lower the experiment-factor pseudo-F to ≤ 81.64 (vs 113.55 for Bray-Curtis) while raising the composition-factor pseudo-F to ≥ 11.10 (vs 4.72), and write the full comparison to permanova_results.csv.

## Related tools

- **memo-ms** (Counts MS2 peaks and neutral losses per sample (SpectraDocuments) and builds aligned/unaligned MEMO distance matrices (MemoMatrix) for the MS2-based branch of the benchmark) — github.com/mandelbrot-project/memo
- **q2-cscs** (QIIME 2 plugin computing the chemical structural and compositional similarity (CSCS) distance from a GNPS feature table and cosine-score edges file) — madeleineernst/q2-cscs
- **scikit-bio** (Computes the feature-table Bray-Curtis dissimilarity matrix and runs PERMANOVA (999 permutations) to obtain pseudo-F statistics)
- **matchms** (Underlying package used by MEMO for handling MS2 spectra) — https://github.com/matchms/matchms
- **qemistree-analyses** (Reference analyses repository for computing Qemistree UniFrac-style distances on q2-qemistree chemical trees) — knightlab-analyses/qemistree-analyses

## Examples

```
qiime cscs cscs --p-css-edges GNPS_edges.tsv --i-features GNPS_buckettable.qza --p-cosine-threshold 0.5 --p-normalization --o-distance-matrix cscs_distance_matrix.qza
```

## Evaluation signals

- Experiment-factor pseudo-F for every MS2-based method is ≤ 81.64, clearly below the feature-table Bray-Curtis value of 113.55.
- Composition-factor pseudo-F for MS2-based methods is ≥ 11.10, above the Bray-Curtis value of 4.72.
- Each PERMANOVA run uses 999 permutations; six distance matrices × two factors yield twelve pseudo-F statistics, all recorded in permanova_results.csv.
- Every distance matrix is a symmetric square matrix over the same 27 samples with zero diagonal; CSCS dissimilarities fall in [0, 1] (1-CSCS), as in the q2-cscs unit-test matrix.
- MEMO matrices are built without a prior feature alignment step, distinguishing them from the alignment-dependent Qemistree/Bray-Curtis matrices.

## Limitations

- The benchmark targets chemodiverse samples and strong RT shifts (different LC methods or instruments); MEMO's advantage rests on RT-agnostic MS2 fingerprinting and does not exploit retention-time information.
- q2-cscs requires a GNPS bucket table and an edges file of pairwise cosine scores, and its output depends on the chosen cosine threshold (e.g. 0.5 in the unit test) and normalization flag.
- The pseudo-F benchmark is specific to the 27-sample Qemistree evaluation dataset; values (81.64/113.55 experiment, 11.10/4.72 composition) are dataset-specific thresholds, not universal cutoffs.

## Evidence

- [abstract] a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [methods] The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [readme] The distance matrix, which is computed through q2-cscs represents the chemical structural and compositional dissimilarity for all pairs of samples in a feature table and corresponds to 1-CSCS.: [evidence span withheld — anchor status: external_doc]
- [intro] MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift"
