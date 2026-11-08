---
name: sample-group-outlier-detection
description: Use when a batch of raw MS runs (Thermo .raw, Agilent 'd', Bruker 'd',
  mzML) has been processed to extract peak metrics (mass error, retention time error,
  arrival time error) for auto-tracked and user-specified ions and you need to flag
  which individual runs are anomalous per sample group.
when_to_use_negative:
- When you need a single combined QC score for the whole batch rather than per-metric/per-group
  anomaly flags — PeakQC deliberately avoids producing one global QC metric
- When the input is already a feature table or QC summary without raw-peak-metric
  reports (mass/RT/AT errors), since the Isolation Forest models operate on those
  extracted metrics
- When no sample-group structure exists or a single-run dataset is provided, because
  the algorithm fits models within each sample group
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: PeakQC
  role: Fits Isolation Forest models per peak metric and sample group to flag anomalous
    MS runs, then renders error heatmaps
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:2807066f9af87ad57dae860e304e2a2366f76565197025831efc1233c250fa64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - PeakQC
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sample-group-outlier-detection

## Summary

Detect anomalous MS runs in a batch by fitting an Isolation Forest model separately for each precursor-ion peak metric within each sample group, then compiling flagged runs into a structured outlier table and error heatmaps. This is PeakQC's machine-learning-based quality-control layer for pinpointing specific defective metrics rather than issuing a single combined QC score.

## When to use

Use when a batch of raw MS runs (Thermo .raw, Agilent 'd', Bruker 'd', mzML) has been processed to extract peak metrics (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions and you need to flag which individual runs are anomalous per sample group.

## When NOT to use

- When you need a single combined QC score for the whole batch rather than per-metric/per-group anomaly flags — PeakQC deliberately avoids producing one global QC metric
- When the input is already a feature table or QC summary without raw-peak-metric reports (mass/RT/AT errors), since the Isolation Forest models operate on those extracted metrics
- When no sample-group structure exists or a single-run dataset is provided, because the algorithm fits models within each sample group

## Inputs

- Raw MS files supported by IonToolPack: Agilent 'd', Thermo '.raw', Bruker 'd', mzML (LC-MS, LC-IMS-MS, direct infusion, DDA/DIA with or without fragmentation)
- CSV reports of extracted peak metrics (mass error, retention time error, arrival time error) for auto-tracked and user-specified ions
- Sample-group assignments for each MS run or sample in the batch

## Outputs

- CSV outlier table listing flagged anomalous MS runs with metric, sample group, and run identifiers
- Error heatmaps (JPG/PDF) displaying detected outlier MS runs together with the corresponding defective metrics
- Per peak-metric × sample-group combination coverage of anomaly labels

## How to apply

Partition the batch by sample group so that anomaly detection is performed within each group, then load the CSV reports of extracted peak metrics — mass error, retention time error, and arrival time error — computed for both auto-tracked ions (the topmost abundant MS1 ions found per sample group) and user-specified ions with theoretical or reference values. For each peak metric × sample group combination, fit an Isolation Forest model, exploiting the algorithm's binary-tree splitting, linear time complexity, and low memory requirement. Score every MS run per metric/group and label runs whose metric value is anomalous relative to the group. Compile all flagged runs into a structured outlier table (CSV) with metric, sample group, and run identifiers, and generate error heatmaps (JPG/PDF) visualizing the per-metric deviations of the flagged runs. The decision point is the per-metric/per-group granularity: do not merge scores into a combined QC metric, since PeakQC's design flags specific defective metrics to pinpoint causes like chromatography, sensitivity, or mass calibration issues.

## Related tools

- **PeakQC** (Fits Isolation Forest models per peak metric and sample group to flag anomalous MS runs, then renders error heatmaps) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe --tool PeakQC --input raw/*.raw --sample-groups groups.csv --outliers-out outlier_runs.csv
```

## Evaluation signals

- The outlier table (CSV) must cover every peak-metric × sample-group combination
- Flagged anomalous runs must be consistent with the deviations rendered in the generated error heatmaps (JPG/PDF)
- Each flagged run should be traceable to a specific defective metric (mass error, RT error, or AT error) rather than a combined score
- Results should be reproducible per sample group: re-running on the same batch and group assignments yields the same flagged run set
- Heatmap deviations should align with the metric values that triggered anomaly labeling for the corresponding runs

## Limitations

- Outlier detection is performed separately for each sample group; cross-group comparisons are not part of this skill
- Because detection is per-metric, a run can be flagged for one metric and not others — no aggregate anomaly score is emitted
- Quality of the anomaly detection depends on the upstream extraction of peak metrics for auto-tracked and user-specified ions; errors in that step propagate
- Isolation Forest was chosen for linear time complexity and low memory, but its per-group fitting assumes sufficient within-group runs to characterize normal behavior

## Evidence

- [results] PeakQC performs outlier detection on individual peak metrics separately for each sample group using the Isolation Forest algorithm: [evidence span withheld — anchor status: not_found]
- [results] Isolation Forest rationale: [evidence span withheld — anchor status: not_found]
