---
name: peaklist-gap-filling
description: Use when a peak list has been deisotoped and aligned across multiple
  raw LC-MS data files, but individual samples are missing detections for peaks found
  in other samples, blocking normalization and export.
when_to_use_negative:
- The input is already a normalized feature table — gap filling operates on aligned,
  non-normalized peak lists and must precede the normalization module
- Only a single raw data file is being processed with no cross-sample alignment, since
  there is no aligned peak list whose missing features can be recovered
- The data are not LC-MS — the paper's modules were tested mainly on LC-MS data, with
  GC-MS and MASCOT interoperability described only as possible future expansion
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Hosts the gap-filling module that recovers missing features from raw data
    after peak list alignment; its modular design lets gap filling run between alignment
    and normalization
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Implementation platform for MZmine 2's gap-filling module, making the workflow
    platform independent
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:8572db9b474177856e58df508e6453b06e1e24256bb1f911c3bc985d363e2e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - JFreeChart
  - Java
  - MZmine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Peak list gap filling

## Summary

Fills missing values in an aligned MZmine 2 peak list by detecting peaks that are expected but absent in individual samples, so that every LC-MS sample carries a complete feature table before normalization and export.

## When to use

Use when a peak list has been deisotoped and aligned across multiple raw LC-MS data files, but individual samples are missing detections for peaks found in other samples, blocking normalization and export.

## When NOT to use

- The input is already a normalized feature table — gap filling operates on aligned, non-normalized peak lists and must precede the normalization module
- Only a single raw data file is being processed with no cross-sample alignment, since there is no aligned peak list whose missing features can be recovered
- The data are not LC-MS — the paper's modules were tested mainly on LC-MS data, with GC-MS and MASCOT interoperability described only as possible future expansion

## Inputs

- Aligned peak list produced by the RANSAC-based (or Join) peak list aligner across multiple samples
- Imported raw LC-MS data files held in the MZmine 2 project at full mass precision
- Gap-filling module parameters (m/z and retention-time detection tolerances) set in the modular parameter dialog

## Outputs

- Gap-filled peak list in which each aligned row has a detected feature per sample where possible
- Downstream normalized_peak_list.csv after the normalization and export modules are applied to the gap-filled list

## How to apply

Run gap filling as stage 6 of the MZmine 2 workflow, immediately after the deisotoped peak lists are aligned across samples and before normalization. The module revisits the imported raw data — which were loaded with full mass precision, 'without any degradation due to inadequate resampling' — and searches each sample where the aligned peak list reports a missing feature, looking for the chromatographic peak at the m/z and retention-time position defined by the aligned row. Because MZmine 2 is modular and 'never removes the original unprocessed data', keep each intermediate peak list (pre-alignment, aligned, gap-filled) as a separate file so the gap-filling result can be recomputed or audited. After gap filling, apply the normalization module to the gap-filled peak list and export the final table as normalized_peak_list.csv; judge success by comparing the missing-value count and row count before and after filling.

## Related tools

- **MZmine 2** (Hosts the gap-filling module that recovers missing features from raw data after peak list alignment; its modular design lets gap filling run between alignment and normalization) — https://github.com/mzmine/mzmine2
- **Java** (Implementation platform for MZmine 2's gap-filling module, making the workflow platform independent)

## Evaluation signals

- Row count of the gap-filled peak list equals the row count of the aligned peak list — gap filling recovers values within existing aligned rows rather than adding new ones
- The number of missing (null) feature intensities per sample decreases relative to the aligned peak list without changing already-detected values
- Recovered peaks fall within the module's m/z and retention-time tolerance of the aligned row's position, and import/peak-detection precision was preserved 'without any degradation due to inadequate resampling'
- Normalization applied after gap filling exports a complete normalized_peak_list.csv with no residual missing entries from expected features
- Intermediate peak lists (deisotoped, aligned, gap-filled) still exist as separate files, since MZmine 2 never removes original unprocessed data

## Limitations

- The currently developed modules, including gap filling, were tested mainly on LC-MS data; validation on GC-MS or proteomics search-engine interoperation (e.g. MASCOT) is not demonstrated
- Gap filling depends on the quality of the preceding alignment — on the P2 dataset fraction 00, over 6800 rows after alignment made some alignment comparisons unfavorable (RANSAC worse than Join aligner in that single case)
- The paper's 'current version' is referenced without a version identifier and no changelog is available, so the exact behavior of the gap-filling module may vary across releases

## Evidence

- [results] Workflow position of gap filling: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
- [methods] Full-precision raw data underpin gap filling: "We designed the data import and peak detection modules to maintain the precision of the imported data without any degradation due to inadequate resampling."
- [readme] Modularity and tool context: "MZmine 2 is an open-source software for mass-spectrometry data processing. The goals of the project is to provide a user-friendly, flexible and easily extendable software"
