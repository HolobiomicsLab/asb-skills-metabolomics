---
name: targeted-untargeted-metabolomics-merging
description: Use when you have centroided LC-HRMS mzML files plus a set of molecules
  with known molecular composition (SumForm_c), adducts, and RT boundaries, and need
  to benchmark untargeted pre-processing outputs (XCMS, MZmine 2, MS-DIAL) against
  them.
when_to_use_negative:
- Input is already a pre-processed feature table from XCMS/MZmine — that belongs to
  the reliability-assessment/matching step, not benchmark generation
- Only raw vendor files are available and have not been converted to centroided mzML
  via MSconvert
- No target molecules with known molecular composition and RT boundaries are available
  (mzRAPP requires user-supplied targets)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzRAPP
  role: 'Primary tool: cascades get_mz_table, get_ROIs and find_bench_peaks to generate
    and validate the benchmark from user-supplied targets and mzML files'
  repo: https://github.com/YasinEl/mzRAPP
- name: enviPat
  role: Predicts isotopologue patterns and provides the adduct and instrument-resolution
    lists used for isotope prediction
- name: Skyline
  role: Manual peak curation tool from which peak RT boundaries can be exported for
    the target file
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: R
  role: Implementation language; mzRAPP is installed via devtools::install_github
    and launched with callmzRAPP()
  repo: https://github.com/YasinEl/mzRAPP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:6d551df8955be3a8ec575a6c9bfc1bbb55f1c40abe905d5943dc22658c6428d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - R
  - Skyline
  - enviPat
  - mzRAPP
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# targeted-untargeted-metabolomics-merging

## Summary

Generates a validated benchmark of chromatographic peaks for target molecules with known retention behavior from user-supplied mzML files, exploiting the merging of targeted and non-targeted metabolomics to then score non-targeted data pre-processing (NPP) reliability.

## When to use

Use when you have centroided LC-HRMS mzML files plus a set of molecules with known molecular composition (SumForm_c), adducts, and RT boundaries, and need to benchmark untargeted pre-processing outputs (XCMS, MZmine 2, MS-DIAL) against them.

## When NOT to use

- Input is already a pre-processed feature table from XCMS/MZmine — that belongs to the reliability-assessment/matching step, not benchmark generation
- Only raw vendor files are available and have not been converted to centroided mzML via MSconvert
- No target molecules with known molecular composition and RT boundaries are available (mzRAPP requires user-supplied targets)

## Inputs

- Centroided mzML files (e.g. 30 MTBLS267 *_POS.mzML files from MetaboLights)
- Sample-group CSV with sample_name and sample_group columns
- Target file CSV with molecule, SumForm_c (e.g. C10H15N5O10P2), main_adduct (e.g. M+H), user.rtmin and user.rtmax in seconds
- Instrument/resolution selection (enviPat resolution list, e.g. OrbitrapXL,Velos,VelosPro_R60000@400) or custom two-column R/mz resolution CSV
- Adduct selection (enviPat adducts such as M+NH4, M+Na, M+K)

## Outputs

- Benchmark.csv — validated benchmark of 47 molecules / 157 features / 2870 peaks (MTBLS267 reference) including all adducts and isotopologues
- Skyline transition list and peak boundaries export for manual RT-border curation

## How to apply

Provide centroided mzML files (converted/centroided via ProteoWizard MSconvert), a sample-group CSV (sample_name, sample_group), and a target CSV with molecule, SumForm_c, main_adduct, user.rtmin/user.rtmax (seconds), and optional adduct_c, StartTime.EIC/EndTime.EIC and FileName columns. In mzRAPP's Generate Benchmark tab (or via R script), select the instrument/resolution (e.g. OrbitrapXL,Velos,VelosPro_R60000@400), additional adducts (e.g. M+NH4, M+Na, M+K — all enviPat adducts are allowed), and set parameters: lowest isotopologue fraction 0.05, min. 6 scans per peak, mz precision 6 ppm, mz accuracy 5 ppm. mzRAPP cascades get_mz_table, get_ROIs, and find_bench_peaks to predict isotopologues with enviPat, extract EICs/ROIs, and pick peaks; only isotopologues where the most abundant plus at least one additional isotopologue are found are kept, and peaks with area/height >30% off the predicted isotopologue-ratio value or Pearson correlation <0.85 with the most abundant isotopologue are removed. Export the resulting benchmark (e.g. Benchmark.csv) and optionally an 'Export Skyline Transition list and peak boundaries' file for manual curation of RT borders.

## Related tools

- **mzRAPP** (Primary tool: cascades get_mz_table, get_ROIs and find_bench_peaks to generate and validate the benchmark from user-supplied targets and mzML files) — https://github.com/YasinEl/mzRAPP
- **enviPat** (Predicts isotopologue patterns and provides the adduct and instrument-resolution lists used for isotope prediction)
- **Skyline** (Manual peak curation tool from which peak RT boundaries can be exported for the target file) — https://skyline.ms/project/home/software/Skyline/begin.view
- **R** (Implementation language; mzRAPP is installed via devtools::install_github and launched with callmzRAPP()) — https://github.com/YasinEl/mzRAPP

## Examples

```
library(mzRAPP); callmzRAPP()  # open the Generate Benchmark tab, or use devtools::install_github("YasinEl/mzRAPP", dependencies = TRUE, build_vignettes = TRUE) to install
```

## Evaluation signals

- Reference MTBLS267 benchmark contains 47 molecules, 157 features (including all adducts and isotopologues) and 2870 peaks in total
- Peak heights span several orders of magnitude; low isotopologues are filtered by peak-shape correlation and isotopologue-ratio bias < 30%
- Only isotopologues with the most abundant plus at least one additional isotopologue detected are retained in the final benchmark
- Isotopologue peaks with Pearson correlation < 0.85 versus the highest isotopologue must have been removed
- Every molecule must be represented by at least one row whose adduct_c equals its selected main_adduct in the target file

## Limitations

- If the main_adduct is not detected, other adducts of that molecule will not be accepted, so the most trusted adduct (generally M+H or M-H) should be chosen as main adduct
- Peaks with provided user.rtmin/user.rtmax are still rejected if the isotopic information does not fit
- Files must be centroided mzML; vendor conversion and centroiding must be done beforehand (e.g. via MSconvert)
- An adduct is only added if at least two isotopologues of the respective adduct can be detected

## Evidence

- [readme] mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files.: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] Isotopologue peaks with an area or height which is more than 30% off the predicted value or a Pearson Correlation coef < 0.85 (as compared to the highest isotopologue) are removed.: [evidence span withheld — anchor status: not_found]
- [readme] In order to generate a benchmark you need to provide your centroided mzML files. Conversion of files of different vendors to mzML as well as centroiding can be done by Proteowizards MSconvert: [evidence span withheld — anchor status: external_doc]
- [methods] Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue: "Sufficient quality of low peaks was ensured by removing isotopologues that do not satisfy criteria in peak shape (peak shape correlation with most abundant isotopologue) and abundance (Isotopologue"
- [readme] mzRAPP generates extracted ion chromatograms for all isotopologues predictable for those molecules and applies provided RT boundaries.: [evidence span withheld — anchor status: external_doc]
