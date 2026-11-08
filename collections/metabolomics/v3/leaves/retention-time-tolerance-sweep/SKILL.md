---
name: retention-time-tolerance-sweep
description: Use when comparing peak-list alignment algorithms on LC-MS peak lists
  with known ground-truth correspondences, and you need a tolerance sweep (e.g., 5/20/50
  s RT) scored by precision and recall.
when_to_use_negative:
- Data has no ground-truth alignments or known correspondences — precision/recall
  scoring against truth tables is not possible
- The task is general MS data processing (peak picking, deisotoping, gap filling,
  normalization) rather than aligner benchmarking
- Input is a single peak list from one sample — alignment requires multiple peak lists
  from a sample series
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Platform used to import peak lists, run the RANSAC and Join aligner modules,
    and export aligned peak lists for scoring
  repo: https://github.com/mzmine/mzmine2
- name: RANSAC aligner
  role: Alignment module under test; fits alignment functions to matching peak pairs
    via iterative random sample consensus and discards outliers
  repo: https://github.com/mzmine/mzmine2
- name: Join aligner
  role: Baseline alignment module used as the comparator across all RT tolerance sweeps
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Runtime and build technology for MZmine 2 (built with ./gradlew or gradlew.bat)
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:77678970eb24f5f9c942d6d3e8df50dd07bedc0240a4657839b970a60bf0a51d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - Join aligner
  - MZmine 2
  - RANSAC aligner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# retention-time-tolerance-sweep

## Summary

A benchmarking skill that sweeps peak-list alignment (RANSAC vs Join aligners in MZmine 2) across retention-time tolerances of 5 s, 20 s, and 50 s and scores each run against ground-truth alignments to quantify precision and recall.

## When to use

Use when comparing peak-list alignment algorithms on LC-MS peak lists with known ground-truth correspondences, and you need a tolerance sweep (e.g., 5/20/50 s RT) scored by precision and recall.

## When NOT to use

- Data has no ground-truth alignments or known correspondences — precision/recall scoring against truth tables is not possible
- The task is general MS data processing (peak picking, deisotoping, gap filling, normalization) rather than aligner benchmarking
- Input is a single peak list from one sample — alignment requires multiple peak lists from a sample series

## Inputs

- 12 synthetic lipidomic peak lists (20 samples each, seed sample expanded with random non-linear RT deviation)
- Real proteomic and metabolomic peak lists from Lange et al. (P1, P2, M1, M2) with ground-truth alignment tables
- Retention-time tolerance settings (5 s, 20 s, 50 s) and a fixed m/z tolerance

## Outputs

- Per-dataset, per-fraction, per-aligner, per-tolerance precision and recall values (e.g., ransac_precision_recall_results.csv)
- Comparison table covering the 13 pairwise alignments for RANSAC vs Join aligner

## How to apply

Import each dataset's peak lists into MZmine 2, preserving ground-truth alignment labels for scoring. For each dataset, run the Join aligner and the RANSAC aligner at RT tolerances of 5 s, 20 s, and 50 s while keeping the m/z tolerance fixed; RANSAC uses iterative model fitting to fit alignment functions to matching peak pairs and discard non-matching outliers. For synthetic lipidomic datasets (12 sets, 20 samples each derived from a seed sample with random non-linear RT deviation), score against the known synthetic correspondences; for the real Lange et al. datasets (P1, P2 proteomic, M1, M2 metabolomic), score against their published 'ground truth' alignment tables. Compute precision (fraction of aligned peak pairs that are correct) and recall (fraction of true peak pairs that are aligned) per dataset, fraction, aligner, and RT tolerance, then aggregate into a comparison table. Validate against the reported pattern: RANSAC alone reaches 100% precision and recall on the synthetic sets, and beats the Join aligner in 11 of 13 real alignments.

## Related tools

- **MZmine 2** (Platform used to import peak lists, run the RANSAC and Join aligner modules, and export aligned peak lists for scoring) — https://github.com/mzmine/mzmine2
- **RANSAC aligner** (Alignment module under test; fits alignment functions to matching peak pairs via iterative random sample consensus and discards outliers) — https://github.com/mzmine/mzmine2
- **Join aligner** (Baseline alignment module used as the comparator across all RT tolerance sweeps) — https://github.com/mzmine/mzmine2
- **Java** (Runtime and build technology for MZmine 2 (built with ./gradlew or gradlew.bat)) — https://github.com/mzmine/mzmine2

## Examples

```
/opt/mzmine/bin/mzmine -batch ransac_join_alignment_sweep.xml
```

## Evaluation signals

- On the 12 synthetic datasets, RANSAC must reach 100% precision and 100% recall at the tested tolerances, while Join aligner does not reach perfect alignment
- On the 13 Lange et al. pairwise alignments (P1 and P2 fractions, M1, M2), RANSAC outperforms Join aligner in 11 of 13
- RANSAC should be best on all P1 fractions and worse only on P2 dataset fraction 00
- Precision is computed as the fraction of aligned peak pairs that are correct and recall as the fraction of true peak pairs that are aligned — check both are defined and in [0, 1] for every dataset/tolerance cell
- The aggregated table covers all 12 datasets × 3 tolerances (synthetic) and 13 alignments × 3 tolerances (real) without missing cells

## Limitations

- The Join aligner's performance depends strongly on the fixed RT tolerance; results are only comparable when m/z tolerance is held constant
- RANSAC was worse than Join on P2 dataset fraction 00, attributed to the high number of features in that fraction (over 6800 rows after alignment)
- The manuscript's Tranche hash for downloading the associated data is referenced but absent from the text, so exact dataset retrieval may require external sourcing (Lange et al.)
- Validation in the paper was performed on LC-MS data; extension to other data types (e.g., GC-MS) is described only as possible future work

## Evidence

- [results] Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets.: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] 11 of 13 alignments, worse only on P2 fraction 00: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [supplementary] Supplementary precision/recall table description: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [results] Real benchmark datasets with ground truth: "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [readme] MZmine CLI batch flag from README: [evidence span withheld — anchor status: external_doc]
