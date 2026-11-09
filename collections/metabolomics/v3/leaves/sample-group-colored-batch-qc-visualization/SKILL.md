---
name: sample-group-colored-batch-qc-visualization
description: Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion;
  DDA or DIA) needs an omics-agnostic global QC overview, and you want a PCA view
  of MS1-derived time-vs-m/z images colored by sample group to spot batch outliers
  or group-separating trends.
when_to_use_negative:
- Input is already a processed feature table or identification-based QC metrics (e.g.,
  spectral match counts) rather than raw MS1 data — PeakQC's image-based PCA requires
  raw runs to build time-vs-m/z images
- The task is pinpointing the specific root cause of performance deterioration (chromatography,
  sensitivity, mass calibration); the PCA view flags compromised runs but identification-based
  metrics do little to pinpoint causes and detailed peak-metric extraction is needed
- Data are not mass spectrometry runs (e.g., NMR or sequencing data) — the skill is
  grounded in MS1 time-vs-m/z representations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakQC
  role: Performs the PCA on MS1-derived time-vs-m/z images and produces the group-colored
    batch QC figure via its QC evaluation module
  repo: https://github.com/pnnl/IonToolPack
- name: IonToolPack
  role: Installable-free GUI suite housing PeakQC; imports raw MS files (Thermo .raw,
    Agilent/Bruker .d, mzML) and runs processing
  repo: https://github.com/pnnl/IonToolPack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00146
    title: 'PeakQC: A Software Tool for Omics-Agnostic Automated Quality Control of
      Mass Spectrometry Data'
  source_content_hash: sha256:e07b7d066afe7b1917a0d9916676e91ff7530007fc01935f5850466920b39737
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pnnl/IonToolPack
  tools:
  - IonToolPack
  - PeakQC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sample-group-colored-batch-qc-visualization

## Summary

Reconstructs PeakQC's PCA-based batch quality assessment: each MS run in a batch is represented by a time-versus-m/z image generated from MS1 data, and samples are projected onto leading principal components and colored by sample group so compromised or outlier MS runs are quickly spotted.

## When to use

Use when a batch of raw MS runs (LC-MS, LC-IMS-MS, or direct infusion; DDA or DIA) needs an omics-agnostic global QC overview, and you want a PCA view of MS1-derived time-vs-m/z images colored by sample group to spot batch outliers or group-separating trends.

## When NOT to use

- Input is already a processed feature table or identification-based QC metrics (e.g., spectral match counts) rather than raw MS1 data — PeakQC's image-based PCA requires raw runs to build time-vs-m/z images
- The task is pinpointing the specific root cause of performance deterioration (chromatography, sensitivity, mass calibration); the PCA view flags compromised runs but identification-based metrics do little to pinpoint causes and detailed peak-metric extraction is needed
- Data are not mass spectrometry runs (e.g., NMR or sequencing data) — the skill is grounded in MS1 time-vs-m/z representations

## Inputs

- Raw MS files for a batch of runs (Agilent '.d', Thermo '.raw', Bruker '.d', or mzML) from LC-MS, LC-IMS-MS, or direct infusion acquisitions, with or without fragmentation (DDA/DIA)
- MS1 data per run (used to build the time-versus-m/z image representation)
- Optionally, user-specified sample groups for coloring the PCA plot

## Outputs

- PCA scatter plot of the batch with samples/runs projected on leading principal components, colored by sample group (exported as PDF)
- Time-versus-m/z image per MS run derived from MS1 data
- Companion CSV reports and JPG/PDF diagnostic figures (e.g., Isolation Forest error heatmaps per sample group) for downstream QC inspection

## How to apply

First generate a time-versus-m/z image per MS run from the MS1 data (PeakQC's prior processing step), which omics-agnostically represents each sample without requiring spectral identifications. Run principal component analysis on the batch of image representations using PeakQC's QC evaluation module, and project each sample/run onto the leading principal components. Color the PCA points by automatically assigned or user-specified sample groups so that batch outliers or trends separating biological groups become visible. Export the initial PCA figure as a PDF for later QC inspection, alongside the JPG/PDF diagnostic figures and CSV reports PeakQC generates (e.g., error heatmaps from the Isolation Forest outlier detection on peak metrics such as mass, retention time, and arrival time errors). The rationale is that identification-based QC tools cannot pinpoint causes of performance deterioration, whereas the image-based PCA gives a rapid, unbiased view of which runs deviate from the batch before wasting instrument time or precious samples.

## Related tools

- **PeakQC** (Performs the PCA on MS1-derived time-vs-m/z images and produces the group-colored batch QC figure via its QC evaluation module) — https://github.com/pnnl/IonToolPack
- **IonToolPack** (Installable-free GUI suite housing PeakQC; imports raw MS files (Thermo .raw, Agilent/Bruker .d, mzML) and runs processing) — https://github.com/pnnl/IonToolPack

## Examples

```
IonToolPack.exe  # import raw MS files (Thermo .raw / Agilent .d / Bruker .d / mzML) for the batch, select PeakQC, and click 'Process'
```

## Evaluation signals

- A PCA figure is produced where each point corresponds to one MS run in the batch and points are colored by (automatically assigned or user-specified) sample groups
- The PCA input representation is verifiably a time-versus-m/z image generated from MS1 data for every run in the batch
- The initial PCA figure is exported as a PDF, and companion JPG/PDF diagnostic figures plus CSV reports are generated for the batch
- Compromised/defective runs appear as outliers separated from their group in the leading principal components (compare against the paper's Figure 2 defective-run examples)
- Group-separating trends or batch outliers are visually discernible in the PCA view, consistent with the paper's stated purpose of quickly spotting compromised MS runs

## Limitations

- The PCA view flags outliers but, like other approaches, does little to pinpoint the specific cause of performance deterioration (chromatography, sensitivity, mass calibration); a decrease in spectral matches could stem from many reasons
- MS2-related diagnostic figures for user-specified ions require their expected fragment ions and intensities to be included in the input; mobiligrams require arrival time in the input and IMS separation in the data
- Large-scale cross-batch comparisons over months/years via unattended QC evaluations are a planned command-line capability, not necessarily available in the GUI workflow

## Evidence
