---
name: lc-ms-feature-detection-benchmarking
description: Use when evaluating or comparing LC-MS metabolomics feature-detection
  tools against a manually verified/certified reference feature list, or when reproducing
  the asari vs XCMS/MZmine benchmark counts.
when_to_use_negative:
- Input is already a processed feature table with no raw mzML/Raw data available —
  this benchmark requires running the full processing pipelines from raw data
- Data are from low-resolution instruments or non-Orbitrap platforms without parameter
  re-optimization — asari is explicitly not validated for low-resolution data
- The goal is quantification accuracy rather than feature detection benchmarking against
  certified feature lists
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Primary tool under benchmarking; run in default mode (5 ppm, positive) to
    detect features and annotate isotopic patterns via khipu/JMS
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Baseline comparison tool run with comparable settings on the same datasets
- name: MZmine
  role: Baseline comparison tool run with comparable settings on the same datasets
- name: khipu
  role: Pre-annotation of degenerate ions / isotopologues used by asari to group features
    into empirical compounds
  repo: https://github.com/shuzhao-li-lab/khipu
- name: JMS
  role: Reusable data structures for peaks and empirical compound search used alongside
    khipu in asari annotation
  repo: https://github.com/shuzhao-li/JMS
- name: ThermoRawFileParser
  role: Converts Thermo .RAW benchmark files to .mzML prior to processing
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:be36f7d7afb36e218896183b8b57dba4a84dd647b347826c51dd3e2c53a4c8be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - JMS
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  - khipu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lc-ms-feature-detection-benchmarking

## Summary

Reproduce the asari feature-detection benchmarks from Li et al. 2023 by running asari (default parameters, 5 ppm m/z precision, positive mode) against XCMS and MZmine on the HZV029, Yeast2021, and E. coli certified feature lists, and counting how many verified features each tool detects.

## When to use

Use when evaluating or comparing LC-MS metabolomics feature-detection tools against a manually verified/certified reference feature list, or when reproducing the asari vs XCMS/MZmine benchmark counts.

## When NOT to use

- Input is already a processed feature table with no raw mzML/Raw data available — this benchmark requires running the full processing pipelines from raw data
- Data are from low-resolution instruments or non-Orbitrap platforms without parameter re-optimization — asari is explicitly not validated for low-resolution data
- The goal is quantification accuracy rather than feature detection benchmarking against certified feature lists

## Inputs

- Thermo .RAW files or centroided mzML files for the benchmark datasets (HZV029, Yeast2021, E. coli)
- Certified/verified reference feature lists: 402 Thermo-verified features (HZV029), 314 Chen et al. certified features (Yeast2021), 643 certified E. coli features
- Benchmarked processing tools: asari, XCMS, MZmine (with khipu/JMS for asari annotation)

## Outputs

- Detection counts per tool per dataset matched to the certified lists (expected: 386/402 on HZV029, 310/314 on Yeast2021, 621/643 for asari vs 581 for XCMS on E. coli)
- Total extracted features with correct isotopic patterns per tool (e.g. 1,525 asari vs 1,399 XCMS on E. coli)
- Summary table mirroring Fig. 4a-c of the paper

## How to apply

Obtain the three benchmark datasets (HZV029 with 402 Thermo-verified features, Yeast2021 with 314 Chen et al. certified features, and 643 certified E. coli features). Convert Thermo .RAW files to .mzML with ThermoRawFileParser (or msConvert for other vendor formats), then run asari with default parameters (5 ppm m/z precision, positive ionization mode) on each dataset, using khipu and JMS for isotopic pattern and empirical compound annotation. Run XCMS and MZmine with comparable settings on the same data for baseline comparison. Match each tool's detected features against the certified/verified reference lists, count detections per tool per dataset, and also record total extracted features with correct isotopic patterns (e.g. 1,525 for asari vs 1,399 for XCMS on E. coli). Compile results into a summary table mirroring Fig. 4a-c.

## Related tools

- **asari** (Primary tool under benchmarking; run in default mode (5 ppm, positive) to detect features and annotate isotopic patterns via khipu/JMS) — https://github.com/shuzhao-li/asari
- **XCMS** (Baseline comparison tool run with comparable settings on the same datasets)
- **MZmine** (Baseline comparison tool run with comparable settings on the same datasets)
- **khipu** (Pre-annotation of degenerate ions / isotopologues used by asari to group features into empirical compounds) — https://github.com/shuzhao-li-lab/khipu
- **JMS** (Reusable data structures for peaks and empirical compound search used alongside khipu in asari annotation) — https://github.com/shuzhao-li/JMS
- **ThermoRawFileParser** (Converts Thermo .RAW benchmark files to .mzML prior to processing) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- asari default-mode detection on HZV029 matches 386 of 402 Thermo-verified features (second highest among compared tools, next to MS-DIAL)
- Yeast2021 reproduction yields 310 of 314 Chen et al. certified features for asari
- E. coli reproduction yields 621 of 643 certified features for asari vs 581 for XCMS
- Total features with correct isotopic patterns on E. coli are ~1,525 for asari vs ~1,399 for XCMS
- Per-dataset counts agree across tools within the ranges reported in the paper's Fig. 4a-c summary table

## Limitations

- Asari's settings were mostly tested on Orbitrap platforms; they need optimization for other platforms and the software does not work with low-resolution data
- Efficiency is not optimal on data from longer chromatography, and very high-noise data may need customized processing
- ppm precision is only a practical approximation; a 2nd order polynomial model may be required to cover the full m/z range more accurately
- MS/MS spectra are ignored by asari in the default LC-MS workflow

## Evidence

- [results] HZV029 benchmark detection count: [evidence span withheld — anchor status: not_found]
- [results] benchmark setup: [evidence span withheld — anchor status: not_found]
- [abstract] asari favorable comparison: "Asari compares favorably to other tools in feature detection and quantification"
- [readme] raw conversion: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML."
- [discussion] platform limitation: "Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms."
