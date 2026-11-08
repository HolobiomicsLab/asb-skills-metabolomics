---
name: metabolomics-tool-comparison-analysis
description: Use when benchmarking LC-MS metabolomics feature-detection tools (asari
  vs XCMS vs MZmine) against certified/verified feature lists such as the 402 Thermo-verified
  features on HZV029, 314 Chen et al. certified features on Yeast2021, or 643 certified
  E. coli features.
when_to_use_negative:
- Input is already a processed feature table — no raw mzML feature extraction is needed
- Data are low resolution or from non-Orbitrap platforms — asari's settings 'will
  need further optimization for other platforms' and 'the software does not work with
  low resolution data'
- The analysis goal is MS/MS-based identification — 'MS/MS spectra are ignored by
  asari' in the default LC-MS workflow
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: asari
  role: Primary tool under benchmark; run with default parameters (5 ppm m/z precision,
    positive ionization mode) to extract features
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Baseline comparison tool run with comparable settings on the same datasets
- name: MZmine
  role: Baseline comparison tool run with comparable settings on the same datasets
- name: ThermoRawFileParser
  role: Converts Thermo .RAW benchmark files to .mzML before processing
  repo: https://github.com/compomics/ThermoRawFileParser
- name: khipu
  role: Pre-annotation of isotopic patterns / grouping ions to empirical compounds
    for the extracted features
  repo: https://github.com/shuzhao-li-lab/khipu
- name: JMS
  role: Isotopic pattern annotation and mass search utilities used with khipu during
    feature annotation
  repo: https://github.com/shuzhao-li/JMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:e778955c04c63550d21b230a17c101e7cefffdcf94d8295ad94f7fee698930b5
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
yes_rate: 0.6470588235294118
n_claims: 17
---

# metabolomics-tool-comparison-analysis

## Summary

Reproduce multi-dataset LC-MS feature detection benchmarks by running asari (default settings, 5 ppm m/z precision, positive mode) against XCMS and MZmine and matching detected features to manually verified reference lists. This establishes a quantified comparison of detection counts per tool per dataset.

## When to use

Use when benchmarking LC-MS metabolomics feature-detection tools (asari vs XCMS vs MZmine) against certified/verified feature lists such as the 402 Thermo-verified features on HZV029, 314 Chen et al. certified features on Yeast2021, or 643 certified E. coli features.

## When NOT to use

- Input is already a processed feature table — no raw mzML feature extraction is needed
- Data are low resolution or from non-Orbitrap platforms — asari's settings 'will need further optimization for other platforms' and 'the software does not work with low resolution data'
- The analysis goal is MS/MS-based identification — 'MS/MS spectra are ignored by asari' in the default LC-MS workflow

## Inputs

- Benchmark LC-MS datasets: HZV029, Yeast2021, and E. coli with certified/verified feature lists
- Thermo .RAW files (converted to centroided .mzML via ThermoRawFileParser or msConvert)
- Manually verified feature reference lists (402 Thermo-verified HZV029 features, 314 Yeast2021 certified features, 643 E. coli certified features)

## Outputs

- Detection counts per tool per dataset matched to verified features (e.g. 386/402 on HZV029)
- Total extracted feature counts with correct isotopic patterns (e.g. 1,525 for asari vs 1,399 for XCMS on E. coli)
- Summary comparison table mirroring Fig. 4a-c

## How to apply

Obtain the three benchmark datasets (HZV029, Yeast2021, E. coli certified feature lists) and convert any Thermo .RAW files to centroided .mzML using ThermoRawFileParser. Run asari with default parameters (5 ppm m/z precision, positive ionization mode) on each dataset; annotate isotopic patterns via the khipu/JMS packages. Run XCMS and MZmine with comparable settings on the same data as baseline comparators. Match each tool's detected features against the verified reference lists (402 features on HZV029, 314 on Yeast2021, 643 on E. coli) and compute detection counts per tool per dataset. Validate against the published reproduction targets: 386/402 on HZV029, 310/314 on Yeast2021, and 621/643 on E. coli (vs 581 for XCMS), plus total feature counts such as 1,525 for asari vs 1,399 for XCMS on E. coli. Compile results into a summary table mirroring Fig. 4a-c.

## Related tools

- **asari** (Primary tool under benchmark; run with default parameters (5 ppm m/z precision, positive ionization mode) to extract features) — https://github.com/shuzhao-li/asari
- **XCMS** (Baseline comparison tool run with comparable settings on the same datasets)
- **MZmine** (Baseline comparison tool run with comparable settings on the same datasets)
- **ThermoRawFileParser** (Converts Thermo .RAW benchmark files to .mzML before processing) — https://github.com/compomics/ThermoRawFileParser
- **khipu** (Pre-annotation of isotopic patterns / grouping ions to empirical compounds for the extracted features) — https://github.com/shuzhao-li-lab/khipu
- **JMS** (Isotopic pattern annotation and mass search utilities used with khipu during feature annotation) — https://github.com/shuzhao-li/JMS

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- asari default settings detect 386 of 402 Thermo-verified features on HZV029 (second highest, next to MS-DIAL)
- asari detects 310 of 314 certified features on Yeast2021 and 621 of 643 on E. coli (vs 581 for XCMS)
- Total extracted features with correct isotopic patterns: 1,525 for asari vs 1,399 for XCMS on the E. coli dataset
- Benchmark tables reproduce Fig. 4a-c counts from the paper
- Feature tables match against reference lists with correct m/z and retention time correspondence (5 ppm precision)

## Limitations

- Asari settings are mostly optimized for Orbitrap platforms and 'will need further optimization for other platforms'
- 'The software does not work with low resolution data' and 'the efficiency is not optimal on data from longer chromatography'
- Data of very high noise may need customized processing; asari ignores MS/MS spectra in the default LC-MS workflow

## Evidence

- [results] On the HZV029 benchmark of 402 Thermo-verified features, asari in its default setting detected 386 of these features: [evidence span withheld — anchor status: not_found]
- [intro] Multiple software tools have been developed for LC-MS metabolomics data processing: "Multiple software tools have been developed for LC-MS metabolomics data processing over the years4–11, and the most widely used are XCMS9 and MZmine7."
- [methods] We use ThermoRawFileParser to convert Thermo .RAW files to .mzML: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms: "Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms."
- [readme] Input data are centroid mzML files from LC, GC or DI-MS metabolomics: [evidence span withheld — anchor status: external_doc]
