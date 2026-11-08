---
name: ransac-alignment-performance-comparison
description: Decision support for choosing between RANSAC and Join aligners in MZmine
  when aligning peak lists across LC-MS fractions, especially when feature counts
  exceed thousands of rows and alignment quality must be quantified.
when_to_use_negative:
- Input is raw MS data not yet peak-picked — alignment comparison requires already-detected
  peak lists
- Only synthetic datasets with a single correct alignment are needed — for those,
  RANSAC already achieved 100% precision and recall, so a comparative Join benchmark
  adds little
- No ground-truth alignment table exists, making precision/recall computation impossible
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Runs both the RANSAC aligner and Join aligner on the P1/P2/M1/M2 peak lists
    and produces aligned peak lists for scoring
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Runtime platform for MZmine 2; building from sources via ./gradlew requires
    the Java Development Kit
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:59b43b1b76a283c2ff9129137bf9b0fd2229131f1278ab4887c0014977a11a3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ransac-alignment-performance-comparison

## Summary

Benchmarks the MZmine 2 RANSAC peak list aligner against the Join aligner by computing precision and recall against ground-truth alignment tables, and attributes performance degradation to high feature counts (e.g. the P2 fraction 00 case with over 6800 rows).

## When to use

Decision support for choosing between RANSAC and Join aligners in MZmine when aligning peak lists across LC-MS fractions, especially when feature counts exceed thousands of rows and alignment quality must be quantified.

## When NOT to use

- Input is raw MS data not yet peak-picked — alignment comparison requires already-detected peak lists
- Only synthetic datasets with a single correct alignment are needed — for those, RANSAC already achieved 100% precision and recall, so a comparative Join benchmark adds little
- No ground-truth alignment table exists, making precision/recall computation impossible

## Inputs

- Real proteomic (P1, P2) and metabolomic (M1, M2) peak lists with Lange et al. 'ground truth' alignment tables
- Alignment evaluation data downloaded from ProteomeCommons.org Tranche
- Retention-time tolerance settings used in the alignment evaluation

## Outputs

- Precision and recall values per aligner (RANSAC, Join) per dataset fraction, e.g. written to alignment_reanalysis.tsv
- Tabulation of precision/recall against number of aligned feature rows
- Flagged underperforming cases (e.g. P2 fraction 00) with row counts and parameter context

## How to apply

Import the alignment evaluation data from the ProteomeCommons.org Tranche deposit together with the Lange et al. 'ground truth' alignment tables for the proteomic (P1, P2) and metabolomic (M1, M2) datasets. Run both the RANSAC aligner and the Join aligner in MZmine 2 across all dataset fractions and retention-time tolerance settings, then compute precision and recall for each aligner against the ground-truth alignments. Tabulate precision and recall against the number of aligned feature rows (e.g. over 6800 rows for P2 fraction 00) so that performance can be correlated with feature count. Expect the RANSAC aligner to outperform Join in most alignments (11 of 13 in the reference study) but flag cases where RANSAC degrades on fractions with a very high number of features, where building a suitable model becomes difficult.

## Related tools

- **MZmine 2** (Runs both the RANSAC aligner and Join aligner on the P1/P2/M1/M2 peak lists and produces aligned peak lists for scoring) — https://github.com/mzmine/mzmine2
- **Java** (Runtime platform for MZmine 2; building from sources via ./gradlew requires the Java Development Kit) — https://github.com/mzmine/mzmine2

## Examples

```
./gradlew && mzmine -batch alignment_batch.xml --input P2_fraction00.mzmine --aligners RANSAC,Join
```

## Evaluation signals

- Computed precision/recall values reproduce the reported outcome: RANSAC better in 11 of 13 alignments, worse in only a single case
- The single worse case is P2 dataset fraction 00, whose aligned peak list contains over 6800 rows
- On the 12 synthetic datasets, RANSAC reaches 100% in both precision and recall
- Precision and recall are tabulated for both RANSAC and Join across every dataset fraction and retention-time tolerance setting
- Row counts recorded for the flagged fraction match the context (over 6800 rows after alignment)

## Limitations

- RANSAC alignment degrades on fractions with very high feature counts (over 6800 rows after alignment), making it difficult to build a suitable model
- Evaluation depends on availability of 'ground truth' alignment tables, which exist only for the specific Lange et al. P1/P2/M1/M2 datasets
- The Tranche hash referenced for downloading the evaluation data is not provided in the manuscript text

## Evidence

- [results] RANSAC better in 11 of 13 alignments: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] P2 fraction 00 high feature count: "the high number of features in this fraction (over 6800 rows after alignment)"
- [results] Synthetic dataset benchmark: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] Dataset provenance: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [readme] Build instructions: "To build the MZmine package from the sources, run the following command: ./gradlew or gradlew.bat"
