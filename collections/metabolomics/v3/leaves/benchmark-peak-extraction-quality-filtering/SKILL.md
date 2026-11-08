---
name: benchmark-peak-extraction-quality-filtering
description: Use when a validated benchmark peak set must be derived from chromatographic
  ROIs of target molecules — i.e., after get_ROIs has produced an ROI data.table and
  peaks must be extracted, consistency-checked, and filtered before benchmark export.
when_to_use_negative:
- Input is already a finished benchmark (e.g. the reference Benchmark.csv from ucloud)
  — skip generation and go directly to reliability assessment of NPP output
- mzML files are profile-mode or not centroided — the benchmark requires centroided
  mzML input
- Target molecules lack known retention-time boundaries; user.rtmin/user.rtmax must
  be supplied (e.g. curated via Skyline) before peak extraction is meaningful
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: Provides find_bench_peaks, the peak-extraction and quality-filtering function
    of the benchmark cascade
  repo: https://github.com/YasinEl/mzRAPP
- name: enviPat
  role: Predicts isotopologue patterns and provides the adduct list and instrument
    resolution list used to define which peaks are extracted
- name: XCMS
  role: findmzROI functionality underlies chromatographic region-of-interest extraction
    feeding find_bench_peaks
- name: MSnbase
  role: readMSData and chromatogram provide raw-file access and EIC extraction
- name: Skyline
  role: Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries
    can be exported
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:124abf87f9848f8a6a4caa7d7f2eadf1a0da9083b084b6e0120e6efd2c56d842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MSnbase
  - R
  - Skyline
  - XCMS
  - enviPat
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# benchmark-peak-extraction-quality-filtering

## Summary

The find_bench_peaks stage of mzRAPP's benchmark cascade: extract chromatographic peaks with provided RT boundaries for all predicted isotopologues from ROI tables, then apply peak-shape correlation and isotopologue-ratio filters to yield a validated benchmark data.table.

## When to use

Use when a validated benchmark peak set must be derived from chromatographic ROIs of target molecules — i.e., after get_ROIs has produced an ROI data.table and peaks must be extracted, consistency-checked, and filtered before benchmark export.

## When NOT to use

- Input is already a finished benchmark (e.g. the reference Benchmark.csv from ucloud) — skip generation and go directly to reliability assessment of NPP output
- mzML files are profile-mode or not centroided — the benchmark requires centroided mzML input
- Target molecules lack known retention-time boundaries; user.rtmin/user.rtmax must be supplied (e.g. curated via Skyline) before peak extraction is meaningful

## Inputs

- ROI data.table returned by get_ROIs
- Isotopologue pattern table from get_mz_table (enviPat predictions)
- Target file csv with molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax columns
- Sample-group csv (sample_name, sample_group)
- Centroided mzML files (e.g. 30 MTBLS267 *_POS.mzML files)
- Parameters: Min. # of scans per peak = 6, mz precision = 6 ppm, mz accuracy = 5 ppm, lowest isotopologue = 0.05, isotopologue ratio bias cutoff

## Outputs

- Benchmark data.table of validated peaks for all accepted isotopologues and adducts
- Benchmark.csv exported to the working directory

## How to apply

Feed the ROI data.table from get_ROIs (built from enviPat-predicted isotopologue patterns of the target file, e.g. OrbitrapXL/Velos R60000@400, adducts M+NH4/M+Na/M+K) into find_bench_peaks, extracting peaks for all predicted isotopologues within the user-provided RT boundaries. Apply the configured quality cutoffs: minimum 6 scans per peak, mz precision 6 ppm and mz accuracy 5 ppm, and lowest isotopologue relative abundance >= 0.05. Remove isotopologues that fail peak-shape correlation with the most abundant isotopologue (Pearson correlation < 0.85) or whose area/height deviates more than 30% from the predicted isotopologue ratio (isotopologue ratio bias cutoff). Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are retained, and adducts are only accepted if the main_adduct was detected. The output benchmark data.table is then exported as Benchmark.csv; for MTBLS267 it should contain 47 molecules, 157 features, and 2870 peaks.

## Related tools

- **mzRAPP** (Provides find_bench_peaks, the peak-extraction and quality-filtering function of the benchmark cascade) — https://github.com/YasinEl/mzRAPP
- **enviPat** (Predicts isotopologue patterns and provides the adduct list and instrument resolution list used to define which peaks are extracted)
- **XCMS** (findmzROI functionality underlies chromatographic region-of-interest extraction feeding find_bench_peaks)
- **MSnbase** (readMSData and chromatogram provide raw-file access and EIC extraction)
- **Skyline** (Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries can be exported) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
library(mzRAPP); bench <- find_bench_peaks(get_ROIs(get_mz_table(target_file, sample_groups, instrument='OrbitrapXL/Velos R60000@400', adducts=c('M+NH4','M+Na','M+K'), lowest_iso=0.05), files, min_scans=6, mz_prec=6, mz_acc=5)); fwrite(bench, 'Benchmark.csv')
```

## Evaluation signals

- Benchmark generated from all 30 MTBLS267 mzML files contains 47 molecules, 157 features (including all adducts and isotopologues), and 2870 peaks in total
- Output matches the reference Benchmark.csv deposited on ucloud
- Every retained isotopologue has peak-shape Pearson correlation >= 0.85 with the most abundant isotopologue and area/height within 30% of the predicted isotopologue ratio
- No benchmark entry exists for a molecule whose main_adduct was not detected, and each accepted adduct has at least two detected isotopologues
- Each benchmark peak spans at least the minimum number of scans per peak (6) and lies within the provided user.rtmin/user.rtmax boundaries

## Limitations

- Isotopologue peaks with area or height more than 30% off the predicted value, or Pearson correlation < 0.85 versus the highest isotopologue, are removed — low-abundance isotopologues may be lost
- If the main_adduct is not detected, other adducts for that molecule will not be accepted; adducts are only added if at least two isotopologues are detected
- Peak boundaries can be rejected despite user-provided user.rtmin/user.rtmax if the isotopic information does not fit
- mzRAPP's benchmark approach requires merging targeted information with non-targeted data on the same dataset and supports only LC-HRMS pre-processing assessment

## Evidence

- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [readme] Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are considered for the final benchmark.: [evidence span withheld — anchor status: external_doc]
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue: "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue"
- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] If the main_adduct is not detected also other adducts wont be accepted.: [evidence span withheld — anchor status: not_found]
