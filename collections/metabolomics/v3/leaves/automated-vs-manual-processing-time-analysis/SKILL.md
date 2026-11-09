---
name: automated-vs-manual-processing-time-analysis
description: Use when a study claims an automated data-processing tool accelerates
  preprocessing of a large dataset and the actual wall-clock times and speedup ratio
  versus manual expert processing must be measured or reproduced.
when_to_use_negative:
- The goal is to assess preprocessing quality (peak annotation fidelity, precision,
  bias) rather than processing-time savings — use validation metrics like median accuracy
  >99.9% or median CV instead
- The sample set is small enough that manual integration is tractable and a timing
  benchmark is not meaningful
- The automated tool cannot ingest the data format (e.g., data not convertible to
  mzML), so no comparable automated run is possible
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakMeister
  role: Automated preprocessing pipeline whose execution time is benchmarked against
    manual integration of MSI-CE-MS data
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which the PeakMeister preprocessing script ('code')
    is executed and timed
  repo: https://github.com/PBM-Group/PeakMeister
- name: mzML
  role: Open-source file format to which all acquisition files must be converted before
    automated preprocessing can be timed
- name: MSI-CE-MS
  role: High-throughput separation platform whose multiplexed data are the subject
    of the timing comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:d6605bab26f174aa90918b5ee483c627d2cd6a984140c1d1cc847912ddaa479d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  - mzML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# automated-vs-manual-processing-time-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for quantifying wall-clock preprocessing-time savings of an automated pipeline (PeakMeister on MSI-CE-MS mzML data) versus manual vendor-software integration by an experienced analyst, expressed as a speedup ratio (>30-fold: ~4 days vs ~125 days).

## When to use

Use when a study claims an automated data-processing tool accelerates preprocessing of a large dataset and the actual wall-clock times and speedup ratio versus manual expert processing must be measured or reproduced.

## When NOT to use

- The goal is to assess preprocessing quality (peak annotation fidelity, precision, bias) rather than processing-time savings — use validation metrics like median accuracy >99.9% or median CV instead
- The sample set is small enough that manual integration is tractable and a timing benchmark is not meaningful
- The automated tool cannot ingest the data format (e.g., data not convertible to mzML), so no comparable automated run is possible

## Inputs

- All MSI-CE-MS acquisition files converted to open-source mzML format (serum/plasma samples, pooled QC/reference samples, calibrants, blanks)
- Completed 'Mass List and Parameters.xlsx' template with targeted mass list and preprocessing parameters (extraction.window.ppm, migration.window.seconds, snr.threshold, minimum.peak.width.seconds)
- Experienced-analyst manual integration time for the identical sample set (e.g., ~125 days) or an integration rate from which it can be estimated

## Outputs

- Recorded wall-clock time for automated preprocessing of the complete sample set (e.g., ~4 days on a desktop Core i7-9700)
- Timing comparison table with absolute times for automated and manual preprocessing
- Computed speedup ratio (manual time / automated time), expected to exceed 30-fold

## How to apply

Convert all acquisition files of the full sample set (e.g., 5,000 ENANI-2019 serum samples, 420 pooled QC samples, NIST SRM-1950 plasma, calibrants, and blanks) to mzML and place them in the 'mzML Files' folder required by the automated tool. Configure the tool's parameter template (e.g., 'Mass List and Parameters.xlsx' with extraction.window.ppm ~30-35, migration.window.seconds, snr.threshold, minimum.peak.width.seconds), then execute the pipeline on a defined reference machine (the study used a desktop Core i7-9700) and record the wall-clock time for the complete set. Estimate the manual baseline from an experienced analyst's vendor-software integration rate (the study reported ~125 days for the identical set). Compute speedup as manual time divided by automated time; the study's validation criterion was automated preprocessing within ~4 days versus ~125 days manual, i.e., a >30-fold reduction.

## Related tools

- **PeakMeister** (Automated preprocessing pipeline whose execution time is benchmarked against manual integration of MSI-CE-MS data) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which the PeakMeister preprocessing script ('code') is executed and timed) — https://github.com/PBM-Group/PeakMeister
- **mzML** (Open-source file format to which all acquisition files must be converted before automated preprocessing can be timed)
- **MSI-CE-MS** (High-throughput separation platform whose multiplexed data are the subject of the timing comparison)

## Examples

```
Rscript code  # open the PeakMeister project file and execute the 'code' R script on the mzML Files folder to time automated preprocessing
```

## Evaluation signals

- Automated preprocessing of the full sample set (serum/plasma + QC + calibrants + blanks) completes within approximately 4 days
- Estimated manual preprocessing time for the identical sample set is approximately 125 days
- Computed speedup ratio (manual/automated) is consistent with the reported >30-fold acceleration
- The automated run processes the complete cohort, matching the study scope (5,000 serum samples and 420 QC samples from ENANI-2019 plus NIST SRM-1950 plasma samples)
- Timing is recorded on a defined reference machine (desktop Core i7-9700) so the comparison is reproducible

## Limitations

- The manual baseline is an estimate based on an experienced analyst's integration rate, not a directly measured repetition
- The measured speedup is machine- and dataset-specific (Core i7-9700 desktop; ENANI-2019 sample set) and may not transfer to other hardware or data
- PeakMeister's timing advantage applies to targeted full-scan MSI-CE-MS data; it does not use dynamic time warping, so unrelated workflows may behave differently

## Evidence

- [discussion] we demonstrate that PeakMeister, an open-access software tool available on GitHub, can significantly reduce data preprocessing times in MSI-CE-MS by over 30-fold: "we demonstrate that PeakMeister, an open-access software tool available on GitHub, can significantly reduce data preprocessing times in MSI-CE-MS by over 30-fold"
- [abstract] PeakMeister accelerated data preprocessing times by 30-fold compared to manual processing of MSI-CE-MS data: "PeakMeister accelerated data preprocessing times by 30-fold compared to manual processing of MSI-CE-MS data"
- [readme] Convert all your data files to open-source mzML files and save them in the folder titled "mzML Files": "Convert all your data files to open-source mzML files and save them in the folder titled "mzML Files""
- [discussion] all data preprocessing using PeakMeister was directly performed on mzML files: "all data preprocessing using PeakMeister was directly performed on mzML files"
