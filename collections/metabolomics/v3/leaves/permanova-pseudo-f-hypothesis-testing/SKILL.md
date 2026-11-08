---
name: permanova-pseudo-f-hypothesis-testing
description: Use when a precomputed sample-sample distance matrix (MEMO, Qemistree
  UniFrac, CSCS, or feature-table Bray-Curtis) must be tested for group effects —
  e.g. 'experiment' (C18 vs C18 RT-shift, Q-Exactive vs Maxis II Q-TOF) vs 'composition'
  — via PERMANOVA pseudo-F.
when_to_use_negative:
- No distance matrix and no sample metadata factor with replicate groups exist yet
  — PERMANOVA operates on precomputed distance matrices plus categorical groupings,
  not raw spectra or per-feature tables
- The question is feature-level (which peaks/losses differ) rather than sample-level
  group separation — PERMANOVA pseudo-F tests factors over samples only
- A parametric effect-size estimate or per-feature significance testing is required
  — PERMANOVA is a non-parametric pseudo-F permutation test, not a differential-abundance
  method
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: scikit-bio
  role: Provides the permanova function (999 permutations) for computing pseudo-F
    statistics and p-values on distance matrices, plus Bray-Curtis dissimilarity and
    PCoA ordination
- name: memo-ms
  role: 'Builds the MEMO distance matrices under test: counts MS2 peaks and neutral
    losses per sample (SpectraDocuments) into MemoMatrix aligned/unaligned matrices'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Handles the MS2 spectra underlying the MS2 fingerprints from which tested
    MEMO distances are derived
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Handles MS2 spectra and converts them into documents as part of the MEMO fingerprint
    pipeline feeding the tested distance matrices
  repo: https://github.com/iomega/spec2vec
- name: q2-cscs
  role: Produces the CSCS (chemical structural and compositional similarity) distance
    matrices that are one of the six inputs benchmarked by PERMANOVA
  repo: https://github.com/madeleineernst/q2-cscs
- name: qemistree-analyses
  role: Source of the Qemistree weighted/unweighted UniFrac-style distance computations
    benchmarked against MEMO via PERMANOVA
  repo: https://github.com/knightlab-analyses/qemistree-analyses
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:c912d4438bbb545d57be6a24e97a168003acaabd0bc21bc3195470c6dce81370
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# permanova-pseudo-f-hypothesis-testing

## Summary

Non-parametric PERMANOVA hypothesis testing of sample-level distance matrices using scikit-bio, reporting pseudo-F statistics and p-values (999 permutations) for factors such as LC method, MS platform, or sample composition. In the MEMO workflow it is the benchmark metric quantifying how well each clustering method (MEMO, Qemistree UniFrac, CSCS, feature-table Bray-Curtis) mitigates the artificial retention-time batch effect.

## When to use

Use when a precomputed sample-sample distance matrix (MEMO, Qemistree UniFrac, CSCS, or feature-table Bray-Curtis) must be tested for group effects — e.g. 'experiment' (C18 vs C18 RT-shift, Q-Exactive vs Maxis II Q-TOF) vs 'composition' — via PERMANOVA pseudo-F.

## When NOT to use

- No distance matrix and no sample metadata factor with replicate groups exist yet — PERMANOVA operates on precomputed distance matrices plus categorical groupings, not raw spectra or per-feature tables
- The question is feature-level (which peaks/losses differ) rather than sample-level group separation — PERMANOVA pseudo-F tests factors over samples only
- A parametric effect-size estimate or per-feature significance testing is required — PERMANOVA is a non-parametric pseudo-F permutation test, not a differential-abundance method

## Inputs

- Sample-sample distance matrix (e.g. memo-ms MemoMatrix aligned/unaligned MEMO distances, Qemistree UniFrac, q2-cscs CSCS distance matrix .qza, or scikit-bio Bray-Curtis from the feature abundance table)
- Sample metadata table with categorical factors, e.g. 'experiment' (C18 vs C18 RT-shift LC method; Q-Exactive Orbitrap vs Maxis II Q-TOF) and 'composition'
- MS2 peak lists (MGF) and GNPS FBMN/Qemistree artifacts from MassIVE deposit MSV000083306 (27 samples)

## Outputs

- PERMANOVA results table with pseudo-F statistics and p-values per distance matrix per factor (e.g. permanova_results.csv, pseudo_f_verification.csv)
- Comparison table benchmarking the six clustering methods against reported pseudo-F values

## How to apply

Build a symmetric sample-sample distance matrix first — e.g. aligned/unaligned MEMO matrices from SpectraDocuments counting MS2 peaks and neutral losses to the precursor via memo-ms, Qemistree weighted/unweighted UniFrac and CSCS distances, or Bray-Curtis dissimilarity computed directly from the feature abundance table with scikit-bio. Then run scikit-bio's permanova function on each distance matrix once per metadata factor ('experiment' and 'composition') with 999 permutations to obtain pseudo-F statistics and p-values. Interpret the pair of pseudo-F values jointly: a method that mitigates the RT batch effect shows a LOWER pseudo-F for the experiment factor while retaining a HIGHER pseudo-F for composition (MEMO benchmark: experiment ≤ 81.64 vs 113.55 for feature-table Bray-Curtis; composition ≥ 11.10 vs 4.72). For the cross-platform QE vs Q-ToF comparison, expect experiment pseudo-F = 213.35 and composition pseudo-F = 5.19, both p = 0.001. Write the full comparison table (e.g. permanova_results.csv or pseudo_f_verification.csv) so computed values can be diffed against reported benchmarks.

## Related tools

- **scikit-bio** (Provides the permanova function (999 permutations) for computing pseudo-F statistics and p-values on distance matrices, plus Bray-Curtis dissimilarity and PCoA ordination)
- **memo-ms** (Builds the MEMO distance matrices under test: counts MS2 peaks and neutral losses per sample (SpectraDocuments) into MemoMatrix aligned/unaligned matrices) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Handles the MS2 spectra underlying the MS2 fingerprints from which tested MEMO distances are derived) — https://github.com/matchms/matchms
- **spec2vec** (Handles MS2 spectra and converts them into documents as part of the MEMO fingerprint pipeline feeding the tested distance matrices) — https://github.com/iomega/spec2vec
- **q2-cscs** (Produces the CSCS (chemical structural and compositional similarity) distance matrices that are one of the six inputs benchmarked by PERMANOVA) — https://github.com/madeleineernst/q2-cscs
- **qemistree-analyses** (Source of the Qemistree weighted/unweighted UniFrac-style distance computations benchmarked against MEMO via PERMANOVA) — https://github.com/knightlab-analyses/qemistree-analyses

## Examples

```
from skbio.stats.distance import permanova; result = permanova(distance_matrix, metadata, column='experiment', permutations=999)
```

## Evaluation signals

- For the Qemistree evaluation dataset (27 samples, Q-Exactive), MS2-based methods must show experiment-factor pseudo-F ≤ 81.64 vs 113.55 for feature-table Bray-Curtis, and composition-factor pseudo-F ≥ 11.10 vs 4.72
- For the cross-platform QE vs Q-ToF PCoA analysis, PERMANOVA on sample-wise-normalized aligned MEMO distances must reproduce pseudo-F = 213.35 (experiment) and pseudo-F = 5.19 (composition), both with p = 0.001
- All p-values are computed with 999 permutations and reported alongside pseudo-F (e.g. p = 0.001)
- Input distance matrices are symmetric, square, and sample-indexed across the same 27 samples, matching the metadata factor columns used in the test
- The full comparison table is written to a CSV (permanova_results.csv / pseudo_f_verification.csv) whose values diff against the reported benchmark table

## Limitations

- PERMANOVA requires sufficient replication per factor level; with small designs (e.g. 27 samples across two LC methods) permutation resolution is limited to the 999-permutation p-value floor (p = 0.001)
- Pseudo-F values are only comparable across methods when computed on the same sample set and same factor definitions; cross-platform MEMO requires alignment and sample-wise normalization of matrices before testing
- PCoA of cross-platform MEMO matrices is dominated by MS platform (pseudo-F = 213.35), so composition effects must be assessed on PC2/PC3 rather than overall clustering

## Evidence

- [abstract] MEMO allows clustering large amounts of chemodiverse samples based on LC-MS/MS profiles in a retention time agnostic manner: [evidence span withheld — anchor status: not_found]
- [methods] scikit-bio install and permanova usage: "conda install -c conda-forge scikit-bio"
- [intro] MS2 fingerprint generation feeding the tested distances: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [readme] q2-cscs README on CSCS distance matrix output used for PCoA/PERMANOVA-style analyses: [evidence span withheld — anchor status: external_doc]
