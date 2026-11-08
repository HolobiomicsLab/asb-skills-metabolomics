---
name: error-heatmap-visualization
description: Use when after per-metric/per-sample-group Isolation Forest outlier detection
  on precursor-ion peak metrics you need heatmaps showing which MS runs deviate on
  which metric, to localize instrument performance issues (e.g., mass calibration
  drift, RT instability).
when_to_use_negative:
- You need a single combined QC score per run — PeakQC's outlier detection and heatmaps
  operate per metric/per group, not on an aggregated quality score
- Input is not precursor-ion peak metric data (e.g., raw vendor files alone) — metrics
  must first be extracted for auto-tracked or user-specified ions
- Comparing samples across groups — anomaly detection and heatmap rendering are performed
  within each sample group only
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Performs the per-metric/per-group Isolation Forest outlier detection and generates
    the error heatmaps (JPG/PDF) and outlier CSV reports
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:270409a93a2ebfd348b78de0d0776a475ca5ef1f03587126c24085a458aceccd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - PeakQC
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# error-heatmap-visualization

## Summary

Visualize anomalous MS runs identified by Isolation Forest outlier detection as error heatmaps that pair each flagged run with the defective precursor-ion peak metric (mass error, retention time error, or arrival time error) within its sample group, as implemented in the PeakQC QC pipeline.

## When to use

Use when after per-metric/per-sample-group Isolation Forest outlier detection on precursor-ion peak metrics you need heatmaps showing which MS runs deviate on which metric, to localize instrument performance issues (e.g., mass calibration drift, RT instability).

## When NOT to use

- You need a single combined QC score per run — PeakQC's outlier detection and heatmaps operate per metric/per group, not on an aggregated quality score
- Input is not precursor-ion peak metric data (e.g., raw vendor files alone) — metrics must first be extracted for auto-tracked or user-specified ions
- Comparing samples across groups — anomaly detection and heatmap rendering are performed within each sample group only

## Inputs

- CSV reports of extracted peak metrics (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions
- Sample-group partitioning of the MS runs in the batch
- Outlier table (CSV) of runs flagged by Isolation Forest per metric and sample group

## Outputs

- Error heatmaps (JPG and PDF) showing per-metric deviations of flagged anomalous MS runs
- Structured outlier table (CSV) with metric, sample group, and run identifiers

## How to apply

First run outlier detection separately for each extracted peak metric (mass error, retention time error, arrival time error) within each sample group, using the Isolation Forest algorithm that detects anomalies with binary trees, has linear time complexity and a low memory requirement. Compile all flagged runs into a structured outlier table (CSV) recording the metric, sample group, and run identifiers. Then render error heatmaps (JPG and PDF) in which the deviations of the flagged runs are displayed per metric so each anomalous run is visually tied to its defective metric. Because detection operates per metric/per group rather than on a combined QC score, generate one heatmap view per peak-metric × sample-group combination. Validation: cross-check that every run labeled anomalous in the CSV table appears with a corresponding deviation in the heatmap, and that the heatmap covers all peak-metric × sample-group combinations.

## Related tools

- **PeakQC** (Performs the per-metric/per-group Isolation Forest outlier detection and generates the error heatmaps (JPG/PDF) and outlier CSV reports) — https://github.com/pnnl/IonToolPack

## Evaluation signals

- The output outlier table covers every peak-metric × sample-group combination
- Every run flagged as anomalous in the CSV outlier table is consistent with the deviations rendered in the error heatmaps
- Heatmaps are produced in both JPG and PDF formats as part of the PeakQC report set
- Each heatmap entry pairs a run identifier with the specific defective metric (mass error, RT error, or arrival time error)
- Flagged runs are detected within sample groups, not across the whole batch

## Limitations

- No combined QC score is produced — anomalies are per metric and per sample group only
- Detection depends on the preceding extraction of peak metrics for auto-tracked or user-specified ions; ions not tracked yield no heatmap coverage
- Standalone installation instructions (pip/conda/docker) for PeakQC are not documented in the abstract/README, and the README provides no changelog

## Evidence
