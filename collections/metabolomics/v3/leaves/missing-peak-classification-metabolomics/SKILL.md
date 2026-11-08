---
name: missing-peak-classification-metabolomics
description: Use when a processed LC-HRMS feature/peak table (e.g. MZmine 2 aligned
  or unaligned csv output) must be assessed for which benchmark peaks are absent and
  why, as part of NPP reliability assessment.
when_to_use_negative:
- Input is a targeted metabolomics feature table already validated against a transition
  list (e.g. Skyline output) — the skill assesses non-targeted pre-processing against
  a benchmark, not targeted results
- No benchmark dataset with user-supplied retention-time boundaries and molecular
  formulas is available for the mzML files — classification against matched benchmark
  peaks cannot be performed
- The NPP output comes from a tool not among the commonly supported pre-processing
  tools (XCMS, MZmine 2/3, MS-DIAL, OpenMS, El-MAVEN, etc.) with a csv export mzRAPP
  can parse
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: Provides the benchmark, the matching routines (match_peaks_to_benchmark, match_NPPpeaks_to_NPPfeatures)
    and the missing-peak classification routines (check_missing_peaks, classify_false_negative)
    used to categorize missed peaks
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation language of mzRAPP; routines are invoked via R/devtools installation
    of the GitHub package
  repo: https://github.com/YasinEl/mzRAPP
- name: MZmine 2
  role: Source of the example NPP output (30 unaligned csv files plus MZmine_aligned.csv)
    whose missing peaks are classified against the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:ee726ce8df6b52bbdf1c909814de3e017ad3ab521cb5902158d523b1ccea4606
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MZmine 2
  - R
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# missing-peak-classification-metabolomics

## Summary

Classifies peaks missing from a non-targeted pre-processing (NPP) output by comparing it against a validated mzRAPP benchmark, distinguishing the nature of missed peaks as a performance metric of tools like MZmine 2, XCMS, or MS-DIAL on LC-HRMS data.

## When to use

Use when a processed LC-HRMS feature/peak table (e.g. MZmine 2 aligned or unaligned csv output) must be assessed for which benchmark peaks are absent and why, as part of NPP reliability assessment.

## When NOT to use

- Input is a targeted metabolomics feature table already validated against a transition list (e.g. Skyline output) — the skill assesses non-targeted pre-processing against a benchmark, not targeted results
- No benchmark dataset with user-supplied retention-time boundaries and molecular formulas is available for the mzML files — classification against matched benchmark peaks cannot be performed
- The NPP output comes from a tool not among the commonly supported pre-processing tools (XCMS, MZmine 2/3, MS-DIAL, OpenMS, El-MAVEN, etc.) with a csv export mzRAPP can parse

## Inputs

- Benchmark.csv — mzRAPP benchmark dataset generated from target molecules with known SumForm_c, main_adduct and user.rtmin/user.rtmax boundaries
- NPP pre-processing output in csv format (e.g. 30 unaligned MZmine 2 files from MZmine_unaligned folder plus MZmine_aligned.csv)
- Sample-group file (e.g. SampleGroups_MTBLS267.csv)
- Options table with benchmark-generation parameters (lowest isotopologue 0.05, min. 6 scans per peak, 6 ppm mz precision, 5 ppm mz accuracy)

## Outputs

- Missing-peak/false-negative classification table per benchmark molecule
- NPP assessment metrics table (e.g. mzmine2_npp_assessment_metrics.csv) with post-alignment detection rate, degenerated IR proportion, and alignment error rate
- Sunburst plots of found/not-found peaks (plot_sunburst_peaks) showing classified missing peaks

## How to apply

Load the benchmark (Benchmark.csv) generated from targeted mzML peak extraction, and the NPP output (e.g. 30 unaligned MZmine 2 csv files plus MZmine_aligned.csv), parsing them with mzRAPP's import_ungrouped_mzmine/import_grouped_mzmine and check_nonTargeted_input routines. Match benchmark peaks to NPP peaks and features using match_peaks_to_benchmark and match_NPPpeaks_to_NPPfeatures, then run the missing-peak classification routines (check_missing_peaks, classify_false_negative) to characterize the nature of missed peaks. The benchmark itself is filtered during generation: isotopologue peaks with area or height more than 30% off the predicted value or Pearson correlation < 0.85 versus the most abundant isotopologue are removed, so only reliable targets are used for classification. Combine the classified missed peaks with found/not-found counts (visualized via plot_sunburst_peaks) and assemble the results into the metrics table (e.g. mzmine2_npp_assessment_metrics.csv) alongside detection rate and alignment error metrics.

## Related tools

- **mzRAPP** (Provides the benchmark, the matching routines (match_peaks_to_benchmark, match_NPPpeaks_to_NPPfeatures) and the missing-peak classification routines (check_missing_peaks, classify_false_negative) used to categorize missed peaks) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation language of mzRAPP; routines are invoked via R/devtools installation of the GitHub package) — https://github.com/YasinEl/mzRAPP
- **MZmine 2** (Source of the example NPP output (30 unaligned csv files plus MZmine_aligned.csv) whose missing peaks are classified against the benchmark)

## Examples

```
library(mzRAPP); vignette("Vignette_mzRAPP_Example_workflow")
```

## Evaluation signals

- Post-alignment found-peaks proportion falls within the reported 82–92% range for the MZmine 2 MTBLS267 run
- Degenerated isotopologue ratios fall within the reported 1–9% range
- Alignment errors account for up to about 7% of divergences, consistent with the reported MZmine 2 run
- Classification output is assembled into mzmine2_npp_assessment_metrics.csv containing missing-value classifications together with detection and alignment metrics
- Sunburst plots of found/not-found peaks render and partition peaks into found versus classified-missing categories

## Limitations

- Benchmark quality depends on user-supplied peak boundaries; mzRAPP rejects peaks whose isotopic information does not fit, and only isotopologues for which the most abundant plus at least one additional isotopologue are detected are kept
- Benchmark generation removes isotopologue peaks with area/height >30% off the predicted value or Pearson correlation < 0.85 with the highest isotopologue, so low-quality/low-abundance peaks are excluded from classification
- mzRAPP requires centroided mzML files (vendor conversion/centroiding via MSconvert) and an instrument resolution list; parameters like mz precision (6 ppm) and mz accuracy (5 ppm) must match the instrument

## Evidence

- [other] Missing peak classification via benchmark matching: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark-based performance metrics: "mzRAPP produces several performance metrics for different steps of the pre-processing workflow"
- [readme] Benchmark filtering thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark purpose: [evidence span withheld — anchor status: external_doc]
