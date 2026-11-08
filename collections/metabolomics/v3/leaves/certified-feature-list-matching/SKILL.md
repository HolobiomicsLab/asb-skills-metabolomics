---
name: certified-feature-list-matching
description: Use when benchmarking feature detection of LC-MS processing tools on
  datasets with known ground truth, e.g. HZV029 (402 Thermo-verified features), Yeast2021
  (314 certified features), or E. coli (643 certified features).
when_to_use_negative:
- Input is already a feature table that has been annotated or matched to a compound
  database (e.g. via khipu pre-annotation against HMDB 4) — this skill benchmarks
  detection recall, not compound annotation
- The dataset has no manually verified or certified ground-truth feature list, so
  detection counts cannot be validated against reference features
- MS/MS-based identification is the goal — asari ignores MS/MS spectra in its default
  LC-MS workflow, so this matching is on MS1 features only
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: asari
  role: Primary LC-MS feature detection tool run with default parameters (5 ppm m/z
    precision, positive ionization mode) whose feature output is matched against certified
    lists
  repo: github.com/shuzhao-li-lab/asari
- name: XCMS
  role: Baseline processing tool run with comparable settings to produce comparison
    feature lists matched against the same reference lists
- name: MZmine
  role: Baseline processing tool run with comparable settings for the multi-tool feature
    detection comparison
- name: khipu
  role: Pre-annotation tool grouping ions into empirical compounds used alongside
    asari's feature output for isotopic-pattern-aware benchmarking
  repo: https://github.com/shuzhao-li-lab/khipu
- name: ThermoRawFileParser
  role: Converts Thermo .RAW benchmark files to centroided .mzML input for the processing
    tools
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:5d993a0ba2462357bc36ac100293c0f06cd5b749808b58ae378a7a4a0e5165ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JMS
  - MZmine
  - ThermoRawFileParser
  - XCMS
  - asari
  - khipu
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# certified-feature-list-matching

## Summary

Match LC-MS features extracted by a processing tool (asari, XCMS, MZmine) against manually certified/Thermo-verified reference feature lists to quantify feature detection recall and benchmark tools.

## When to use

Use when benchmarking feature detection of LC-MS processing tools on datasets with known ground truth, e.g. HZV029 (402 Thermo-verified features), Yeast2021 (314 certified features), or E. coli (643 certified features).

## When NOT to use

- Input is already a feature table that has been annotated or matched to a compound database (e.g. via khipu pre-annotation against HMDB 4) — this skill benchmarks detection recall, not compound annotation
- The dataset has no manually verified or certified ground-truth feature list, so detection counts cannot be validated against reference features
- MS/MS-based identification is the goal — asari ignores MS/MS spectra in its default LC-MS workflow, so this matching is on MS1 features only

## Inputs

- Detected feature table from the processing tool (e.g. asari preferred_Feature_table.tsv or full_Feature_table.tsv with m/z, retention time, intensity)
- Certified/verified reference feature list (402 Thermo-verified HZV029 features, 314 Chen et al. Yeast2021 certified features, 643 certified E. coli features)
- Centroided mzML files of the benchmark datasets (converted from Thermo .RAW via ThermoRawFileParser)

## Outputs

- Per-tool, per-dataset count of recovered certified features (e.g. 386/402 HZV029, 310/314 Yeast2021, 621/643 E. coli)
- Total extracted feature counts with correct isotopic patterns for benchmark comparison
- Summary table of detection counts mirroring Fig. 4a-c

## How to apply

Run each processing tool on the benchmark dataset (asari with default parameters: 5 ppm m/z precision, positive ionization mode), then match the tool's extracted feature list against the certified/verified reference list by m/z (and retention time) agreement, e.g. within the 5 ppm mass tolerance asari uses. Count how many reference features are recovered (e.g. 386 of 402 on HZV029, 310 of 314 on Yeast2021, 621 of 643 on E. coli for asari) and record the total number of extracted features with correct isotopic patterns (e.g. 1,525 for asari vs 1,399 for XCMS on E. coli). Compile per-tool, per-dataset detection counts into a summary table mirroring Fig. 4a-c to compare detection counts across tools such as asari, XCMS, and MS-DIAL.

## Related tools

- **asari** (Primary LC-MS feature detection tool run with default parameters (5 ppm m/z precision, positive ionization mode) whose feature output is matched against certified lists) — github.com/shuzhao-li-lab/asari
- **XCMS** (Baseline processing tool run with comparable settings to produce comparison feature lists matched against the same reference lists)
- **MZmine** (Baseline processing tool run with comparable settings for the multi-tool feature detection comparison)
- **khipu** (Pre-annotation tool grouping ions into empirical compounds used alongside asari's feature output for isotopic-pattern-aware benchmarking) — https://github.com/shuzhao-li-lab/khipu
- **ThermoRawFileParser** (Converts Thermo .RAW benchmark files to centroided .mzML input for the processing tools) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process --mode pos --input mydir/projectx_dir  # then match export/preferred_Feature_table.tsv m/z values against the 402-feature HZV029 Thermo-verified list
```

## Evaluation signals

- Reproduced detection counts match reported values: 386 of 402 Thermo-verified features on HZV029, 310 of 314 on Yeast2021, and 621 of 643 (vs 581 for XCMS) on E. coli for asari in default setting
- asari's HZV029 detection count is the second highest among compared tools, next to MS-DIAL
- Total extracted features with correct isotopic patterns on E. coli are 1,525 for asari vs 1,399 for XCMS
- Matched features respect the tool's mass precision (e.g. 5 ppm in asari default) when paired to the reference list

## Limitations

- Inconsistency among processing tools is attributed to deficiencies in mass alignment and feature quality controls, so benchmark counts are sensitive to those steps
- asari settings are mostly tested on Orbitrap platforms; detection counts may differ on other platforms or low-resolution data
- The borderline feature cases are a tradeoff between sensitivity and true discovery rate, affecting which features count as detected

## Evidence

- [results] On the HZV029 benchmark of 402 Thermo-verified features, asari in its default setting detected 386 of these features: [evidence span withheld — anchor status: not_found]
- [results] Match features against certified reference lists: [evidence span withheld — anchor status: not_found]
- [results] Validation counts on Yeast2021 and E. coli: [evidence span withheld — anchor status: not_found]
- [results] asari default parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Asari compares favorably to other tools in feature detection and quantification: "Asari compares favorably to other tools in feature detection and quantification"
