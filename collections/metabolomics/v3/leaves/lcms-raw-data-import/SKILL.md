---
name: lcms-raw-data-import
description: Use when raw vendor or open-format LC-MS files need to be loaded into
  MZmine 2 before peak picking, or when designing a processing pipeline that must
  import MS data without precision loss from resampling.
when_to_use_negative:
- Input is already a peak list or aligned feature table — peak detection, alignment,
  or gap-filling modules, not raw import, should be applied.
- Only GC-MS or proteomics search-engine interoperation is required — MZmine 2 modules
  were tested mainly on LC-MS data and GC-MS / MASCOT support is future expansion,
  not demonstrated.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Provides the data import module that loads raw MS data while preserving mass
    precision; all downstream modules operate on the imported data
  repo: http://mzmine.sourceforge.net/
- name: Java
  role: Implementation platform — MZmine 2 is developed in Java, making it platform
    independent for import across operating systems
- name: JFreeChart
  role: Embedded visualization library used by MZmine 2 visualizers for previewing
    imported and processed data during parameter setup
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:90ab4db9da7a3c2b0af9045331aa9fd479db2dbf1de182f59ee44247693e7ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JFreeChart
  - Java
  - MZmine 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lcms-raw-data-import

## Summary

Import raw LC-MS data files into MZmine 2 as the first stage of the modular data processing workflow, preserving full mass precision without resampling degradation so that downstream peak detection operates on unmodified data.

## When to use

Use when raw vendor or open-format LC-MS files need to be loaded into MZmine 2 before peak picking, or when designing a processing pipeline that must import MS data without precision loss from resampling.

## When NOT to use

- Input is already a peak list or aligned feature table — peak detection, alignment, or gap-filling modules, not raw import, should be applied.
- Only GC-MS or proteomics search-engine interoperation is required — MZmine 2 modules were tested mainly on LC-MS data and GC-MS / MASCOT support is future expansion, not demonstrated.

## Inputs

- Raw LC-MS data files (e.g. vendor or open MS formats) loaded via the MZmine 2 data import module

## Outputs

- Imported raw data with full mass precision preserved, available to filtering/smoothing and peak-picking modules; original unprocessed data retained throughout the session

## How to apply

Begin the MZmine 2 workflow by importing the raw data files with the data import module, which was explicitly designed to maintain the precision of the imported data without degradation due to inadequate resampling. MZmine 2 preserves the original, unprocessed data in memory — intermediate peak lists are kept as separate files and the raw data is never removed — so import is a non-destructive step. After import, optionally run the filtering/smoothing plug-in (e.g. Savitzky-Golay or wavelet-based) before peak picking, which can be set up with online previews linked to the embedded visualization modules. Keep each resulting intermediate peak list in a separate file so the full provenance chain from raw import to normalized output is retained.

## Related tools

- **MZmine 2** (Provides the data import module that loads raw MS data while preserving mass precision; all downstream modules operate on the imported data) — http://mzmine.sourceforge.net/
- **Java** (Implementation platform — MZmine 2 is developed in Java, making it platform independent for import across operating systems)
- **JFreeChart** (Embedded visualization library used by MZmine 2 visualizers for previewing imported and processed data during parameter setup)

## Evaluation signals

- Imported data retains the original mass precision — no degradation from inadequate resampling should be observable in the imported raw data.
- The original unprocessed data remains available after import; MZmine 2 never removes it, and each intermediate peak list is kept as a separate file.
- Subsequent peak-picking parameter setup can use the embedded visualization previews over the imported data (TIC, spectra, 2D views via JFreeChart).
- Imported files feed the expected workflow chain: raw import → filtering/smoothing (optional) → peak picking → deisotoping → alignment → gap filling → normalization.

## Limitations

- Developed modules were tested mainly on LC-MS data; validation on other data types (e.g. GC-MS) is described only as possible future expansion.
- Interoperation with proteomics search engines such as MASCOT is only mentioned as possible future expansion.
- MZmine 2 requires Java; MZmine 2 development required JDK version 12 or newer.

## Evidence

- [results] Import is non-destructive; original data preserved: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
- [methods] Import preserves mass precision without resampling degradation: "We designed the data import and peak detection modules to maintain the precision of the imported data without any degradation due to inadequate resampling."
- [methods] Previewing during parameter setup relies on linked visualization modules: "critical data processing methods such as peak picking were linked to embedded visualization modules, providing online previews during parameter setup."
- [methods] MZmine 2 implementation language and platform independence: "MZmine 2 was developed using Java technology, and is therefore platform independent."
- [results] Original unprocessed data retained through the workflow: [evidence span withheld — anchor status: not_found]
- [abstract] MZmine 2 obtained from project website: "can be obtained from the project website at: http://mzmine.sourceforge.net/."
