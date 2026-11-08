---
name: benchmark-reproduction-against-published-values
description: Use when a paper reports a concrete quantitative result (peak detection
  rate, annotation count, threshold behavior) and you must rerun the pipeline with
  the stated parameters to confirm or compare against those values.
when_to_use_negative:
- Your goal is novel discovery rather than validation of a published result — the
  skill is anchored to reproducing reported numbers, not exploring new datasets.
- 'Your data is 1D GC–MS or LC–MS: GcDUO''s readFolderCDF and procesData assume 2D
  chromatography with a defined modulation time.'
- You lack the paper's stated parameters (thresholds, m/z range, modulation time);
  re-deriving them is a different task than benchmark reproduction.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: Open-source R package used to run the full pipeline (readFolderCDF, procesData,
    duoID, ConsProcessing) whose published fruitybeer benchmark is being reproduced
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Runtime environment in which GcDUO is installed via devtools and the benchmark
    workflow is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:1d6b6f3ba80e1ef1bfb937b3cad5bc36d6a2a5cd096e7bb01bd68abbe0853dc6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - GcDUO
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-reproduction-against-published-values

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a published computational result end-to-end (here, GcDUO annotating 85% of the 27 fruitybeer GC×GC–MS peaks) by following the paper's exact parameters and validating outputs against the reported numbers.

## When to use

Use when a paper reports a concrete quantitative result (peak detection rate, annotation count, threshold behavior) and you must rerun the pipeline with the stated parameters to confirm or compare against those values.

## When NOT to use

- Your goal is novel discovery rather than validation of a published result — the skill is anchored to reproducing reported numbers, not exploring new datasets.
- Your data is 1D GC–MS or LC–MS: GcDUO's readFolderCDF and procesData assume 2D chromatography with a defined modulation time.
- You lack the paper's stated parameters (thresholds, m/z range, modulation time); re-deriving them is a different task than benchmark reproduction.

## Inputs

- GC×GC–MS raw data in CDF format (fruitybeer dataset)
- In-house spectral library as MSP file
- Published parameter set from the source article (modulation time, m/z range, S/N thresholds, cosine cutoff, RI tolerance)
- Reported benchmark values from the article (e.g., 27 peaks, 85% detection rate)

## Outputs

- finalGcDuo$peaks table of detected features
- duoID annotation table with cosine-similarity matches
- finalGcDuo$area and $intensity tables per sample
- Reproduction comparison report (detected/27, annotated fraction, deviations vs. paper values)

## How to apply

Install the tool exactly as the authors specify (devtools::install_github("mariallr/GcDUO") with the EBImage dependency via BiocManager). Import the raw data with the paper's instrument settings — for fruitybeer, readFolderCDF with modulationTime = 2.5 s and mzRange = c(35, 440). Run peak picking with the article's parameters (procesData, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3), build the spectral library from the in-house MSP via getMSP/spectramatrix, and annotate with duoID at match_cutoff = 0.7 cosine similarity and RIrange = 30. Integrate peak areas/intensities with ConsProcessing (win_width = 20), export the peak and annotation tables, then compare your counts and match rates against the reported values (85% of 27 peaks detected; three detected-but-unannotated peaks below cosine 0.7; one compound missing from the library; 16 of 33 library entries lacking a retention index). Flag any deviation and check whether it stems from a parameter choice (e.g., decane requires lowering the signal-to-noise threshold from 3 to 1) rather than a pipeline error.

## Related tools

- **GcDUO** (Open-source R package used to run the full pipeline (readFolderCDF, procesData, duoID, ConsProcessing) whose published fruitybeer benchmark is being reproduced) — https://github.com/mariallr/GcDuo
- **R** (Runtime environment in which GcDUO is installed via devtools and the benchmark workflow is executed)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440)); DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- Detected-peak fraction matches the reported 85% of the 27 fruitybeer peaks
- Exactly three peaks are detected but unannotated (no library compound with cosine similarity > 0.7), with spectra showing characteristic fragments of the expected compounds
- Only one compound is missing from the in-house library, and 16 of 33 library entries lack a retention index (RI = NA)
- The expected compound appears in the matched lists for each annotated peak
- Exported CSVs (fruitybeer_peak_detection.csv, fruitybeer_annotations.csv) are non-empty and consistent with finalGcDuo$peaks and duoID outputs

## Limitations

- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted, so not all reported peaks are recoverable (non-detected peaks in the fruitybeer study)
- Detected peaks with no library compound above cosine 0.7 are returned unannotated even when spectra show the expected compound's fragments
- Decane is missed at signal-to-noise threshold 3 and only detected when lowered to 1 — parameter sensitivity affects reproducibility
- GcDUO is implemented in R, not optimized for large memory loads or multicore parallelism, and may require high-performance computing for large datasets; no software version is pinned in the article

## Evidence

- [results] fruitybeer benchmark detection rate: "three peaks were detected (peak picking) but none of the compounds in the in-house library exhibited a cosine similarity higher than 0.7"
- [results] library gap: "Only one compound was missing from our in-house library."
- [results] small-peak failure mode: "The non-detected peaks correspond to small chromatographic signals with too few characteristic fragments to deconvolute."
- [readme] install command: "devtools::install_github("mariallr/GcDUO")"
- [abstract] validation vs ChromaTOF: "GcDUO was validated against the gold-standard software for comprehensive GC, demonstrating a high correlation (R2 = 0.9) in peak area measurements"
