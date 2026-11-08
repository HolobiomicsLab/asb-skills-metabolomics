---
name: benchmark-size-validation
description: Use when an mzRAPP 'Generate Benchmark' run has just produced a Benchmark.csv
  and you must confirm it matches the reference benchmark size and passed the isotopologue
  quality filtering before running NPP reliability assessment.
when_to_use_negative:
- You already downloaded the pre-prepared Benchmark.csv from the ucloud deposit (https://ucloud.univie.ac.at/index.php/s/nu9FLARAa8XXtJC)
  — it is already validated; skip to reliability assessment.
- Your goal is assessing NPP output quality (split peaks, alignment errors) rather
  than checking the benchmark itself — use the reliability-assessment metrics workflow
  instead.
- Your dataset is not an mzRAPP benchmark with enviPat-predicted isotopologues — feature/peak
  count expectations like 157 features/2870 peaks are specific to the MTBLS267 Orbitrap
  setup.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mzRAPP
  role: Generates and exports Benchmark.csv via the Generate Benchmark tab (get_mz_table,
    get_ROIs, find_bench_peaks cascade) whose size and filtering are then validated
  repo: https://github.com/YasinEl/mzRAPP
- name: R
  role: Runtime environment; mzRAPP is implemented in R and launched with library(mzRAPP);
    callmzRAPP()
  repo: https://github.com/YasinEl/mzRAPP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:65d7714ee69205f551a67b15cd681d4f1ea536ceb519b8135b1bf7807ae0a31d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# benchmark-size-validation

## Summary

Validates an mzRAPP-generated benchmark dataset (Benchmark.csv) against the reference scale and quality filters of the MTBLS267 example: 47 molecules yielding 157 features across M+NH4, M+Na and M+K adducts and 2870 total peaks, with low-intensity isotopologues removed by peak-shape correlation and isotopologue-ratio-bias (<30%) criteria.

## When to use

Use when an mzRAPP 'Generate Benchmark' run has just produced a Benchmark.csv and you must confirm it matches the reference benchmark size and passed the isotopologue quality filtering before running NPP reliability assessment.

## When NOT to use

- You already downloaded the pre-prepared Benchmark.csv from the ucloud deposit (https://ucloud.univie.ac.at/index.php/s/nu9FLARAa8XXtJC) — it is already validated; skip to reliability assessment.
- Your goal is assessing NPP output quality (split peaks, alignment errors) rather than checking the benchmark itself — use the reliability-assessment metrics workflow instead.
- Your dataset is not an mzRAPP benchmark with enviPat-predicted isotopologues — feature/peak count expectations like 157 features/2870 peaks are specific to the MTBLS267 Orbitrap setup.

## Inputs

- Benchmark.csv (mzRAPP benchmark export: molecule, adduct, isotopologue, mz, RT boundaries, per-file peak areas/heights)
- enviPat-predicted isotopologue tables (from the target file with SumForm_c, main_adduct, user.rtmin/user.rtmax)
- 30 centroided MTBLS267 _POS.mzML files
- SampleGroups_MTBLS267.csv and Target_File_MTBLS267.csv

## Outputs

- Validation report: molecule count (expected 47), feature count (expected 157), peak count (expected 2870)
- Confirmation that isotopologue-ratio bias < 30% and peak-shape correlation >= 0.85 filters were applied to low-intensity peaks

## How to apply

After running the benchmark cascade (get_mz_table, get_ROIs, find_bench_peaks) on the 30 MTBLS267 _POS.mzML files with OrbitrapXL,Velos,VelosPro_R60000@400 resolution, lowest isotopologue fraction 0.05, min. 6 scans per peak, mz precision 6 ppm and mz accuracy 5 ppm, open the exported Benchmark.csv and count distinct molecules, features (molecule/adduct/isotopologue combinations) and peaks. The reference run reports 47 molecules, 157 features and 2870 peaks with peak heights spanning several orders of magnitude. Verify that filtering removed poor-quality low peaks: isotopologues failing peak-shape correlation with the most abundant isotopologue (Pearson r < 0.85) or with area/height more than 30% off the enviPat-predicted isotopologue ratio are removed, and only molecules whose theoretically most abundant isotopologue plus at least one additional isotopologue are found enter the benchmark. Divergence in feature/peak counts signals mis-set parameters (e.g. wrong adducts or too-lenient mz accuracy) rather than a tool failure.

## Related tools

- **mzRAPP** (Generates and exports Benchmark.csv via the Generate Benchmark tab (get_mz_table, get_ROIs, find_bench_peaks cascade) whose size and filtering are then validated) — https://github.com/YasinEl/mzRAPP
- **R** (Runtime environment; mzRAPP is implemented in R and launched with library(mzRAPP); callmzRAPP()) — https://github.com/YasinEl/mzRAPP

## Examples

```
library(mzRAPP); callmzRAPP()  # open Generate Benchmark tab, run extraction, then inspect exported Benchmark.csv
```

## Evaluation signals

- Benchmark.csv contains exactly 47 molecules, 157 features (including all adducts and isotopologues) and 2870 peaks as in the reference run
- Peak heights in the benchmark span several orders of magnitude, consistent with low isotopologues surviving the 0.05 relative-abundance threshold
- No retained isotopologue deviates more than 30% from its enviPat-predicted area/height, and no retained peak has Pearson correlation < 0.85 with the most abundant isotopologue
- Every benchmarked molecule has its theoretically most abundant isotopologue plus at least one additional isotopologue detected
- Every feature satisfies the configured quality gates: >= 6 scans per peak, mz precision 6 ppm, mz accuracy 5 ppm

## Limitations

- The 47/157/2870 counts are specific to the MTBLS267 example with the M+NH4, M+Na, M+K adduct selection on an Orbitrap Velos R60000@400 setup; other datasets will have different expected sizes.
- Peaks whose user.rtmin/user.rtmax are provided are still rejected if the isotopic information does not fit, so counts can be lower than expected even with correct parameters.
- Additional adducts are only added if at least two isotopologues of that adduct are detected, so low-abundance adducts may be silently absent from the benchmark.

## Evidence

- [readme] Only isotopologues for which the theoretically most abundant and at least one additional isotopologue are found are considered for the final benchmark.: [evidence span withheld — anchor status: external_doc]
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue: "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue"
- [other] Benchmark generation in mzRAPP is implemented as a cascade of three functions (get_mz_table, get_ROIs, find_bench_peaks): [evidence span withheld — anchor status: not_found]
- [other] check Benchmark.csv against the reported reference — 47 molecules, 157 features (including all adducts and isotopologues), and 2870 peaks in total: [evidence span withheld — anchor status: not_found]
