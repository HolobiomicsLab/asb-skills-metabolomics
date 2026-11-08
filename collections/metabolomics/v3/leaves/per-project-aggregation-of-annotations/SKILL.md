---
name: per-project-aggregation-of-annotations
description: Use when a training or evaluation dataset for EIC peak-quality annotation
  spans multiple LC-MS metabolomics projects and you need per-project good/poor label
  counts to verify dataset balance before CNN training.
when_to_use_negative:
- Input is a raw feature table with m/z and RT columns destined for pyEVA classification
  — this skill operates on pre-existing manual annotations, not on feature tables
  being scored
- You need to run the CNN model on new mzXML/mzML files to predict peak quality —
  use pyEVA's main.py pipeline instead of aggregating existing labels
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: EVA
  role: C#-based Windows program embedding the trained CNN that generated the EIC
    quality annotations being aggregated; the annotation dataset derives from the
    EVA project
  repo: github.com/HuanLab-backup/EVA
- name: pyEVA
  role: Python rewrite of EVA in the HuanLab ecosystem whose project the annotated
    EIC dataset is associated with
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:12f5ada6b3158e018141d996fc31df6e9f6ff9c94d3334341336651c84a27dc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# per-project-aggregation-of-annotations

## Summary

Aggregates manually annotated EIC plot quality labels (good vs poor) across the 22 LC-MS metabolomics projects used to train the EVA CNN, producing per-project counts and percentages of each label class plus a total row summing to 25,095 annotated plots.

## When to use

Use when a training or evaluation dataset for EIC peak-quality annotation spans multiple LC-MS metabolomics projects and you need per-project good/poor label counts to verify dataset balance before CNN training.

## When NOT to use

- Input is a raw feature table with m/z and RT columns destined for pyEVA classification — this skill operates on pre-existing manual annotations, not on feature tables being scored
- You need to run the CNN model on new mzXML/mzML files to predict peak quality — use pyEVA's main.py pipeline instead of aggregating existing labels

## Inputs

- Manually inspected EIC plot annotation dataset (25,095 plots with good/poor quality labels) with per-project identifiers for the 22 LC-MS metabolomics projects

## Outputs

- eic_label_distribution.csv — one row per project plus a total row, with counts and percentages of good vs poor EIC labels per project

## How to apply

Load the manually inspected EIC plot dataset (25,095 images with good/poor quality annotations) from the HuanLab EVA repository and parse the per-plot quality label and project identifier for each of the 22 LC-MS-based metabolomics projects. Count plots labeled good and plots labeled poor both overall and within each project, then compute the percentage of each class per project. Aggregate these counts and percentages into a single summary table with one row per project plus a total row, and write it out as eic_label_distribution.csv. Validate that the table's total row count equals the reported 25,095 manually annotated EIC plots collected from 22 projects of various sample types, LC and MS conditions.

## Related tools

- **EVA** (C#-based Windows program embedding the trained CNN that generated the EIC quality annotations being aggregated; the annotation dataset derives from the EVA project) — github.com/HuanLab-backup/EVA
- **pyEVA** (Python rewrite of EVA in the HuanLab ecosystem whose project the annotated EIC dataset is associated with) — https://github.com/HuanLab/pyEVA

## Examples

```
import pandas as pd; df = pd.read_csv('annotations.csv'); agg = df.groupby(['project'])['label'].value_counts(normalize=True).unstack(fill_value=0)*100; agg.to_csv('eic_label_distribution.csv')
```

## Evaluation signals

- Total row count in eic_label_distribution.csv equals 25,095, matching the reported number of manually annotated EIC plots
- Exactly 22 project rows appear in the per-project table, one for each LC-MS-based metabolomics project
- Per-project good and poor counts sum to the total across all 22 projects with no missing or double-counted plots
- Good + poor percentages per project each sum to 100%, confirming no plots carry ambiguous or missing labels

## Limitations

- The aggregation depends on labels having been assigned via manual inspection of all EIC plots; if any plots lack labels the total will not reach 25,095
- Coverage is restricted to the 22 LC-MS-based metabolomics projects of various sample types, LC and MS conditions described in the paper — extrapolating label distributions to other projects is unsupported

## Evidence
