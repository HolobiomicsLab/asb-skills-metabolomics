---
name: lc-hrms-benchmark-dataset-generation
description: Use when you have centroided LC-HRMS mzML files plus a list of molecules
  with known composition (SumForm_c) and user-provided RT borders (user.rtmin/user.rtmax),
  and you need a benchmark to score XCMS, MZmine, MS-DIAL or similar NPP outputs.
when_to_use_negative:
- You only have an already pre-processed feature table (peak/feature matrix) from
  XCMS, MZmine 2 or MS-DIAL — benchmark generation must run directly on raw centroided
  mzML files before NPP is applied.
- Your molecules lack known retention behavior (no user.rtmin/user.rtmax borders),
  since mzRAPP requires RT boundaries per compound, ideally exported from a manual
  peak-curation tool like Skyline.
- Your data are profile-mode or low-resolution — enviPat-based isotopologue prediction
  and ppm-level mz precision/accuracy (6/5 ppm) require centroided high-resolution
  LC-HRMS data.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: R package implementing benchmark generation (get_mz_table, get_ROIs, find_bench_peaks)
    via the Generate Benchmark shiny tab; launched with callmzRAPP()
  repo: https://github.com/YasinEl/mzRAPP
- name: enviPat
  role: Underlying library for isotopologue prediction, adduct list and instrument/resolution
    list used by mzRAPP
- name: Skyline
  role: Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries
    are exported (via 'Export Skyline Transition list and peak boundaries')
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:05b7e408d4e3f9a00c8b7414ebefa6a68bfa8ab6f4ceaad73926ad066a8941bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - Skyline
  - enviPat
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# LC-HRMS benchmark dataset generation for NPP reliability assessment

## Summary

Generates a validated benchmark of chromatographic peaks for enviPat-predicted isotopologues of target molecules directly from centroided mzML files, using mzRAPP's cascade (get_mz_table, get_ROIs, find_bench_peaks). The benchmark is a prerequisite for any non-targeted pre-processing (NPP) reliability assessment with mzRAPP.

## When to use

Use when you have centroided LC-HRMS mzML files plus a list of molecules with known composition (SumForm_c) and user-provided RT borders (user.rtmin/user.rtmax), and you need a benchmark to score XCMS, MZmine, MS-DIAL or similar NPP outputs.

## When NOT to use

- You only have an already pre-processed feature table (peak/feature matrix) from XCMS, MZmine 2 or MS-DIAL — benchmark generation must run directly on raw centroided mzML files before NPP is applied.
- Your molecules lack known retention behavior (no user.rtmin/user.rtmax borders), since mzRAPP requires RT boundaries per compound, ideally exported from a manual peak-curation tool like Skyline.
- Your data are profile-mode or low-resolution — enviPat-based isotopologue prediction and ppm-level mz precision/accuracy (6/5 ppm) require centroided high-resolution LC-HRMS data.

## Inputs

- Centroided mzML files (converted/centroided with ProteoWizard MSconvert)
- Sample-group CSV with columns sample_name and sample_group (e.g. SampleGroups_MTBLS267.csv)
- Target-file CSV with molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax (optional adduct_c, StartTime.EIC, EndTime.EIC, FileName; e.g. Target_File_MTBLS267.csv)
- Instrument/resolution selection from enviPat resolution list (or custom two-column R/m/z CSV)
- Adduct selection from enviPat adducts list (e.g. M+NH4, M+Na, M+K)

## Outputs

- Benchmark.csv — validated benchmark of isotopologue peak boundaries, heights and areas (for the MTBLS267 example: 47 molecules, 157 features, 2870 peaks spanning several orders of magnitude in peak height)

## How to apply

Launch mzRAPP in R via library(mzRAPP); callmzRAPP() and open the Generate Benchmark tab. Load centroided mzML files, a sample-group CSV (columns sample_name, sample_group) and a target CSV with molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax (optionally adduct_c, StartTime.EIC/EndTime.EIC, FileName); RT borders should come from manual peak curation such as a Skyline-exported transition list. Select instrument/resolution from the enviPat list (e.g. OrbitrapXL,Velos,VelosPro_R60000@400 for MTBLS267) and additional adducts (e.g. M+NH4, M+Na, M+K) — these are only accepted if the main_adduct is found and at least two isotopologues of the adduct are detected. Set parameters: lowest isotopologue fraction 0.05, minimum 6 scans per peak, mz precision 6 ppm, mz accuracy 5 ppm, and a multiprocess plan. mzRAPP then extracts EICs, applies RT borders, and filters peaks by peak-shape correlation with the most abundant isotopologue and isotopologue-ratio bias (< 30% deviation from predicted area/height), exporting the result as Benchmark.csv to the working directory.

## Related tools

- **mzRAPP** (R package implementing benchmark generation (get_mz_table, get_ROIs, find_bench_peaks) via the Generate Benchmark shiny tab; launched with callmzRAPP()) — https://github.com/YasinEl/mzRAPP
- **enviPat** (Underlying library for isotopologue prediction, adduct list and instrument/resolution list used by mzRAPP)
- **Skyline** (Manual peak curation tool from which user.rtmin/user.rtmax peak boundaries are exported (via 'Export Skyline Transition list and peak boundaries')) — https://skyline.ms/project/home/software/Skyline/begin.view

## Examples

```
library(mzRAPP); callmzRAPP()  # then use the 'Generate Benchmark' tab to load mzML, SampleGroups_MTBLS267.csv and Target_File_MTBLS267.csv
```

## Evaluation signals

- Reference-scale check for the MTBLS267 example: Benchmark.csv contains 47 molecules, 157 features (all adducts and isotopologues) and 2870 peaks in total, with peak heights spanning several orders of magnitude.
- Structural check: every accepted feature must have the theoretically most abundant plus at least one additional isotopologue detected; additional adducts are only kept if the main_adduct was found.
- Quality check: low isotopologues are removed when peak-shape Pearson correlation with the most abundant isotopologue < 0.85 or when area/height deviates more than 30% from the predicted isotopologue ratio.
- Parameter conformance: lowest isotopologue fraction 0.05, minimum 6 scans per peak, mz precision 6 ppm, mz accuracy 5 ppm were set as in the published example.
- Peaks rejected when isotopic information does not fit, even though user.rtmin/user.rtmax were provided.

## Limitations

- Requires centroided mzML input; vendor conversion and centroiding must be done beforehand (e.g. with ProteoWizard MSconvert).
- If the main_adduct of a molecule is not detected, no other adducts for that molecule will be accepted, so main_adduct choice (usually M+H or M-H) strongly affects benchmark coverage.
- For instruments not in the enviPat resolution list, a custom CSV (columns R and m/z, at least 10 equally distributed m/z values) must be supplied.
- Fewer than 5 mzML files is discouraged (the MTBLS267 example uses 30 *_POS.mzML files); benchmark quality grows with the number of target molecules.

## Evidence

- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [methods] Download all 30 mzML files (at least five if you do not want to process that many) ending on "_POS.mzML" from the repository MTBLS267 on [metabolights]: "Download all 30 mzML files (at least five if you do not want to process that many) ending on "_POS.mzML" from the repository MTBLS267"
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance: "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance"
- [readme] Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are considered for the final benchmark.: [evidence span withheld — anchor status: external_doc]
