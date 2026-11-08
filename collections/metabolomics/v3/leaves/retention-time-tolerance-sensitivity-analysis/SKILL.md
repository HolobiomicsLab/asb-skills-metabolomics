---
name: retention-time-tolerance-sensitivity-analysis
description: Use when peak lists from multiple LC-MS samples have been aligned with
  configurable retention-time (RT) tolerance parameters and you need to test whether
  the aligner's precision/recall holds across tolerance settings and feature-row counts,
  or to identify dataset fractions where one aligner.
when_to_use_negative:
- The input is already a validated alignment-performance table and no re-tuning of
  RT tolerance or re-comparison against ground truth is required
- Only synthetic datasets with known exact correspondences are available and precision/recall
  is trivially 100% (as in the paper's synthetic sets), so a tolerance sensitivity
  sweep adds no information
- The goal is raw-data processing (peak detection, deisotoping, gap filling) rather
  than evaluating alignment parameter robustness
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Runs the RANSAC and Join peak-list alignment modules whose RT tolerance parameters
    are being swept, and produces the aligned feature lists evaluated here
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: MZmine 2 is platform-independent and is developed/run with Java, so the alignment
    evaluation is executed in a Java environment (JDK 12 or newer for the mzmine2
    codebase)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:6016545d2ec46c47f63e2e2d045c261f6745eeb36626836a0d397cbaf7032880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# retention-time-tolerance-sensitivity-analysis

## Summary

Sensitivity analysis skill that evaluates how alignment precision and recall of peak-list aligners (e.g., RANSAC vs Join in MZmine 2) vary with retention-time tolerance settings and dataset feature density, to flag settings where a chosen aligner degrades.

## When to use

Use when peak lists from multiple LC-MS samples have been aligned with configurable retention-time (RT) tolerance parameters and you need to test whether the aligner's precision/recall holds across tolerance settings and feature-row counts, or to identify dataset fractions where one aligner underperforms another.

## When NOT to use

- The input is already a validated alignment-performance table and no re-tuning of RT tolerance or re-comparison against ground truth is required
- Only synthetic datasets with known exact correspondences are available and precision/recall is trivially 100% (as in the paper's synthetic sets), so a tolerance sensitivity sweep adds no information
- The goal is raw-data processing (peak detection, deisotoping, gap filling) rather than evaluating alignment parameter robustness

## Inputs

- Aligned peak-list / feature tables for each dataset fraction (proteomic P1, P2; metabolomic M1, M2)
- Ground-truth alignment tables published with the reference study (Lange et al.)
- Retention-time tolerance parameter settings used in the alignment evaluation
- Tranche-deposited alignment evaluation data (ProteomeCommons.org Tranche hash referenced in the paper)

## Outputs

- Per-fraction precision and recall values for each aligner (RANSAC, Join) at each retention-time tolerance setting
- A sensitivity table relating precision/recall to feature-row count (e.g., over 6800 aligned rows for P2 fraction 00)
- Flagged degradation cases with row-count and parameter context (e.g., P2 fraction 00 for RANSAC)
- Structured result file such as alignment_reanalysis.tsv

## How to apply

Import the aligned feature tables for each dataset fraction together with tables of 'ground truth' alignments (in this paper, the real proteomic P1 and P2 and metabolomic M1 and M2 datasets from Lange et al.). Compute precision and recall for each aligner — e.g., the RANSAC aligner and the Join aligner in MZmine 2 — against the ground-truth alignment for every fraction. Vary the retention-time tolerance parameters used in the alignment evaluation and tabulate precision/recall against both tolerance settings and the aligned row count (e.g., P2 fraction 00 with over 6800 rows after alignment). Flag cases where performance degrades: here RANSAC was better in 11 of 13 alignments but worse in a single case (P2 fraction 00), attributed to that fraction's high feature count making it difficult for RANSAC to build a suitable model. Write the per-fraction, per-tolerance results to a structured table (e.g., alignment_reanalysis.tsv) so the sensitivity trend is auditable. Validate that the re-computed precision/recall values reproduce the reported outcome before drawing any conclusion about aligner choice.

## Related tools

- **MZmine 2** (Runs the RANSAC and Join peak-list alignment modules whose RT tolerance parameters are being swept, and produces the aligned feature lists evaluated here) — https://github.com/mzmine/mzmine2
- **Java** (MZmine 2 is platform-independent and is developed/run with Java, so the alignment evaluation is executed in a Java environment (JDK 12 or newer for the mzmine2 codebase))

## Examples

```
mzmine --batch alignment_sweep.batch  # run RANSAC and Join aligner modules over P1/P2/M1/M2 fraction peak lists at each RT tolerance setting, then recompute precision/recall against the Lange et al. ground-truth tables and write alignment_reanalysis.tsv
```

## Evaluation signals

- Recomputed precision and recall for each fraction match the published ground-truth evaluation within reporting tolerance
- Outcome distribution reproduces the paper's finding: RANSAC better in 11 of 13 alignments, worse in only one case (P2 dataset fraction 00)
- The flagged degradation case corresponds to the documented feature density — over 6800 rows after alignment in P2 fraction 00
- Results table covers all evaluated RT tolerance settings for both aligners, so the sensitivity trend is visible rather than a single-point comparison
- All four real datasets (P1, P2, M1, M2) are represented in the output; no fraction is silently dropped

## Limitations

- Developed modules (including the aligners under test) were validated mainly on LC-MS data; extension to other data types is not demonstrated
- On very dense fractions (>6800 aligned rows), the RANSAC aligner may fail to build a suitable model and yield worse results than the Join aligner
- The 'ground truth' alignments come from a prior study (Lange et al.), so evaluation quality depends on the fidelity of those reference tables
- The Tranche hash for the underlying data is referenced in the paper but not given in the text, so data retrieval may require external lookup

## Evidence

- [results] RANSAC aligner performance vs Join and the P2 00 degradation: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] Feature count of the degraded fraction: "the high number of features in this fraction (over 6800 rows after alignment)"
- [results] Ground-truth-based evaluation datasets: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [results] RANSAC performance on synthetic control sets: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [discussion] Tranche data availability caveat: "The data associated with this manuscript may be downloaded from ProteomeCommons.org Tranche using the following hash:"
