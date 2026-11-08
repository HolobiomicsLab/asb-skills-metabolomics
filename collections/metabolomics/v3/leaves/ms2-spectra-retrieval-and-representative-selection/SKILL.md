---
name: ms2-spectra-retrieval-and-representative-selection
description: 'Use when you have raw HRMS data (mzML) plus a compound or mass list
  and need first-look prescreening: retrieve EICs and MS2 scans within an m/z tolerance,
  QC each MS1 parent, and pick a representative MS2 spectrum.'
when_to_use_negative:
- The input is already a peak/feature table rather than raw mzML data requiring EIC
  extraction and MS2 scan retrieval
- A full non-target feature-detection workflow (e.g. xcms, RforMassSpectrometry, patRoon)
  is needed instead of list-based prescreening of preselected compounds or masses
- The analyst has no programming expertise and would be better served by Shinyscreen's
  self-documented web GUI or Docker image rather than direct low-level MSnbase/mzR
  scripting
edam_operation: http://edamontology.org/operation_3432
tools:
- name: MSnbase
  role: Low-level R package used to read the mzML file and retrieve precursor ion
    masses, EICs within the specified m/z tolerance, and corresponding MS2 spectral
    scans
- name: mzR
  role: Low-level R alternative for flexible MS data access during first-look exploration
    and prescreening
- name: Shinyscreen
  role: Wraps this extraction/prescreening/QC logic behind an R package, command-line
    tool, Shiny GUI, or network-deployable service, including the quality assessment
    and representative MS2 selection steps
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: R
  role: Implementation language in which the extraction, quality assessment, and selection
    steps are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:b05f68af3e4c3c7cda52876082b32d75316f79aa7f3d8077ddfb7bd77dd590e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - mzR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms2-spectra-retrieval-and-representative-selection

## Summary

Extract precursor-ion EICs and corresponding MS2 spectral scans from a high-resolution MS mzML file using MSnbase/mzR, run a per-MS1-parent quality assessment (MS1/MS2 existence, noise level, retention time shift), and select a representative MS2 spectrum for each listed compound or mass.

## When to use

Use when you have raw HRMS data (mzML) plus a compound or mass list and need first-look prescreening: retrieve EICs and MS2 scans within an m/z tolerance, QC each MS1 parent, and pick a representative MS2 spectrum.

## When NOT to use

- The input is already a peak/feature table rather than raw mzML data requiring EIC extraction and MS2 scan retrieval
- A full non-target feature-detection workflow (e.g. xcms, RforMassSpectrometry, patRoon) is needed instead of list-based prescreening of preselected compounds or masses
- The analyst has no programming expertise and would be better served by Shinyscreen's self-documented web GUI or Docker image rather than direct low-level MSnbase/mzR scripting

## Inputs

- High-resolution mass spectrometry raw file in mzML format
- Validated compound list CSV (compound structures/names or a mass list) with adduct setting
- QC/extraction parameters: experimental m/z error, retention time error, MS1 and MS2 intensity thresholds, retention time shift threshold

## Outputs

- CSV summary table with per-compound m/z, retention time, quality result, MS1/MS2 existence flags, noise and retention-time-shift QC results, and the selected representative MS2 spectrum
- Saved extracted MS1/MS2 data object (EICs within the m/z tolerance and associated MS2 scans)
- Optional ggplot2/cowplot-based plots of MS1/MS2 chromatograms and MS2 spectra for visual inspection

## How to apply

Load the validated compound list (CSV) and the mzML file, and set the experimental m/z and retention time errors plus QC thresholds (MS1 and MS2 intensity thresholds, retention time shift). Use MSnbase to read the mzML file and, for each precursor ion mass from the list (computing precursor m/z from the given adduct setting when only structures are supplied), retrieve the extracted ion chromatograms within the specified m/z tolerance and the corresponding MS2 spectral scans. Subject every MS1 parent data point to the quality assessment stage, checking existence of MS1 and MS2 data, noise level, and retention time shift against the supplied thresholds; filter based on m/z, retention time, or quality score as needed. If MS2 data exist, select a representative MS2 spectrum for each input mass. Write the prescreening results (quality flags, selected representative MS2, retention times, intensities) to a CSV summary table and save the extracted MS1/MS2 data object, flagging entries that fail thresholds rather than silently dropping them.

## Related tools

- **MSnbase** (Low-level R package used to read the mzML file and retrieve precursor ion masses, EICs within the specified m/z tolerance, and corresponding MS2 spectral scans)
- **mzR** (Low-level R alternative for flexible MS data access during first-look exploration and prescreening)
- **Shinyscreen** (Wraps this extraction/prescreening/QC logic behind an R package, command-line tool, Shiny GUI, or network-deployable service, including the quality assessment and representative MS2 selection steps) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **R** (Implementation language in which the extraction, quality assessment, and selection steps are scripted)

## Examples

```
library(MSnbase); raw <- readMSData("sample.mzML", mode = "onDisk")
```

## Evaluation signals

- The summary CSV exists and contains the expected columns (m/z, retention time, quality result)
- Each compound entry has a recorded MS1/MS2 existence flag plus noise-level and retention-time-shift QC results
- Entries failing the MS1/MS2 intensity or retention-time-shift threshold checks are flagged, not silently dropped
- A representative MS2 spectrum is selected only for input masses where MS2 data exist ('if present')
- Retrieved EICs are confined to the specified m/z tolerance around each precursor ion mass

## Limitations

- Low-level packages such as mzR and MSnbase require some programming expertise to fully utilize, which is the gap Shinyscreen was designed to bridge
- The skill is scoped to data exploration, visualization and automated quality control with manual overrides on a preselected list of compounds or masses — not comprehensive untargeted processing
- License is Apache 2.0 but no archived code snapshot (Zenodo/OSF) is reported, so reproducibility depends on the live GitLab repository

## Evidence

- [abstract] Shinyscreen is an R package and Shiny-based web application designed for the exploration, visualization, and quality assessment of raw data from high resolution mass spectrometry instruments.: "Shinyscreen is an R package and Shiny-based web application designed for the exploration, visualization, and quality assessment of raw data from high resolution mass spectrometry instruments."
- [methods] EIC and MS2 scan retrieval: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the"
- [methods] Quality assessment criteria: "The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
- [methods] Representative MS2 selection: "This step is also used to select a representative MS2 spectra (if present) for the input masses"
- [intro] Low-level R package rationale: "the more modern MSnbase [10, 11], which offer great flexibility in analysis of MS data"
