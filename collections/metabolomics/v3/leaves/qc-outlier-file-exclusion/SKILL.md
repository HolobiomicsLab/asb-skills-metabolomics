---
name: qc-outlier-file-exclusion
description: 'Use when post-processing PeakMeister outputs: review per-run peak annotation
  fidelity and internal standard signal-to-noise to decide whether any mzML data file
  shows systematic misannotation or failed QC and should be excluded before reference
  range or summary statistics are computed.'
when_to_use_negative:
- When misannotation affects most runs in a batch (e.g. wrong migration time prediction
  across all files) — fix the PeakMeister parameters instead of dropping files
- When a single analyte fails QC in many files — treat as a metabolite-level issue
  (e.g. low abundance near method detection limit) rather than excluding whole sample
  files
- Before PeakMeister has been run at all — this skill operates on already-generated
  PeakMeister outputs, not raw mzML
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: Source of the peak annotation, interference, and peak area tables plus EIE
    plots that are reviewed to detect misannotated/outlier data files
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Environment used to load PeakMeister output tables, flag outlier files via
    annotation and internal standard QC metrics, and subset the remaining data for
    downstream analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:25b098fee6e6a9c0513bcfe6e86cdb3efb67c43c26c06a9bf26db6cc209e35db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# QC Outlier File Exclusion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify and exclude whole data files (e.g. a single MSI-CE-MS sample file whose 73 peaks were misannotated due to a low ornithine-15N internal standard signal) from the PeakMeister output before computing downstream statistics, using peak annotation/interference records and extracted ion electropherogram plots to justify each exclusion.

## When to use

Use when post-processing PeakMeister outputs: review per-run peak annotation fidelity and internal standard signal-to-noise to decide whether any mzML data file shows systematic misannotation or failed QC and should be excluded before reference range or summary statistics are computed.

## When NOT to use

- When misannotation affects most runs in a batch (e.g. wrong migration time prediction across all files) — fix the PeakMeister parameters instead of dropping files
- When a single analyte fails QC in many files — treat as a metabolite-level issue (e.g. low abundance near method detection limit) rather than excluding whole sample files
- Before PeakMeister has been run at all — this skill operates on already-generated PeakMeister outputs, not raw mzML

## Inputs

- PeakMeister output tables (peak areas, migration times/indices, annotation and interference records) generated from mzML files
- Saved PeakMeister processing parameters (Mass List and Parameters.xlsx)
- Extracted ion electropherogram (EIE) plots per analyte per run
- Internal standard signal records, including ornithine-15N signal-to-noise per file

## Outputs

- Curated sample peak area table with identified outlier files removed (e.g. the file with 73 misannotated peaks excluded)
- Exclusion log mapping each removed file to the QC signal that triggered it
- Cleaned input set for downstream metabolite statistics (e.g. 40-metabolite reference ranges for ~5,000 ENANI-2019 participants)

## How to apply

Load the PeakMeister outputs (peak area table, migration index table, saved parameters, and EIE plots) and inspect each sample file's annotation record rather than relying on batch-level QC alone. Identify outlier files where many analyte peaks fall below the snr.threshold or are recorded as '<LOD', where interference.comigration.threshold flags show widespread overlap, or where the ornithine-15N internal standard signal is abnormally low — the article's example excluded one file in which 73 peaks were misannotated for exactly this reason. Cross-check borderline decisions against the extracted ion electropherogram plots to confirm the misannotation is systematic (a single run) rather than a parameter-tuning issue affecting the whole batch. Excluding a file is a data-level decision, not a parameter change: keep the saved PeakMeister parameters (e.g. ~30-35 ppm extraction.window.ppm, migration.window.seconds, snr.threshold) unchanged so the remaining files retain the validated >99.9% median annotation accuracy and 16.0% median CV precision. Document the excluded file and the trigger signal (e.g. low ornithine-15N standard, misannotated peak count) so the downstream 40-metabolite reference range computation can be reproduced.

## Related tools

- **PeakMeister** (Source of the peak annotation, interference, and peak area tables plus EIE plots that are reviewed to detect misannotated/outlier data files) — github.com/PBM-Group/PeakMeister
- **R** (Environment used to load PeakMeister output tables, flag outlier files via annotation and internal standard QC metrics, and subset the remaining data for downstream analysis)

## Examples

```
Rscript -e 'pm <- read.csv("peak_areas.csv"); flagged <- names(which(colSums(pm=="<LOD")>50)); kept <- pm[!names(pm) %in% flagged]; write.csv(kept,"peak_areas_clean.csv",row.names=FALSE)'
```

## Evaluation signals

- Number of analyte peaks recorded as '<LOD' (below snr.threshold) per file is low for retained runs and markedly elevated for the excluded file (e.g. 73 misannotated peaks driven by a low ornithine-15N standard)
- Retained files reproduce the article's peak annotation fidelity: median accuracy >99.9%
- Retained files reproduce the article's intermediate precision: median CV = 16.0%
- Mean peak-integration bias of the retained set matches the reported mean bias = -2.1% (or -1.3% against NIST SRM-1950 quantification)
- Downstream output on the curated set (e.g. 40-metabolite reference ranges in Table S2 from ~5,000 ENANI-2019 participants) is reproducible when the same files are excluded

## Limitations

- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard are prone to peak misannotation and may cause false outlier flags
- Low abundance metabolites with signals near the method detection limit can be misannotated, conflating instrument failure with genuine low signal
- Excluding a file loses its contribution to downstream statistics entirely; the rationale should rest on annotation/QC evidence, not arbitrary thresholds

## Evidence

- [results] misannotated 73 peaks due to a low ornithine-15N internal standard signal: [evidence span withheld — anchor status: not_found]
