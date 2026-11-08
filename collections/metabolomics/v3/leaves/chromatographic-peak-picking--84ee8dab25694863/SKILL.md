---
name: chromatographic-peak-picking
description: Use when starting from raw MS data files (e.g. LC-MS runs) and a feature-level
  peak list is needed as the basis for deisotoping, alignment, and gap filling.
when_to_use_negative:
- Input is already a deisotoped, aligned, or gap-filled peak list — peak picking should
  precede those steps, not be re-run on derived tables
- The goal is alignment quality tuning (use the RANSAC or Join aligner instead)
- 'Data type is outside the validated scope: the developed modules were tested mainly
  on LC-MS data, so GC-MS or proteomics search-engine workflows are not demonstrated'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3384
tools:
- name: MZmine 2
  role: Hosts the peak detection module with alternative plug-in detectors (Centroid,
    Exact mass, Local maxima) and Savitzky-Golay deconvolution
  repo: https://github.com/mzmine/mzmine2
- name: JFreeChart
  role: Backs the embedded visualization modules that provide online previews of picked
    peaks during parameter setup
- name: Java
  role: Implementation language of the platform-independent peak picking modules;
    build from source with JDK 12 or newer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:dba615ce8d0005120d2a28c3bbfad6aef5ad2edf8e5fe65d300cd298fd6e94ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - JFreeChart
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chromatographic-peak-picking

## Summary

Detects chromatographic peaks in raw LC-MS data using MZmine 2's modular peak detection module, choosing among alternative plug-in detectors (Centroid, Exact mass, Local maxima) and optionally deconvolving overlapping peaks with the Savitzky-Golay smoothed second-derivative method.

## When to use

Use when starting from raw MS data files (e.g. LC-MS runs) and a feature-level peak list is needed as the basis for deisotoping, alignment, and gap filling.

## When NOT to use

- Input is already a deisotoped, aligned, or gap-filled peak list — peak picking should precede those steps, not be re-run on derived tables
- The goal is alignment quality tuning (use the RANSAC or Join aligner instead)
- Data type is outside the validated scope: the developed modules were tested mainly on LC-MS data, so GC-MS or proteomics search-engine workflows are not demonstrated

## Inputs

- Raw MS data files (e.g. LC-MS runs imported via the MZmine 2 data import module)
- Optionally smoothed/filtered raw data (Savitzky-Golay or wavelet-based plug-in output)
- Peak detector parameter set (detector type: Centroid, Exact mass, or Local maxima; baseline cut-off)

## Outputs

- Peak list of detected chromatographic peaks (preserving full mass precision, no resampling degradation)
- Intermediate peak list file kept separate, since MZmine 2 never removes the original unprocessed data

## How to apply

Import raw MS data with the MZmine 2 data import module first: import and peak detection are designed to maintain full mass precision without degradation from inadequate resampling. Optionally apply the filtering/smoothing plug-in (Savitzky-Golay or wavelet-based) to the raw data before picking. Choose a peak detector plug-in — Centroid, Exact mass, or Local maxima — and set the baseline cut-off; for chromatographic deconvolution of overlapping peaks, the Savitzky-Golay method uses the smoothed second derivative of the chromatogram curve to detect the borders of individual peaks. Use the embedded visualization previews (linked via JFreeChart) during parameter setup to verify that detector parameters produce correct peak boundaries before batch-processing all files.

## Related tools

- **MZmine 2** (Hosts the peak detection module with alternative plug-in detectors (Centroid, Exact mass, Local maxima) and Savitzky-Golay deconvolution) — https://github.com/mzmine/mzmine2
- **JFreeChart** (Backs the embedded visualization modules that provide online previews of picked peaks during parameter setup)
- **Java** (Implementation language of the platform-independent peak picking modules; build from source with JDK 12 or newer)

## Examples

```
/opt/mzmine/bin/mzmine -batch peakpicking_batch.xml
```

## Evaluation signals

- Peak boundaries in the preview visualizations match the chromatogram inflection points (Savitzky-Golay second-derivative borders fall on true peak edges)
- Mass precision of detected peaks matches the imported raw data (no degradation due to resampling)
- Baseline cut-off removes noise peaks without discarding low-abundance true peaks (check against the shoulder/noise model plug-in behaviour on FTMS data)
- Downstream steps succeed: the peak list can be deisotoped with CDK isotope grouping and aligned (RANSAC or Join aligner) without cascading errors
- Original unprocessed raw data files remain untouched and each intermediate peak list is stored as a separate file

## Limitations

- Modules were tested mainly on LC-MS data; validation on other data types (e.g. gas chromatography-MS) is described only as possible future expansion, not demonstrated
- Alternative plug-in algorithms require per-dataset parameter tuning; the critical parameter setup relies on manual inspection of the online previews
- Batch-mode operation depends on the build/install working correctly (e.g. JDK 12 or newer for mzmine2; the mzmine2 repo is now archived as reference only, with development moved to the new mzmine repository)

## Evidence

- [results] Peak detection is decomposed into customizable steps with alternative plug-in algorithms; among the chromatographic peak deconvolution methods, the Savitzky-Golay algorithm uses the smoothed second derivative of the chromatogram curve to detect the borders of individual peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Data import and peak detection maintain the precision of the imported data without resampling degradation.: "We designed the data import and peak detection modules to maintain the precision of the imported data without any degradation due to inadequate resampling."
- [methods] Peak picking is linked to embedded visualization modules with online previews during parameter setup.: "critical data processing methods such as peak picking were linked to embedded visualization modules, providing online previews during parameter setup."
- [results] The results-section workflow orders the stages: raw data file import, optional filtering/smoothing, peak picking, deisotoping, alignment, gap filling, normalization.: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
- [readme] The mzmine2 README confirms MZmine 2 is an open-source MS data processing tool and that building requires JDK 12 or newer.: "MZmine development requires Java Development Kit (JDK) version 12 or newer"
