---
name: rt-shift-cross-method-sample-comparison
description: Use when samples are acquired with different LC methods or strong retention-time
  shifts, features overlap poorly (chemodiverse data), and you need RT-agnostic sample
  clustering or batch-effect quantification from MS2 spectra.
when_to_use_negative:
- Samples were all acquired under identical chromatographic conditions with good feature
  overlap, where a simple feature-table Bray-Curtis dissimilarity suffices
- Only MS1 feature tables are available and no MS2 fragmentation spectra were recorded
  (MEMO fingerprints require MS2 peaks and neutral losses)
- The goal is feature/compound-level annotation or molecular networking rather than
  sample-level clustering and batch-effect assessment
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: memo-ms
  role: Counts MS2 peaks and neutral losses per sample (SpectraDocuments) and builds
    aligned/unaligned MEMO distance matrices (MemoMatrix)
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: Computes the feature-table Bray-Curtis dissimilarity matrix and runs PERMANOVA
    (999 permutations) on all six distance matrices
- name: matchms
  role: Underlying package for handling MS2 spectra used to build MEMO
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts MS2 spectra into documents for fingerprint generation within MEMO
  repo: https://github.com/iomega/spec2vec
- name: q2-cscs
  role: Computes the chemical structural and compositional similarity (CSCS) distance
    matrix from a GNPS feature table and cosine-score edges file
  repo: madeleineernst/q2-cscs
- name: qemistree-analyses
  role: Reference analyses for computing Qemistree UniFrac-style distances on the
    evaluation dataset
  repo: knightlab-analyses/qemistree-analyses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:b84f66bb73d7f5be8118f4092e1c2025fccd9f697feacf2b75b29cc0f38dd5f4
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# rt-shift-cross-method-sample-comparison

## Summary

Compares metabolomics samples across chromatographic conditions (e.g., C18 vs C18 RT-shift on a Q-Exactive) by vectorizing each sample from its MS2 fragmentation spectra (MS2 peaks + neutral losses) with MEMO, then benchmarking the resulting distance matrices against Qemistree UniFrac, CSCS, and feature-table Bray-Curtis using PERMANOVA pseudo-F.

## When to use

Use when samples are acquired with different LC methods or strong retention-time shifts, features overlap poorly (chemodiverse data), and you need RT-agnostic sample clustering or batch-effect quantification from MS2 spectra.

## When NOT to use

- Samples were all acquired under identical chromatographic conditions with good feature overlap, where a simple feature-table Bray-Curtis dissimilarity suffices
- Only MS1 feature tables are available and no MS2 fragmentation spectra were recorded (MEMO fingerprints require MS2 peaks and neutral losses)
- The goal is feature/compound-level annotation or molecular networking rather than sample-level clustering and batch-effect assessment

## Inputs

- MS2 spectra files and sample metadata (e.g., 27 samples from the Qemistree evaluation dataset, MassIVE MSV000083306 / memo_publication_examples)
- GNPS FBMN/Qemistree job artifacts (feature table and edges file with pairwise cosine scores)
- Feature abundance table (for the Bray-Curtis baseline)

## Outputs

- Aligned and unaligned MEMO sample-sample distance matrices
- Qemistree UniFrac-style and CSCS distance matrices (weighted and unweighted)
- Bray-Curtis dissimilarity matrix
- PERMANOVA pseudo-F statistics per method for 'experiment' and 'composition' factors, written to permanova_results.csv

## How to apply

Load MS2 spectra and metadata into memo-ms, count MS2 peaks and neutral losses (to the precursor) per sample using SpectraDocuments, and build both aligned and unaligned sample-sample distance matrices via MemoMatrix — this RT-agnostic fingerprint alignment avoids the prior feature-alignment step required by feature-table methods. As comparators, compute Qemistree weighted/unweighted UniFrac-style and CSCS weighted/unweighted cosine-similarity distances (q2-cscs, using a GNPS feature table plus pairwise cosine-score edges file), and Bray-Curtis dissimilarity directly on the feature abundance table with scikit-bio. Run PERMANOVA (999 permutations, scikit-bio permanova) on each of the six distance matrices for the 'experiment' factor and the 'composition' factor. Judge success against the reported benchmark: MS2-based methods should yield pseudo-F <= 81.64 vs 113.55 for the experiment factor (mitigated batch effect) and >= 11.10 vs 4.72 for the composition factor (better sample discrimination) relative to feature-table Bray-Curtis.

## Related tools

- **memo-ms** (Counts MS2 peaks and neutral losses per sample (SpectraDocuments) and builds aligned/unaligned MEMO distance matrices (MemoMatrix)) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (Computes the feature-table Bray-Curtis dissimilarity matrix and runs PERMANOVA (999 permutations) on all six distance matrices)
- **matchms** (Underlying package for handling MS2 spectra used to build MEMO) — https://github.com/matchms/matchms
- **spec2vec** (Converts MS2 spectra into documents for fingerprint generation within MEMO) — https://github.com/iomega/spec2vec
- **q2-cscs** (Computes the chemical structural and compositional similarity (CSCS) distance matrix from a GNPS feature table and cosine-score edges file) — madeleineernst/q2-cscs
- **qemistree-analyses** (Reference analyses for computing Qemistree UniFrac-style distances on the evaluation dataset) — knightlab-analyses/qemistree-analyses

## Examples

```
from skbio.stats.distance import permanova; permanova(distance_matrix, grouping, column='experiment', permutations=999)
```

## Evaluation signals

- PERMANOVA pseudo-F for the 'experiment' factor for each MS2-based method is <= 81.64, versus 113.55 for feature-table Bray-Curtis (RT batch effect mitigated)
- PERMANOVA pseudo-F for the 'composition' factor for each MS2-based method is >= 11.10, versus 4.72 for Bray-Curtis (better sample discrimination)
- Each distance matrix is square, symmetric, has a zero diagonal, and covers all 27 samples across the C18 and C18 RT-shift experiments
- PERMANOVA is run with exactly 999 permutations for both the 'experiment' and 'composition' factors on all six distance matrices
- Aligned and unaligned MEMO matrices are both produced, confirming the fingerprint alignment step was executed rather than a feature-table alignment

## Limitations

- Benchmark values are specific to the Qemistree evaluation dataset (27 samples, C18 vs C18 RT-shift, Q-Exactive Orbitrap); other datasets will yield different pseudo-F scales
- Qemistree and CSCS comparators require a prior feature-alignment step and GNPS molecular-networking artifacts (feature table + cosine edges), which MEMO is designed to avoid
- Fingerprint filtering can erase data if implemented incorrectly — a fix to classes.py was needed 'to avoid erasing when filtering'; peaks/losses from blanks should be removed before comparison
- MS2-based distances suit chemodiverse samples (poor feature overlap, strong RT shift, different LC methods or MS platforms such as Maxis Q-ToF vs Q-Exactive) and may offer no advantage where feature overlap is high

## Evidence

- [abstract] we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [methods] The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an MS2 fingerprint of the sample: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [intro] MEMO suits particularly well to compare chemodiverse samples, ie with a poor features overlap, or to compare samples with a strong RT shift,: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift,"
- [readme] This repository contains code to reproduce MEMO analyses. The preprint is available on Biorxiv.: [evidence span withheld — anchor status: not_found]
- [readme] The distance matrix, which is computed through q2-cscs represents the chemical structural and compositional dissimilarity for all pairs of samples: [evidence span withheld — anchor status: external_doc]
