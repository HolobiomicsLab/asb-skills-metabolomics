---
name: java-algorithm-implementation
description: Use when reproducing or benchmarking a published peak-list alignment
  (or similar MS data-processing) algorithm in Java, where the method must be re-implemented
  from the paper's description and evaluated against known ground-truth tables.
when_to_use_negative:
- Input is already an aligned peak list or feature table — the aligner assumes unaligned
  per-sample peak lists as input
- Retention times among samples deviate non-linearly — the Join aligner cannot cope
  with non-linear RT deviation; use the RANSAC aligner instead
- The analysis goal is peak picking, deisotoping, or gap filling rather than aligning
  multiple peak lists
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Java
  role: Language in which the Join aligner algorithm is implemented; MZmine 2 itself
    was developed using Java technology and is platform independent
- name: Java Development Kit (JDK) version 12
  role: Required build toolchain to compile MZmine 2 sources with gradlew before implementing
    or running the algorithm
- name: MZmine 2
  role: Framework hosting the Join aligner module and providing the data structures
    (master peak lists, alignment rows) the implementation operates on
  repo: https://github.com/mzmine/mzmine2
- name: google/styleguide
  role: Google Java Style Guide to be adhered to when writing the implementation in
    this collaborative project
  repo: https://github.com/google/styleguide
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:1a564f839c879a7e02a605a365b8542b77ba643e89b10d8f2d788eb8ca32527e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Java Development Kit (JDK) version 12
  - MZmine 2
  - google/styleguide
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# java-algorithm-implementation

## Summary

Implement a published LC-MS data-processing algorithm (e.g. the MZmine Join aligner) from scratch in Java using the JDK, then benchmark it against ground-truth alignments from synthetic lipidomic and real proteomic/metabolomic datasets by computing precision and recall per alignment.

## When to use

Use when reproducing or benchmarking a published peak-list alignment (or similar MS data-processing) algorithm in Java, where the method must be re-implemented from the paper's description and evaluated against known ground-truth tables.

## When NOT to use

- Input is already an aligned peak list or feature table — the aligner assumes unaligned per-sample peak lists as input
- Retention times among samples deviate non-linearly — the Join aligner cannot cope with non-linear RT deviation; use the RANSAC aligner instead
- The analysis goal is peak picking, deisotoping, or gap filling rather than aligning multiple peak lists

## Inputs

- Benchmark peak lists: 12 synthetic lipidomic datasets (20 samples each, seeded from a sample of 12 different lipidomic studies)
- Real proteomic datasets P1 and P2 from Lange et al. with tables of ground-truth alignments
- Real metabolomic datasets M1 and M2 from Lange et al. with tables of ground-truth alignments
- User-specified m/z and retention time tolerances for the alignment window

## Outputs

- Aligned peak lists (master peak list rows grouping matched peaks across samples)
- Per-dataset precision and recall values written to join_aligner_precision_recall.csv

## How to apply

First build the target toolchain from source: clone mzmine2 and run ./gradlew (or gradlew.bat) with JDK 12 or newer, following the Google Java Style Guide for collaborative code. Implement the algorithm in Java — for the Join aligner, create an empty master peak list, then for each peak find the best matching candidate in the master list within user-specified m/z and retention-time tolerance windows, creating a new master-list row when no candidate matches. Load the benchmark peak lists: the 12 synthetic lipidomic datasets (20 samples each, seeded from 12 lipidomic studies) and the real proteomic (P1, P2) and metabolomic (M1, M2) datasets from Lange et al. with their ground-truth alignment tables. Run the implementation per dataset, count true positives, false positives and false negatives against ground truth, and write per-dataset precision (TP/(TP+FP)) and recall (TP/(TP+FN)) values to CSV.

## Related tools

- **Java** (Language in which the Join aligner algorithm is implemented; MZmine 2 itself was developed using Java technology and is platform independent)
- **Java Development Kit (JDK) version 12** (Required build toolchain to compile MZmine 2 sources with gradlew before implementing or running the algorithm)
- **MZmine 2** (Framework hosting the Join aligner module and providing the data structures (master peak lists, alignment rows) the implementation operates on) — https://github.com/mzmine/mzmine2
- **google/styleguide** (Google Java Style Guide to be adhered to when writing the implementation in this collaborative project) — https://github.com/google/styleguide

## Examples

```
git clone https://github.com/mzmine/mzmine2 && cd mzmine2 && ./gradlew
```

## Evaluation signals

- Output file join_aligner_precision_recall.csv exists with precision and recall columns, each value in the range 0–100
- One CSV entry per benchmark alignment (13 alignments across the synthetic and real datasets)
- The implementation reproduces the published baseline behavior: Join aligner performs worse than RANSAC in 11 of 13 alignments
- Worse results than RANSAC occur only in the P2 dataset fraction 00 case, where row count after alignment exceeds 6800
- Build succeeds from source with ./gradlew under JDK 12 and the final MZmine distribution lands in build/MZmine-version-platform.zip

## Limitations

- The Join aligner cannot cope with a non-linear deviation of retention times among samples
- One disadvantage of the Join aligner is its dependence on fixed m/z and RT tolerance windows, making it sensitive to RT drift
- Join aligner serves as the weaker baseline — RANSAC alignment achieved 100% precision and recall on the synthetic datasets, better results in 11 of 13 alignments

## Evidence

- [other] Join aligner creates an empty master peak list, then aligns each sample peak to the best candidate of the master list: [evidence span withheld — anchor status: not_found]
- [other] Join aligner cannot cope with non-linear RT deviation: "One disadvantage of the Join aligner is the inability to cope with a non-linear deviation of the retention times among samples."
- [results] Benchmark datasets for evaluating the implementation: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [results] RANSAC vs Join aligner benchmark result: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [readme] JDK requirement for building and implementing: "MZmine development requires Java Development Kit (JDK) version 12 or newer (http://jdk.java.net)."
- [readme] Build command for the MZmine source tree: "To build the MZmine package from the sources, run the following command:

 ./gradlew

or

 gradlew.bat"
