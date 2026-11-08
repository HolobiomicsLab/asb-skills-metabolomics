---
name: lc-im-ms-feature-count-comparison
description: Use when comparing replicate-consistent feature counts between original
  and preprocessed LC-IM-MS data to quantify whether smoothing increased reproducible
  feature detection in a lipidomics or omics workflow.
when_to_use_negative:
- The data are raw MS files rather than already-generated feature lists — this skill
  evaluates downstream feature counts, it does not itself find features (use Lipid
  Annotator or a feature-finder after preprocessing with the PreProcessor).
- The analysis goal is lipid annotation throughput (e.g. the 19.4% increase in annotations
  for features found in at least 2 replicates) rather than replicate-consistent MS1
  feature detection counts.
- Input data are not replicate LC-IM-MS runs (e.g. single-shot acquisitions), so replicate-set
  intersection across 3 replicates is undefined.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PNNL PreProcessor
  role: Standalone tool that applies multidimensional smoothing, demultiplexing, intensity
    thresholding, and spike removal to raw LC-IM-MS TOF data to generate the preprocessed
    feature lists compared in this skill
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: R
  role: Used with pandas-style tabular loading and set intersection to compute replicate-consistent
    feature counts and the relative percentage increase
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:e47ae24ee5c5d08bf7d41bc44a68e2125073c1392c924157d9b23f791c8003dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PNNL PreProcessor
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lc-im-ms-feature-count-comparison

## Summary

Quantifies how multidimensional smoothing of LC-IM-MS raw data (via PNNL PreProcessor) changes the number of MS1 lipid features consistently detected across replicates, by intersecting replicate feature lists for original vs preprocessed data and computing the relative percentage increase.

## When to use

Use when comparing replicate-consistent feature counts between original and preprocessed LC-IM-MS data to quantify whether smoothing increased reproducible feature detection in a lipidomics or omics workflow.

## When NOT to use

- The data are raw MS files rather than already-generated feature lists — this skill evaluates downstream feature counts, it does not itself find features (use Lipid Annotator or a feature-finder after preprocessing with the PreProcessor).
- The analysis goal is lipid annotation throughput (e.g. the 19.4% increase in annotations for features found in at least 2 replicates) rather than replicate-consistent MS1 feature detection counts.
- Input data are not replicate LC-IM-MS runs (e.g. single-shot acquisitions), so replicate-set intersection across 3 replicates is undefined.

## Inputs

- Detected-feature tables for the original-data replicates of the NIST SRM 1950 human plasma lipid extract LC-DT IM-MS runs (from MassIVE MSV000087295)
- Detected-feature tables for the PNNL PreProcessor-preprocessed replicates of the same lipid extract
- Choice of replicate count threshold (features consistently detected across 3 replicates)

## Outputs

- Per-condition counts of features consistently detected across all replicates (original vs preprocessed)
- Relative percentage increase in replicate-consistent features, written to replicate_feature_counts.csv

## How to apply

Download the deposited original and preprocessed detected-feature tables for the replicate LC-IM-MS runs (e.g. the 3 replicates of the NIST SRM 1950 human plasma lipid extract from MassIVE ftp://massive.ucsd.edu/MSV000087295) and load each replicate's original-data and preprocessed-data feature list into pandas, treating a feature as an ion with one or multiple isotopic peaks detected in the MS1 data. For each condition (original, preprocessed), intersect the replicate-specific feature sets to count features consistently detected across all 3 replicates. Compute the relative increase as (count_preprocessed - count_original) / count_original * 100 and write per-condition counts plus the percentage to a CSV. The rationale is that multidimensional smoothing (3 points in LC and IM dimensions) improves peak shapes of poorly defined low-abundance signals, so gains should concentrate among low-abundance ions; the PreProcessor workflow applies LC compression (4 frames become 1), IM interpolation (every drift bin becomes 3), demultiplexing with 75% pulses, smoothing (3 points in LC and 7 in IM), an intensity threshold of 50 counts, and 1 adjacent point spike removal.

## Related tools

- **PNNL PreProcessor** (Standalone tool that applies multidimensional smoothing, demultiplexing, intensity thresholding, and spike removal to raw LC-IM-MS TOF data to generate the preprocessed feature lists compared in this skill) — https://omics.pnl.gov/software/pnnl-preprocessor
- **R** (Used with pandas-style tabular loading and set intersection to compute replicate-consistent feature counts and the relative percentage increase)

## Examples

```
import pandas as pd; counts = {c: len(set.intersection(*[set(pd.read_csv(f'{c}_rep{r}_features.csv')['feature_id']) for r in (1,2,3)])) for c in ('original','preprocessed')}; print((counts['preprocessed']-counts['original'])/counts['original']*100)
```

## Evaluation signals

- Recomputed relative increase should fall within approximately 70–71%, matching the paper's reported 70.7% increase in features consistently detected across 3 replicates after smoothing.
- The preprocessed condition should yield a strictly higher replicate-consistent feature count than the original condition.
- Per-condition counts must be integers derived from the exact intersection of the 3 replicate feature sets per condition, not unions or majority counts.
- The largest gains should appear among low-abundance ions, consistent with the paper's finding that multidimensional smoothing improves peak shapes of poorly defined low-abundance signals.

## Limitations

- Results depend on the PreProcessor parameterization (e.g. smoothing with 3 points in LC and 7 in IM, 50-count intensity threshold, 1 adjacent point spike removal); the article specifies no single tool version.
- The 70.7% figure applies to replicate-consistent MS1 feature detection; the annotation-level gain is smaller (19.4% for features found in at least 2 replicates).
- The comparison is specific to LC-DT IM-MS data of a standard human plasma lipid extract; extension to other IM-MS instrumentation requires vendor support or converting to UIMF as an intermediate step.

## Evidence
