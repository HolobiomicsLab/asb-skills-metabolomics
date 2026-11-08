---
name: r-datatable-pipeline-orchestration
description: Reference for implementing chained R data-processing functions where
  the output of one stage (a data.table) feeds the next, e.g. building an LC-HRMS
  benchmark from raw mzML files.
when_to_use_negative:
- You already have a prepared benchmark (e.g. Benchmark.csv from ucloud) and only
  need to run the reliability assessment against NPP outputs — skip generation entirely.
- Your pre-processing tool is not among the five supported NPP tools and you cannot
  export its output in a matching format.
- Your molecules lack known retention-time boundaries or molecular composition, since
  enviPat isotopologue prediction and user.rtmin/user.rtmax are prerequisites.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzRAPP
  role: R package providing the get_mz_table, get_ROIs and find_bench_peaks cascade
    and the benchmark export
  repo: https://github.com/YasinEl/mzRAPP
- name: enviPat
  role: predicts isotopologue patterns and provides the adduct and instrument resolution
    lists
- name: XCMS
  role: xcmsRaw and findmzROI used inside get_ROIs to extract chromatographic regions
    of interest
- name: MSnbase
  role: readMSData and chromatogram used to load mzML files and extract chromatograms
- name: data.table
  role: data structure returned by each pipeline stage and passed to the next
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:b138a0383ce432e3372a9f7feb428dd3de31131e061fcf824bb251835b2f8f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MSnbase
  - R
  - XCMS
  - data.table
  - enviPat
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# r-datatable-pipeline-orchestration

## Summary

Orchestrate a fixed multi-stage R pipeline in which each function returns a data.table that is passed directly as the argument of the next function, as in mzRAPP's benchmark generation cascade (get_mz_table -> get_ROIs -> find_bench_peaks) built on enviPat, XCMS and MSnbase.

## When to use

Reference for implementing chained R data-processing functions where the output of one stage (a data.table) feeds the next, e.g. building an LC-HRMS benchmark from raw mzML files.

## When NOT to use

- You already have a prepared benchmark (e.g. Benchmark.csv from ucloud) and only need to run the reliability assessment against NPP outputs — skip generation entirely.
- Your pre-processing tool is not among the five supported NPP tools and you cannot export its output in a matching format.
- Your molecules lack known retention-time boundaries or molecular composition, since enviPat isotopologue prediction and user.rtmin/user.rtmax are prerequisites.

## Inputs

- centroided mzML files (e.g. 30 MTBLS267 *_POS.mzML files, at least 5)
- sample-group csv (sample_name and sample_group columns)
- target csv with molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax (and optional adduct_c, StartTime.EIC, EndTime.EIC, FileName columns)
- instrument/resolution selection (enviPat resolution list or custom csv with R and m/z columns)

## Outputs

- data.table of predicted isotopologue patterns (from get_mz_table)
- data.table of chromatographic ROIs (from get_ROIs)
- benchmark data.table (from find_bench_peaks), exportable as Benchmark.csv

## How to apply

Structure the workflow as three sequential functions, each returning a data.table that serves as the argument for the subsequent function. First, run get_mz_table on a target file (e.g. MTBLS267 Target_File_MTBLS267.csv) and sample-group csv to predict isotopologue patterns with enviPat, selecting the instrument resolution (here OrbitrapXL/Velos R60000@400), adducts (here M+NH4, M+Na, M+K in addition to the main adduct), and a lowest isotopologue threshold of 0.05. Second, feed the returned table to get_ROIs, which uses MSnbase (readMSData, chromatogram) to extract chromatographic regions of interest directly from the centroided *_POS.mzML files (30, or at least 5) with Min.PointsperPeak = 6 scans, mz precision = 6 ppm and mz accuracy = 5 ppm; get_ROIs internally relies on XCMS's xcmsRaw and findmzROI. Third, run find_bench_peaks on the ROI table to extract and validate benchmark peaks with the provided RT boundaries for all predicted isotopologues, removing isotopologues that fail peak-shape correlation with the most abundant isotopologue (Pearson correlation < 0.85) or exceed the isotopologue ratio bias cutoff (area/height more than 30% off the predicted value). Finally export the finished benchmark data.table, e.g. as Benchmark.csv to the working directory.

## Related tools

- **mzRAPP** (R package providing the get_mz_table, get_ROIs and find_bench_peaks cascade and the benchmark export) — https://github.com/YasinEl/mzRAPP
- **enviPat** (predicts isotopologue patterns and provides the adduct and instrument resolution lists)
- **XCMS** (xcmsRaw and findmzROI used inside get_ROIs to extract chromatographic regions of interest)
- **MSnbase** (readMSData and chromatogram used to load mzML files and extract chromatograms)
- **data.table** (data structure returned by each pipeline stage and passed to the next)

## Examples

```
library(mzRAPP); bm_table <- get_mz_table(target_file, sample_groups, resolution='OrbitrapXL_Velos_R60000@400', adducts=c('M+NH4','M+Na','M+K'), lowest_iso=0.05); rois <- get_ROIs(bm_table, mzml_files); benchmark <- find_bench_peaks(rois)
```

## Evaluation signals

- Benchmark generated from all 30 MTBLS267 mzML files contains 47 different molecules with 157 different features (including all adducts and isotopologues) and 2870 peaks in total.
- Generated Benchmark.csv matches the reference Benchmark.csv deposited on ucloud.
- Each pipeline stage returns a valid data.table object usable as the argument of the next function.
- Only isotopologues for which the most abundant and at least one additional isotopologue are found are retained; peaks with Pearson correlation < 0.85 to the highest isotopologue or area/height > 30% off the predicted value are removed.
- Adducts are only added if at least two isotopologues of the respective adduct can be detected, and only if the main_adduct was found.

## Limitations

- mzML files must be centroided (e.g. via ProteoWizard MSconvert); profile data will not work.
- main_adduct must be detected or other adducts will not be accepted; adducts are added only if at least two isotopologues are detected.
- Lowest isotopologue threshold must be >= 0.05; low-quality peaks are removed by peak-shape correlation and isotopologue ratio bias criteria, so weak isotopologues may be lost.
- The cascade is fixed — get_mz_table, get_ROIs and find_bench_peaks must run in that exact order, each consuming the previous stage's data.table.

## Evidence

- [other] Benchmark generation is a fixed cascade of three functions: [evidence span withheld — anchor status: not_found]
- [other] building on enviPat, XCMS and MSnbase: [evidence span withheld — anchor status: not_found]
- [other] Validation counts for the MTBLS267 benchmark: [evidence span withheld — anchor status: not_found]
- [readme] README validation criteria for isotopologue peaks: [evidence span withheld — anchor status: external_doc]
- [readme] README parameters for benchmark generation: [evidence span withheld — anchor status: not_found]
- [readme] README requirement on adduct detection: [evidence span withheld — anchor status: external_doc]
