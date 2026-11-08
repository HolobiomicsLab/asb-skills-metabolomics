---
name: mass-spectrometry-qc-plot-generation
description: Use when peak metrics for auto-tracked MS1 ions or user-specified theoretical/reference
  ion targets have already been extracted and you need per-ion, per-metric CSV reports
  plus JPG/PDF figures to inspect and track instrument performance.
when_to_use_negative:
- Peak metrics have not yet been extracted — run PeakQC's metric-extraction/PCA QC
  step first; this skill only packages and plots already-extracted metrics.
- The analysis goal is unattended, automated QC comparison across batches over months
  and years (command-line functionality is planned, not the operator-facing report/figure
  workflow).
- Input is a feature list needing harmonization across processing software (that is
  IonToolPack's Comparador role, not report/figure generation).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakQC
  role: 'Primary tool: reporting module writes the CSV summary reports and plotting
    module renders JPG/PDF figures of ion metrics for auto-tracked and user-specified
    ions'
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Distributes PeakQC as a no-install GUI executable (IonToolPack.exe from GitHub
    Releases) that ingests Agilent/Thermo/Bruker/mzML raw MS files
  repo: https://github.com/pnnl/IonToolPack
- name: pandas
  role: Loads and merges the extracted peak-metrics table for auto-tracked and user-specified
    ions before report/figure generation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:0f7ee6885f76a87a230d55edb8ce3af7a5cd43b1976803c74a1f2767eb8e212f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# mass-spectrometry-qc-plot-generation

## Summary

Package extracted precursor-ion peak QC metrics (mass error, retention time error, arrival time error) into CSV summary reports and JPG/PDF figures for auto-tracked and user-specified ions, so an instrument operator can visually inspect and track MS run quality across a batch.

## When to use

Use when peak metrics for auto-tracked MS1 ions or user-specified theoretical/reference ion targets have already been extracted and you need per-ion, per-metric CSV reports plus JPG/PDF figures to inspect and track instrument performance.

## When NOT to use

- Peak metrics have not yet been extracted — run PeakQC's metric-extraction/PCA QC step first; this skill only packages and plots already-extracted metrics.
- The analysis goal is unattended, automated QC comparison across batches over months and years (command-line functionality is planned, not the operator-facing report/figure workflow).
- Input is a feature list needing harmonization across processing software (that is IonToolPack's Comparador role, not report/figure generation).

## Inputs

- Extracted precursor-ion peak metrics table (mass error, retention time error, arrival time error)
- List of auto-tracked ions (topmost abundant MS1 ions per sample group)
- User-specified ions with theoretical or reference m/z values
- Raw MS files readable by IonToolPack: Agilent '.d', Thermo '.raw', Bruker '.d', mzML; LC-MS, LC-IMS-MS, direct infusion; DDA or DIA with/without fragmentation

## Outputs

- Per-ion and per-metric CSV summary reports
- JPG figures visualizing each ion metric across the batch of MS runs
- PDF figures of the same ion metrics
- Diagnostic plots for fragmentation spectra (MS2) of user-specified ions
- Mobiligrams (ion mobility traces) for user-specified ions when AT is included and IMS separation exists
- Navigable output folders structured per ion and metric

## How to apply

Load the extracted peak-metrics table (errors in mass, retention time, and arrival time) with pandas and merge metrics for the auto-tracked ions (the topmost abundant MS1 ions most frequent per sample group) with metrics for user-specified ions given as theoretical or reference values. Use PeakQC's reporting module to write per-ion and per-metric CSV summary reports, and its plotting module to render JPG and PDF figures visualizing each ion metric across the batch of MS runs. Organize all CSV reports and JPG/PDF figures into navigable output folders structured per ion and metric, alongside diagnostic plots for fragmentation spectra and mobiligrams (ion mobility traces) extracted for user-specified ions when arrival time data and IMS separation are present.

## Related tools

- **PeakQC** (Primary tool: reporting module writes the CSV summary reports and plotting module renders JPG/PDF figures of ion metrics for auto-tracked and user-specified ions) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Distributes PeakQC as a no-install GUI executable (IonToolPack.exe from GitHub Releases) that ingests Agilent/Thermo/Bruker/mzML raw MS files) — https://github.com/pnnl/IonToolPack
- **pandas** (Loads and merges the extracted peak-metrics table for auto-tracked and user-specified ions before report/figure generation)

## Examples

```
double-click IonToolPack.exe -> import raw MS files (Agilent .d / Thermo .raw / mzML) -> click “Process” -> inspect per-ion CSV reports and JPG/PDF figures in the output folders
```

## Evaluation signals

- Every auto-tracked ion and every user-specified ion has both a CSV report and a rendered JPG/PDF figure in the per-ion, per-metric output folders.
- Figures correctly display the tracked metric values (mass error, RT error, AT error) per MS run — verified by qualitative visual QC check.
- CSV reports contain per-ion and per-metric summaries matching the merged metrics for auto-tracked and user-specified ions.
- MS2 diagnostic figures appear only for user-specified ions whose expected fragment ions and intensities were included, and mobiligrams appear only when AT data and IMS separation are present.
- Example input and output files match the patterns in the repository's test_data folder.

## Limitations

- Reports and figures are generated only for ions whose metrics were successfully extracted; ions missing metrics will lack corresponding outputs.
- MS2 diagnostic figures require the user to supply expected fragment ions and intensities for the targets.
- Mobiligram extraction requires arrival time to be included in the input and IMS separation to be present in the MS data.
- Large-scale, unattended batch-over-months QC comparisons are a planned command-line capability, not part of the current GUI reporting workflow.

## Evidence

- [results] 'mobiligrams' (i.e., ion mobility traces) are extracted for user-specified ions if the AT is included in the input and the MS data contains IMS separation: [evidence span withheld — anchor status: not_found]
