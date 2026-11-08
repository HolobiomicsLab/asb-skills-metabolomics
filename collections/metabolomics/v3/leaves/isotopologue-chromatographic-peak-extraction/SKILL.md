---
name: isotopologue-chromatographic-peak-extraction
description: Use when generating a benchmark dataset from user-provided molecules
  with known composition and retention-time boundaries, requiring isotopologue-level
  peak extraction from mzML files before NPP reliability assessment.
when_to_use_negative:
- When you already have an NPP output feature table to assess — this skill builds
  the benchmark, not the reliability-assessment comparison.
- When input data are profile-mode mzML or non-mzML vendor formats — files must be
  centroided and converted via MSconvert first.
- When target molecules lack known retention-time boundaries or a valid main_adduct
  (e.g. M+H or M-H) — peaks cannot be extracted or validated.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: R tool implementing the benchmark-generation cascade (get_mz_table, get_ROIs,
    find_bench_peaks) via its Generate Benchmark tab
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation language; mzRAPP is launched with library(mzRAPP); callmzRAPP()
  repo: https://github.com/YasinEl/mzRAPP
- name: Skyline
  role: Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries
    can be exported for the target file
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:22995b9844e6d385a61904ccb7bffc0d3514d34527cd5143bb97917e2f2bcebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - Skyline
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# isotopologue-chromatographic-peak-extraction

## Summary

Extract and validate chromatographic peaks for all enviPat-predicted isotopologues of target molecules directly from centroided mzML files to build an LC-HRMS benchmark dataset for reliability assessment of non-targeted data pre-processing (NPP).

## When to use

Use when generating a benchmark dataset from user-provided molecules with known composition and retention-time boundaries, requiring isotopologue-level peak extraction from mzML files before NPP reliability assessment.

## When NOT to use

- When you already have an NPP output feature table to assess — this skill builds the benchmark, not the reliability-assessment comparison.
- When input data are profile-mode mzML or non-mzML vendor formats — files must be centroided and converted via MSconvert first.
- When target molecules lack known retention-time boundaries or a valid main_adduct (e.g. M+H or M-H) — peaks cannot be extracted or validated.

## Inputs

- Centroided mzML files (e.g. 30 MTBLS267 files ending on '_POS.mzML')
- Sample-group CSV (sample_name and sample_group columns)
- Target-file CSV with per-isotopologue RT borders (molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax)
- Instrument/resolution selection (enviPat resolution list or custom R/mz CSV)

## Outputs

- Benchmark.csv containing 47 molecules / 157 features (all adducts and isotopologues) / 2870 peaks with peak heights spanning several orders of magnitude
- Optionally an exported Skyline transition list and peak boundaries

## How to apply

In mzRAPP's Generate Benchmark tab (or via R), load centroided mzML files, a sample-group CSV, and a target CSV specifying molecule names, SumForm_c neutral compositions, main_adduct, user.rtmin/user.rtmax in seconds, and optional adduct_c/FileName columns. Select the instrument/resolution (e.g. OrbitrapXL,Velos,VelosPro_R60000@400) so enviPat can predict isotopologues at the correct mass resolution, and enable additional adducts such as M+NH4, M+Na, and M+K. Run the cascade get_mz_table → get_ROIs → find_bench_peaks with parameters: lowest isotopologue fraction 0.05, minimum 6 scans per peak, mz precision 6 ppm, and mz accuracy 5 ppm. Peaks are only accepted when the theoretically most abundant isotopologue plus at least one additional isotopologue are detected, with peak boundaries narrowed to intersect the EIC at 5% of maximum peak height. Filtering removes isotopologues whose area/height deviates more than 30% from the predicted ratio or whose Pearson correlation with the most abundant isotopologue is < 0.85; export the result as Benchmark.csv.

## Related tools

- **mzRAPP** (R tool implementing the benchmark-generation cascade (get_mz_table, get_ROIs, find_bench_peaks) via its Generate Benchmark tab) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation language; mzRAPP is launched with library(mzRAPP); callmzRAPP()) — https://github.com/YasinEl/mzRAPP
- **Skyline** (Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries can be exported for the target file) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
library(mzRAPP); callmzRAPP()  # then use the Generate Benchmark tab to load mzMLs, SampleGroups_MTBLS267.csv and Target_File_MTBLS267.csv
```

## Evaluation signals

- Benchmark.csv matches the MTBLS267 reference: 47 molecules, 157 features (including all adducts and isotopologues), and 2870 peaks in total.
- Every retained isotopologue has a Pearson correlation ≥ 0.85 with the most abundant isotopologue and an area/height within 30% of the predicted isotopologue-ratio value.
- Each benchmarked compound shows the theoretically most abundant isotopologue plus at least one additional isotopologue; adducts only appear if the main_adduct was found.
- All retained peaks have at least 6 scans, mz spread within 6 ppm precision and 5 ppm accuracy, and boundaries intersecting the EIC at 5% of maximum peak height.

## Limitations

- Low-abundance isotopologues are frequently removed by the peak-shape correlation (≥ 0.85) and isotopologue-ratio bias (< 30%) filters, so the benchmark under-represents low peaks.
- The main_adduct must be detected for any other adduct of the same molecule to be accepted; additionally at least two isotopologues per adduct must be detectable.
- If user.rtmin/user.rtmax are provided, peaks can still be rejected when the isotopic information does not fit the prediction.

## Evidence

- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [readme] Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are considered for the final benchmark.: [evidence span withheld — anchor status: external_doc]
- [methods] In the Generate Benchmark tab, you have to select all necessary files and set the instrument/resolution used.: "In the <i>Generate Benchmark</i> tab, you have to select all necessary files and set the instrument/resolution used."
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue): "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue)"
