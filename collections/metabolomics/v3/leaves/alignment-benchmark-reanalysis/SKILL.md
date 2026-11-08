---
name: alignment-benchmark-reanalysis
description: Use when re-evaluating an LC-MS peak-list alignment algorithm against
  published ground-truth alignments and you need precision/recall recomputed per dataset
  fraction and per feature count, especially for RANSAC vs Join comparisons on proteomic
  (P1, P2) and metabolomic (M1, M2) data.
when_to_use_negative:
- When the input is raw LC-MS data (mzML/NetCDF) and you need peak detection or deisotoping
  rather than alignment re-scoring.
- When no published ground-truth alignment table exists for your dataset, since precision/recall
  cannot be computed without it.
- When evaluating non-alignment modules (peak identification, isotope grouping, gap
  filling) that this benchmark does not exercise.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MZmine 2
  role: Executes both RANSAC and Join peak-list alignment algorithms and reports precision/recall
    against ground-truth tables
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Underlying platform for scripting or extending the alignment modules in MZmine
    2 when re-running the benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:a8e6b0feb53a7c81e2ec631b9c4e8cc1e85bdfd847dd3c53f0f09371789e239e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# alignment-benchmark-reanalysis

## Summary

Reproduce and stress-test peak-list alignment benchmarks (RANSAC vs Join aligner) from MZmine 2 against Lange et al. ground-truth alignment tables, quantifying how feature-count inflation (e.g. P2 fraction 00 with >6800 aligned rows) degrades RANSAC model fitting.

## When to use

Use when re-evaluating an LC-MS peak-list alignment algorithm against published ground-truth alignments and you need precision/recall recomputed per dataset fraction and per feature count, especially for RANSAC vs Join comparisons on proteomic (P1, P2) and metabolomic (M1, M2) data.

## When NOT to use

- When the input is raw LC-MS data (mzML/NetCDF) and you need peak detection or deisotoping rather than alignment re-scoring.
- When no published ground-truth alignment table exists for your dataset, since precision/recall cannot be computed without it.
- When evaluating non-alignment modules (peak identification, isotope grouping, gap filling) that this benchmark does not exercise.

## Inputs

- LC-MS peak lists from real proteomic datasets P1 and P2 and metabolomic datasets M1 and M2 (Lange et al. datasets)
- Ground-truth alignment tables published by Lange et al. for P1, P2, M1, and M2 fractions
- Synthetic lipidomic benchmark datasets (12 synthetic sets of 20 samples each) as a sanity baseline
- Alignment parameter settings (retention-time tolerance) used in the original benchmark

## Outputs

- Per-fraction precision and recall values for RANSAC and Join aligners (TSV, e.g. alignment_reanalysis.tsv)
- Feature-row counts per aligned fraction (e.g. >6800 rows for P2 fraction 00)
- Recomputed win/loss tally (e.g. RANSAC better in 11 of 13, worse in 1)
- Precision/recall baseline results on the 12 synthetic datasets

## How to apply

Import the real proteomic (P1, P2) and metabolomic (M1, M2) datasets introduced by Lange et al. together with their 'ground truth' alignment tables, then run both the RANSAC aligner and the Join aligner in MZmine 2 across all 13 dataset fractions, recording precision and recall for each. Tabulate the number of aligned feature rows per fraction (e.g. P2 fraction 00 exceeds 6800 rows after alignment) and correlate row count with the RANSAC model-building difficulty, since the paper attributes the single RANSAC failure (P2 fraction 00) to the high feature count preventing it from constructing a suitable alignment model. Validate your recomputed precision/recall values against the published outcome of 11 of 13 alignments favoring RANSAC and 1 case (P2 fraction 00) favoring Join, writing the full comparison to a TSV such as alignment_reanalysis.tsv. Use MZmine 2's alignment module directly or its Java API if re-parameterizing the retention-time tolerance sweep; the paper notes the RANSAC aligner achieved 100% precision and recall on 12 synthetic datasets, so synthetic results should be checked first as a sanity baseline before the real-data benchmark.

## Related tools

- **MZmine 2** (Executes both RANSAC and Join peak-list alignment algorithms and reports precision/recall against ground-truth tables) — https://github.com/mzmine/mzmine2
- **Java** (Underlying platform for scripting or extending the alignment modules in MZmine 2 when re-running the benchmark)

## Examples

```
java -jar mzmine2.jar -batch alignment_batch.xml  # runs RANSAC & Join aligners; export precision/recall to alignment_reanalysis.tsv
```

## Evaluation signals

- Recomputed precision and recall for RANSAC on the 12 synthetic datasets must reach 100% on both metrics.
- Across the 13 real-dataset alignments, the win/loss tally must reproduce RANSAC better in 11 of 13 and worse in exactly 1 (P2 fraction 00).
- The single failing fraction must be identifiable by a feature-row count exceeding 6800 after alignment (P2 fraction 00).
- Output TSV must contain per-fraction rows with precision, recall, aligner name, and row count columns.

## Limitations

- The RANSAC aligner can fail when the feature count is very high (e.g. P2 fraction 00 with over 6800 aligned rows), because it struggles to build a suitable alignment model.
- The original benchmark was validated mainly on LC-MS data; extension to GC-MS or proteomics search-engine interoperation was only suggested, not demonstrated.
- The manuscript references a Tranche data deposit hash but the hash value itself is absent from the text, complicating direct dataset retrieval.

## Evidence

- [results] RANSAC vs Join win/loss on real data: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Feature-count attribution for the failure: "the high number of features in this fraction (over 6800 rows after alignment)"
- [results] Synthetic-dataset ceiling as sanity baseline: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] Ground-truth source and datasets: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [readme] README confirms archived MZmine 2 repository: "This repository is only there as a reference to the old MZmine 2 that was succeeded by mzmine."
