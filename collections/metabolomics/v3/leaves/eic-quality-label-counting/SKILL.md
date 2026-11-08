---
name: eic-quality-label-counting
description: 'Use when characterizing the training data behind a peak-quality classifier:
  parse per-plot good/poor quality labels from manually annotated EIC plots and aggregate
  counts per project.'
when_to_use_negative:
- Input is raw unprocessed metabolomics data rather than already-labeled EIC plot
  annotations — peak picking and EIC extraction (e.g. via xcms in the pyEVA pipeline)
  is a prerequisite, not this skill's scope
- The goal is to classify new, unlabeled EIC plots with the trained model (use EVA/pyEVA
  inference) instead of summarizing the label distribution of the training corpus
- The EIC plots were not manually inspected and lack good/poor quality labels, so
  per-project label counts cannot be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EVA
  role: Source of the 25095 manually annotated EIC plot training corpus from 22 LC-MS-based
    metabolomics projects whose label distribution is being aggregated
  repo: github.com/HuanLab-backup/EVA
- name: pyEVA
  role: Python version of EVA in whose repository context the manually labeled EIC
    plot dataset resides; used for the EIC plots underlying the training labels
  repo: https://github.com/HuanLab/pyEVA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01309
    title: 'EVA: Evaluation of Metabolic Feature Fidelity Using a Deep Learning Model
      Trained With Over 25000 Extracted Ion Chromatograms'
  source_content_hash: sha256:360393a45dc7b1eec29aa0eeab217a3b8d09eeda98dbe0f641f13c3e6c615673
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EVA
  - pyEVA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# eic-quality-label-counting

## Summary

Summarizes the good/poor EIC quality labels from the manually inspected 25095-plot training corpus across 22 LC-MS metabolomics projects, producing a per-project and overall label distribution table that documents the dataset behind the EVA CNN model.

## When to use

Use when characterizing the training data behind a peak-quality classifier: parse per-plot good/poor quality labels from manually annotated EIC plots and aggregate counts per project.

## When NOT to use

- Input is raw unprocessed metabolomics data rather than already-labeled EIC plot annotations — peak picking and EIC extraction (e.g. via xcms in the pyEVA pipeline) is a prerequisite, not this skill's scope
- The goal is to classify new, unlabeled EIC plots with the trained model (use EVA/pyEVA inference) instead of summarizing the label distribution of the training corpus
- The EIC plots were not manually inspected and lack good/poor quality labels, so per-project label counts cannot be computed

## Inputs

- Manually annotated EIC plot dataset (25095 plots with good/poor quality labels) from 22 LC-MS-based metabolomics projects, associated with the EVA project
- Per-plot quality labels and project identifiers from the HuanLab EVA repository

## Outputs

- eic_label_distribution.csv — per-project and total counts/percentages of good vs poor EIC quality labels

## How to apply

Load the manually inspected EIC plot dataset (25095 EIC images annotated good or poor EIC quality) associated with the EVA project from the HuanLab EVA repository, which spans 22 LC-MS-based metabolomics projects covering various sample types, LC and MS conditions. Parse the per-plot quality labels and project identifiers, then count plots labeled good and labeled poor both overall and per project. Aggregate counts and percentages into a single summary table (eic_label_distribution.csv) with one row per project plus a total row. Validate that the total row count equals the reported 25095 manually annotated EIC plots, since the article states all EIC plots were manually inspected to assign good or poor EIC quality for accurate CNN model training.

## Related tools

- **EVA** (Source of the 25095 manually annotated EIC plot training corpus from 22 LC-MS-based metabolomics projects whose label distribution is being aggregated) — github.com/HuanLab-backup/EVA
- **pyEVA** (Python version of EVA in whose repository context the manually labeled EIC plot dataset resides; used for the EIC plots underlying the training labels) — https://github.com/HuanLab/pyEVA

## Examples

```
python main.py  # run inside the pyEVA conda environment after `conda env create --file environment.yml`, with the manually labeled EIC dataset in `input`
```

## Evaluation signals

- Verify eic_label_distribution.csv exists and its total row count equals 25095, matching the reported manually annotated EIC plots
- Confirm the table covers exactly 22 LC-MS-based metabolomics projects plus a total row
- Check that good + poor counts per project sum to that project's plot count, and overall good + poor equals 25095
- Verify percentages in the summary are consistent with raw counts (each percentage recomputable from the count columns)

## Limitations

- Requires complete manual inspection labels: plots lacking a good/poor quality annotation cannot contribute to the distribution
- Distribution reflects only the 22 projects used for EVA CNN training; it does not characterize other metabolomics datasets or unlabeled EIC plots
- pyEVA (Python rewrite of EVA) ships as a runnable program with environment and package dependencies (Anaconda/conda environment.yml, pip requirements, R xcms/ggplot2/scales/dplyr) that must be set up before its demo data or outputs can be used

## Evidence

- [readme] pyEVA provenance and rewrite context: [evidence span withheld — anchor status: not_found]
- [readme] pyEVA EIC processing pipeline context: [evidence span withheld — anchor status: external_doc]
