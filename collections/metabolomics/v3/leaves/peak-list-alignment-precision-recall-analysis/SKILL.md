---
name: peak-list-alignment-precision-recall-analysis
description: Use when comparing peak list alignment algorithms on LC-MS feature tables
  where a 'ground truth' alignment table exists, and you need precision/recall metrics
  to decide which aligner (e.g., RANSAC vs Join) performs better on specific dataset
  fractions.
when_to_use_negative:
- Input is raw LC-MS data before peak detection and deisotoping — this skill evaluates
  peak list alignment only, not the upstream import, filtering/smoothing, peak picking,
  or gap-filling stages.
- No 'ground truth' alignment table is available for the dataset, so precision and
  recall cannot be computed against a reference.
- The goal is peak identification against online databases (PubChem, KEGG, METLIN,
  HMDB) rather than alignment evaluation across multiple peak lists.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Runs the RANSAC and Join peak list alignment modules whose outputs are scored
    against ground-truth alignment tables
  repo: http://mzmine.sourceforge.net/
- name: Java
  role: Platform on which MZmine 2 alignment modules are executed and evaluation code
    is built
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:43a7282b66e3c63186570809bff66078b0cb0fd039929f8329189c6181f5c7a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# peak-list-alignment-precision-recall-analysis

## Summary

Evaluate peak list alignment algorithms (RANSAC aligner vs Join aligner in MZmine 2) by computing precision and recall against 'ground truth' alignment tables from Lange et al., across real proteomic (P1, P2) and metabolomic (M1, M2) datasets plus synthetic lipidomic datasets.

## When to use

Use when comparing peak list alignment algorithms on LC-MS feature tables where a 'ground truth' alignment table exists, and you need precision/recall metrics to decide which aligner (e.g., RANSAC vs Join) performs better on specific dataset fractions.

## When NOT to use

- Input is raw LC-MS data before peak detection and deisotoping — this skill evaluates peak list alignment only, not the upstream import, filtering/smoothing, peak picking, or gap-filling stages.
- No 'ground truth' alignment table is available for the dataset, so precision and recall cannot be computed against a reference.
- The goal is peak identification against online databases (PubChem, KEGG, METLIN, HMDB) rather than alignment evaluation across multiple peak lists.

## Inputs

- MZmine 2 peak list alignment outputs for the RANSAC and Join aligners
- 'Ground truth' alignment tables for the Lange et al. P1, P2 (proteomic) and M1, M2 (metabolomic) datasets
- Tranche deposit alignment evaluation data (ProteomeCommons.org Tranche hash from the manuscript)
- 12 synthetic lipidomic datasets seeded from 12 lipidomic study samples, each expanded to 20 samples

## Outputs

- Per-fraction precision and recall values for the RANSAC aligner
- Per-fraction precision and recall values for the Join aligner
- Precision/recall vs. aligned-row-count table (e.g., P2 fraction 00 > 6800 rows) written to alignment_reanalysis.tsv
- Comparison verdict per dataset fraction identifying which aligner performed better

## How to apply

Import the alignment evaluation data from the ProteomeCommons.org Tranche deposit together with the Lange et al. 'ground truth' alignment tables for the P1, P2, M1, and M2 proteomic and metabolomic datasets. Run each aligner (RANSAC and Join) in MZmine 2 over each dataset fraction, then compute precision and recall for each aligner output against the ground-truth alignments for every fraction. Tabulate precision and recall against the number of aligned feature rows — e.g., the P2 fraction 00 case exceeds 6800 rows after alignment — and across the retention-time tolerance settings used in the alignment evaluation. Flag cases where one aligner underperforms and record its row count and parameter context: the paper attributes the single RANSAC failure (P2 fraction 00) to that fraction's high feature count making it difficult for RANSAC to build a suitable model. Validate that the computed values reproduce the reported outcome — RANSAC better in 11 of 13 alignments with worse results in only a single case — and on synthetic data only the RANSAC algorithm achieved 100% in both precision and recall. Write the result to a table such as alignment_reanalysis.tsv.

## Related tools

- **MZmine 2** (Runs the RANSAC and Join peak list alignment modules whose outputs are scored against ground-truth alignment tables) — http://mzmine.sourceforge.net/
- **Java** (Platform on which MZmine 2 alignment modules are executed and evaluation code is built)

## Examples

```
python analyze_alignment.py --ground-truth lange_p1_p2_m1_m2.csv --ransac ransac_alignment.tsv --join join_alignment.tsv --out alignment_reanalysis.tsv
```

## Evaluation signals

- On synthetic datasets, the RANSAC aligner should reach 100% precision and 100% recall; any lower value indicates a parameter or data-loading error.
- Across the 13 real dataset alignments, the RANSAC aligner should outperform Join in 11 cases and underperform in exactly one (P2 dataset fraction 00); deviations flag tabulation mistakes.
- The single RANSAC underperformance case must coincide with the fraction containing over 6800 aligned rows (P2 fraction 00); row counts outside this range suggest incorrect fraction handling.
- Precision and recall values must be reproducible when re-run across the retention-time tolerance settings used in the alignment evaluation; instability indicates wrong tolerance parameters.
- The output table (alignment_reanalysis.tsv) must contain per-fraction precision, recall, row count, and aligner verdict columns consistent with the reported 11-of-13 outcome.

## Limitations

- The single RANSAC failure (P2 fraction 00, over 6800 aligned rows) is attributed to the high feature count making it difficult for RANSAC to build a suitable model — expect degraded RANSAC performance on very large feature lists.
- The evaluation is grounded on the Lange et al. 'ground truth' alignment tables; results are only as reliable as those reference alignments.
- Developed and tested mainly on LC-MS data; the article notes the modules were tested mainly on LC-MS data and other data types are future expansion.
- The Tranche hash value for the evaluation data is referenced but not present in the article text, so the exact deposit cannot be resolved from the manuscript alone.

## Evidence

- [results] RANSAC vs Join benchmark outcome on real data: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [results] RANSAC 100% precision and recall on synthetic datasets: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] Feature-count explanation for the single RANSAC failure: "the high number of features in this fraction (over 6800 rows after alignment)"
- [results] Evaluation dataset source (Lange et al. ground truth tables): "to use the real proteomic (P1 and P2) and metabolomic (M1 and M2) datasets introduced by Lange et al. [11], together with their tables of "ground truth" alignments"
- [readme] Readme note that development moved to mzmine 3; MZmine 2 is retained as reference: "All development happens in a new repository: [mzmine GitHub](https://github.com/mzmine/mzmine3)"
