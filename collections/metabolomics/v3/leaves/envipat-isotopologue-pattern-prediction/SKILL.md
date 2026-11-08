---
name: envipat-isotopologue-pattern-prediction
description: 'Use when building an LC-HRMS benchmark from target molecules with known
  formulas and RT boundaries: isotopologue patterns must be predicted from SumForm_c
  entries before chromatographic ROI extraction from centroided mzML files.'
when_to_use_negative:
- Input molecules lack a defined neutral molecular composition or a main adduct —
  enviPat prediction requires SumForm_c and main_adduct per molecule
- Only profile-mode data is available — mzRAPP requires centroided mzML files, so
  prediction would feed a benchmark that cannot be extracted
- The goal is generic isotope fine-structure calculation outside mzRAPP's benchmark
  workflow with RT boundaries — a standalone enviPat session is simpler
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: enviPat
  role: Predicts isotopologue patterns from molecular formulas for user-selected adducts
    and instrument resolution; supplies the adducts and resolution lists used by get_mz_table
- name: mzRAPP
  role: Wraps enviPat in the get_mz_table function, the first stage of the benchmark
    generation cascade; GUI accessible via callmzRAPP()
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Implementation language; installed via devtools::install_github('YasinEl/mzRAPP')
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:f5e37e6f1c1bff39031756940725a9ae2c72a9fefd35b2ad4c3328c7434fa296
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MSnbase
  - R
  - enviPat
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# envipat-isotopologue-pattern-prediction

## Summary

Predicts theoretical isotopologue patterns for target molecules using the enviPat R package, based on molecular composition and instrument resolution, as the first step of mzRAPP's benchmark generation cascade (get_mz_table).

## When to use

Use when building an LC-HRMS benchmark from target molecules with known formulas and RT boundaries: isotopologue patterns must be predicted from SumForm_c entries before chromatographic ROI extraction from centroided mzML files.

## When NOT to use

- Input molecules lack a defined neutral molecular composition or a main adduct — enviPat prediction requires SumForm_c and main_adduct per molecule
- Only profile-mode data is available — mzRAPP requires centroided mzML files, so prediction would feed a benchmark that cannot be extracted
- The goal is generic isotope fine-structure calculation outside mzRAPP's benchmark workflow with RT boundaries — a standalone enviPat session is simpler

## Inputs

- Target-file csv (molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax; optional adduct_c, StartTime.EIC, EndTime.EIC, FileName columns)
- Instrument/resolution selection or custom resolution csv (columns: R, m/z)
- Additional adduct selection (enviPat adduct names, e.g. M+NH4, M+Na, M+K)

## Outputs

- data.table of predicted isotopologue patterns per molecule/adduct (input to get_ROIs)
- Lowest-isotopologue-filtered pattern list (threshold >= 0.05 relative abundance)

## How to apply

Provide a target-file csv with molecule names, neutral molecular compositions (SumForm_c, e.g. C10H15N5O10P2, never with a 0 behind an element), a main_adduct per molecule, and user.rtmin/user.rtmax in seconds. Select the instrument resolution from enviPat's resolution list (e.g. OrbitrapXL/Velos R60000@400) or upload a custom two-column csv (R at half height, m/z) with at least 10 equally distributed m/z values. Run mzRAPP's get_mz_table, which calls enviPat to predict isotopologues for enabled adducts (e.g. M+NH4, M+Na, M+K in addition to M+H), setting the lowest relative isotopologue abundance to be considered to >= 0.05. The function returns a data.table of predicted isotopologue patterns that is fed directly to get_ROIs for chromatogram extraction, and later find_bench_peaks, which requires the theoretically most abundant plus at least one additional isotopologue to be detected and removes peaks >30% off predicted area/height or with Pearson correlation < 0.85 to the highest isotopologue.

## Related tools

- **enviPat** (Predicts isotopologue patterns from molecular formulas for user-selected adducts and instrument resolution; supplies the adducts and resolution lists used by get_mz_table)
- **mzRAPP** (Wraps enviPat in the get_mz_table function, the first stage of the benchmark generation cascade; GUI accessible via callmzRAPP()) — https://github.com/YasinEl/mzRAPP
- **R** (Implementation language; installed via devtools::install_github('YasinEl/mzRAPP')) — https://cran.r-project.org/

## Examples

```
library(mzRAPP); bench <- get_mz_table(target_file = "Target_File_MTBLS267.csv", sample_groups = "SampleGroups_MTBLS267.csv", instrument = "OrbitrapXL/Velos", resolution = "R60000@400", adducts = c("M+NH4", "M+Na", "M+K"), lowest_iso = 0.05)
```

## Evaluation signals

- Benchmark built from all 30 MTBLS267 mzML files contains 47 different molecules, 157 features (all adducts and isotopologues), and 2870 peaks in total, matching the reference Benchmark.csv on ucloud
- Every retained molecule has the theoretically most abundant isotopologue plus at least one additional isotopologue detected
- No retained isotopologue has area/height more than 30% off the predicted value or Pearson correlation < 0.85 with the most abundant isotopologue
- get_mz_table returns a data.table object that is accepted as the argument of get_ROIs without schema errors
- Only isotopologues with relative abundance >= 0.05 appear in the output

## Limitations

- If the main_adduct is not detected, other adducts will not be accepted; adducts are only added if at least two isotopologues of the respective adduct can be detected
- Isotopologue peaks with an area or height more than 30% off the predicted value or Pearson correlation < 0.85 (vs. the highest isotopologue) are removed, so weak or distorted peaks are lost
- Peaks with provided user.rtmin/user.rtmax are still rejected if the isotopic information does not fit
- The lowest isotopologue to be considered cannot be below 0.05 relative abundance

## Evidence

- [methods] get_mz_table (IT pattern prediction): [evidence span withheld — anchor status: not_found]
- [readme] enviPat adducts allowed: [evidence span withheld — anchor status: not_found]
- [readme] isotopologue filtering thresholds: [evidence span withheld — anchor status: not_found]
- [readme] resolution requirement: [evidence span withheld — anchor status: external_doc]
- [methods] validation counts: [evidence span withheld — anchor status: not_found]
