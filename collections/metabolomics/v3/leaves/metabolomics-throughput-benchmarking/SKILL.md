---
name: metabolomics-throughput-benchmarking
description: Use when comparing automated preprocessing wall-clock time against a
  manual vendor-software baseline for a large targeted metabolomic data set (e.g.,
  thousands of MSI-CE-MS serum samples) to quantify speedup.
when_to_use_negative:
- The data set is small enough that manual vendor-software preprocessing is not the
  practical bottleneck being measured
- Input files are not in mzML format and conversion is out of scope, since the tool
  requires 'mzML Files' as its input folder
- The benchmark target is a non-targeted or untargeted workflow, since PeakMeister
  is designed for targeted full-scan MSI-CE-MS data with a provided mass list
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakMeister
  role: Automated preprocessing tool whose wall-clock runtime is benchmarked against
    manual integration of MSI-CE-MS data
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister script ('code') is executed
    for the timing benchmark
- name: mzML
  role: Open-source file format to which all acquisition files must be converted before
    the timed preprocessing run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:39eced46d831b73e0bdbb27f85db4ffe909d3103c0d651b4daa1c746528f73bb
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
  - mzML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolomics-throughput-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark wall-clock preprocessing throughput of an automated pipeline (e.g., PeakMeister for MSI-CE-MS) against a manual analyst baseline, computing an absolute-time and speedup-ratio comparison for large-scale metabolomic data sets.

## When to use

Use when comparing automated preprocessing wall-clock time against a manual vendor-software baseline for a large targeted metabolomic data set (e.g., thousands of MSI-CE-MS serum samples) to quantify speedup.

## When NOT to use

- The data set is small enough that manual vendor-software preprocessing is not the practical bottleneck being measured
- Input files are not in mzML format and conversion is out of scope, since the tool requires 'mzML Files' as its input folder
- The benchmark target is a non-targeted or untargeted workflow, since PeakMeister is designed for targeted full-scan MSI-CE-MS data with a provided mass list

## Inputs

- Complete set of MSI-CE-MS acquisition files converted to mzML format (e.g., 5,000 ENANI-2019 serum samples, 420 pooled QC samples, NIST SRM-1950 plasma samples, calibrants, and blanks)
- 'Mass List and Parameters.xlsx' template with targeted mass list and preprocessing parameters (extraction.window.ppm, migration.window.seconds, snr.threshold, minimum.peak.width.seconds)
- Manual preprocessing time baseline for the same sample set from an experienced vendor-software analyst
- Reference hardware specification (e.g., desktop Core i7-9700)

## Outputs

- Wall-clock automated preprocessing time for the full sample set
- Estimated manual preprocessing time for the identical sample set
- Speedup ratio (manual time / automated time)
- Timing comparison table with absolute times and ratio

## How to apply

Convert all acquisition files to open-source mzML format and place them in the 'mzML Files' folder required by the tool; complete the 'Mass List and Parameters.xlsx' template with the targeted mass list and preprocessing parameters (e.g., extraction.window.ppm ~30-35, migration.window.seconds, snr.threshold, minimum.peak.width.seconds). Execute the R script ('code') via the project file on the reference machine and record the wall-clock time for automated preprocessing of the complete sample set. Estimate the manual preprocessing time for the identical sample set using an experienced analyst's integration rate. Compute the speedup ratio (manual time divided by automated time) and compile a timing comparison table with both absolute times and the ratio. Accept the benchmark if the ratio reproduces the expected order of speedup (in the source study: ~4 days automated vs ~125 days manual on a desktop Core i7-9700, a >30-fold reduction).

## Related tools

- **PeakMeister** (Automated preprocessing tool whose wall-clock runtime is benchmarked against manual integration of MSI-CE-MS data) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister script ('code') is executed for the timing benchmark)
- **mzML** (Open-source file format to which all acquisition files must be converted before the timed preprocessing run)

## Examples

```
Convert files to mzML into 'mzML Files', fill 'Mass List and Parameters.xlsx', then open the project in R and run: Rscript code  # timed automated preprocessing of all ENANI-2019 + QC + calibrant + blank mzML files
```

## Evaluation signals

- Automated preprocessing of the complete sample set (serum/plasma, pooled QC, calibrant, and blank files) completes within the measured wall-clock window (~4 days on the reference desktop Core i7-9700)
- Estimated manual time (~125 days) is derived from the experienced-analyst integration rate applied to the identical sample set, not a smaller subset
- Computed speedup ratio (manual / automated) is consistent with the reported 30-fold acceleration (>30-fold)
- Timing comparison table reports both absolute times and the ratio for the same sample inventory (5,000 serum + 420 QC + NIST SRM-1950 + calibrants + blanks)
- Quality metrics from the automated output are acceptable (median peak annotation accuracy >99.9%, median CV = 16.0%), confirming the speedup was not achieved at the cost of fidelity

## Limitations

- Benchmark is specific to targeted full-scan MSI-CE-MS data; PeakMeister is not yet validated for matrices other than serum (e.g., urine, saliva, stool)
- No user-friendly graphical interface exists, so the benchmark requires running the R script via the project file
- Speedup assumes a fixed manual baseline from one experienced analyst; hardware differences change absolute times
- PeakMeister avoids computationally intensive dynamic time warping by using migration time indices, so the speedup depends on reliable internal-standard–flanked analytes; late-migrating metabolites lacking flanking or stable-isotope internal standards are prone to peak misannotation

## Evidence
