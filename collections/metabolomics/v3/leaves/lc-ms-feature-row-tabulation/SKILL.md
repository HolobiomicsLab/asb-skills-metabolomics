---
name: lc-ms-feature-row-tabulation
description: Use when comparing alignment algorithms (RANSAC vs Join) on LC-MS peak
  lists and row counts are needed to explain why one aligner degrades on high-feature-count
  fractions.
when_to_use_negative:
- Input is already a single aligned feature table with no competing aligner to compare
  against — tabulation alone will not yield precision/recall.
- The datasets are not LC-MS peak lists with a corresponding ground-truth alignment;
  the article's modules were tested mainly on LC-MS data only.
- Only synthetic datasets are being analyzed; the tabulation here is grounded in the
  real P1/P2/M1/M2 benchmark rather than the 12 synthetic lipidomic sets.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Runs the RANSAC and Join alignment modules and exports the aligned peak lists
    whose feature row counts are tabulated
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Runtime platform for MZmine 2, enabling the platform-independent batch processing
    of the alignment evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:456a586e19f2a43efb7347ec9fd8f255b1d7c85faa41de47c6af52fcfec50559
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# LC-MS feature row tabulation

## Summary

Tabulates aligned peak-list feature rows across dataset fractions and evaluates alignment precision/recall against ground-truth alignments, using row count (e.g. over 6800 rows) to explain aligner performance differences such as the RANSAC failure on P2 fraction 00.

## When to use

Use when comparing alignment algorithms (RANSAC vs Join) on LC-MS peak lists and row counts are needed to explain why one aligner degrades on high-feature-count fractions.

## When NOT to use

- Input is already a single aligned feature table with no competing aligner to compare against — tabulation alone will not yield precision/recall.
- The datasets are not LC-MS peak lists with a corresponding ground-truth alignment; the article's modules were tested mainly on LC-MS data only.
- Only synthetic datasets are being analyzed; the tabulation here is grounded in the real P1/P2/M1/M2 benchmark rather than the 12 synthetic lipidomic sets.

## Inputs

- Lange et al. 'ground truth' alignment tables for P1, P2, M1, M2
- Alignment evaluation data from the ProteomeCommons.org Tranche deposit
- Peak lists produced by MZmine 2 aligners (RANSAC and Join)

## Outputs

- Tabulated precision and recall per aligner and fraction (e.g. alignment_reanalysis.tsv)
- Feature row counts per dataset fraction
- Flagged failure case (P2 fraction 00) with row count and parameter context

## How to apply

Import the aligned peak lists for the proteomic (P1, P2) and metabolomic (M1, M2) datasets from the Tranche deposit together with the Lange et al. 'ground truth' alignment tables. Compute precision and recall for each aligner (RANSAC and Join) against the ground truth for every fraction using MZmine 2's evaluation modules. Tabulate the resulting precision/recall values alongside the number of aligned feature rows per fraction — e.g. P2 fraction 00 exceeds 6800 rows — and against the retention-time tolerance settings used in the evaluation. When RANSAC performs worse than Join, check whether the fraction's row count is high enough to make building a suitable RANSAC model difficult; the article attributes the single failure case (P2 fraction 00) to exactly this high feature count. Export the tabulated values to a structured file such as alignment_reanalysis.tsv so the 11-of-13 superiority claim can be validated row by row.

## Related tools

- **MZmine 2** (Runs the RANSAC and Join alignment modules and exports the aligned peak lists whose feature row counts are tabulated) — https://github.com/mzmine/mzmine2
- **Java** (Runtime platform for MZmine 2, enabling the platform-independent batch processing of the alignment evaluation)

## Examples

```
python tabulate_alignment.py --datasets P1,P2,M1,M2 --aligners RANSAC,Join --ground-truth lange_tables/ --output alignment_reanalysis.tsv
```

## Evaluation signals

- Precision and recall per fraction reproduce the reported 'better results in 11 of 13 alignments, with worse results obtained in only a single case' outcome.
- P2 fraction 00 row count exceeds 6800 rows after alignment, matching the reported threshold for RANSAC difficulty.
- Both aligners (RANSAC and Join) are evaluated against the same ground-truth tables, so their precision/recall pairs are directly comparable.
- Row counts and retention-time tolerance settings are captured for every fraction so performance can be contextualised.
- The tabulated output is written to a structured file (e.g. alignment_reanalysis.tsv) rather than left in memory.

## Limitations

- RANSAC degraded on the single high-feature-count fraction (P2 fraction 00, over 6800 rows) because a suitable model was hard to build — tabulation should be checked for such row-count extremes.
- Developed modules were tested mainly on LC-MS data only; validation on other data types is not demonstrated.
- The specific Tranche hash for the deposited data is referenced but not present in the text, so reproducibility relies on the described dataset names (P1, P2, M1, M2) rather than a literal hash.

## Evidence

- [results] RANSAC provided better alignment results than Join in 11 of 13 alignments, worse in only one case (P2 dataset fraction 00): "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] P2 fraction 00 high feature count (over 6800 rows): "the high number of features in this fraction (over 6800 rows after alignment)"
- [results] Ground-truth alignment tables from Lange et al. used as evaluation reference: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [results] MZmine 2's core workflow includes alignment and gap filling, producing the feature rows that are tabulated: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
