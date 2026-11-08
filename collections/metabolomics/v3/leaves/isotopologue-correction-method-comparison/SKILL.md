---
name: isotopologue-correction-method-comparison
description: Use when comparing how isotope-correction schemes (MDT, ULS, NRE) perform
  on FAM/MDV data for labeled metabolites at a stated nominal instrument resolution
  (e.g. 140,000 or 280,000 Orbitrap), or when choosing MDT vs ULS for a tracer/compound
  class.
when_to_use_negative:
- Instrument resolution is extremely low — ElemCor becomes identical to IsoCor's no-resolution-effect
  (NRE) correction, so the comparison carries no information
- The compound formula lacks the tracer element — ElemCor's algorithm reports an error
  if e.g. C or S is selected as tracer but the compound has no C or S atoms
- The tracer isotope is not one of 13C, 2H, 15N, 18O, or 34S, which are the only tracer
  elements ElemCor supports
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ElemCor
  role: Applies the MDT and ULS resolution-dependent corrections to FAM data and computes
    MDVs and enrichment; serves as the method under test and improvement over the
    original ULS
  repo: github.com/4dsoftware/elemcor
- name: MATLAB
  role: Environment for the ElemCor source functions and for adapting the comparison
    into flux-analysis pipelines (FiatFlux, INCA compatible)
- name: IsoCor (NRE)
  role: Combinatoric correction without resolution effect; included as the no-resolution-effect
    (NRE) baseline method in the RMSE comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:b417c28f788f0615296e49673bc4021862b05ace5b36d66bdf00d987b4ed039a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemCor
  - IsoCor (NRE)
  - MATLAB
  - MDT
  - ULS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# isotopologue-correction-method-comparison

## Summary

Benchmark isotopologue/natural-abundance correction methods (ElemCor MDT, ElemCor ULS, original ULS, no-resolution-effect/IsoCor-style NRE, and no correction) on simulated high-resolution LC-MS isotope-labeling data, scoring each by RMSE of corrected enrichment against a known theoretical enrichment.

## When to use

Use when comparing how isotope-correction schemes (MDT, ULS, NRE) perform on FAM/MDV data for labeled metabolites at a stated nominal instrument resolution (e.g. 140,000 or 280,000 Orbitrap), or when choosing MDT vs ULS for a tracer/compound class.

## When NOT to use

- Instrument resolution is extremely low — ElemCor becomes identical to IsoCor's no-resolution-effect (NRE) correction, so the comparison carries no information
- The compound formula lacks the tracer element — ElemCor's algorithm reports an error if e.g. C or S is selected as tracer but the compound has no C or S atoms
- The tracer isotope is not one of 13C, 2H, 15N, 18O, or 34S, which are the only tracer elements ElemCor supports

## Inputs

- Simulated or measured FAM XLSX files for labeled samples (e.g. test_labelN_sim.xlsx, test_labelS_sim.xlsx), one column vector of fractional abundances per compound-sample
- Unlabeled-sample FAM XLSX files for ULS correction (e.g. test_unlabelN_sim.xlsx, test_unlabelS_sim.xlsx)
- Nominal instrument resolution and mass analyzer type (e.g. Orbitrap, 140,000 or 280,000)
- Isotopic purity of the nutrient tracer and tracer element selection (13C, 2H, 15N, 18O, or 34S)

## Outputs

- Corrected MDVs and isotopic enrichment values per compound and sample (written into new sheets of the input XLSX, along with fold change of pool size)
- RMSE table comparing corrected enrichment against theoretical enrichment for each compound-tracer-method combination (e.g. rmse_table.csv)

## How to apply

Generate or load simulated fractional abundances of measured isotopologues (FAM) for the target compounds and tracer elements (13C, 15N, 34S supported; also 2H and 18O) at the instrument's nominal resolution, formatted as XLSX with one column vector per sample (FAM length typically Nt+1). Apply each correction method: ElemCor's MDT (resolution-dependent correction matrix from mass difference theory), ElemCor's ULS (deconvolution of unlabeled-sample FAM Nt times to build the natural-abundance correction matrix), the original ULS implementation, no-resolution-effect correction (NRE as in IsoCor), and no correction. Each method yields a mass distribution vector (MDV) via non-negative linear regression of the correction matrix followed by normalization to sum 1, from which isotopic enrichment is computed. Compute RMSE of each corrected enrichment against the known theoretical enrichment (e.g. 20%) per compound-tracer-method combination, and tabulate the results (e.g. rmse_table.csv). Judgment criteria follow the paper: for high-mass CoA metabolites at 280,000 resolution, ElemCor (MDT/ULS) should markedly outperform the original ULS for 13C and 34S, while the original ULS remains excellent for 15N; NRE (IsoCor-style) should be the least accurate.

## Related tools

- **ElemCor** (Applies the MDT and ULS resolution-dependent corrections to FAM data and computes MDVs and enrichment; serves as the method under test and improvement over the original ULS) — github.com/4dsoftware/elemcor
- **MATLAB** (Environment for the ElemCor source functions and for adapting the comparison into flux-analysis pipelines (FiatFlux, INCA compatible))
- **IsoCor (NRE)** (Combinatoric correction without resolution effect; included as the no-resolution-effect (NRE) baseline method in the RMSE comparison)

## Examples

```
ElemCor.exe  # Step 1: load test_labelS_sim.xlsx; Step 3: set purity + resolution 280000; Step 4: tracer=34S; Step 5: analyzer=Orbitrap; Step 6: run
```

## Evaluation signals

- RMSE of corrected enrichment vs the 20% theoretical enrichment should be near zero for ElemCor MDT/ULS on 13C and 34S CoA metabolites (coenzyme A, acetyl-CoA, succinyl-CoA, HMG-CoA) at resolution 280,000, while original ULS shows markedly higher RMSE for those tracers
- For 15N tracer data, the original ULS method should remain accurate (excellent RMSE), comparable to ElemCor
- NRE (IsoCor-style) correction should yield the highest RMSE, since it is significantly less accurate than the resolution-dependent methods
- Corrected MDVs must be normalized to sum to 1 and should appear in a new sheet of the input XLSX alongside enrichment and pool-size fold change
- Example files should process within about 10 seconds on a 2.6 GHz Core i7 (20 seconds on a 1.5 GHz Core i5); failures within a minute usually indicate the wrong tracer element was selected

## Limitations

- ULS requires additional experimental measurement of unlabeled samples, adding cost; MDT is recommended for low-mass compounds for cost efficiency
- At extremely high instrument resolution ElemCor performs no correction at all, and at extremely low resolution it is identical to IsoCor — the correction magnitude is monotonically dependent on resolution
- ElemCor was developed under MATLAB (2016b); running the compiled executable needs MATLAB Compiler Runtime but not MATLAB, and incorporating the source into other pipelines requires collaboration with the authors to share MATLAB functions
- Experimental validation was limited to a single organism (S. cerevisiae) and a single tracer (15N ammonium at 20.0%)
- The current version only prints a general error message; wrong tracer selection or mismatched labeled/unlabeled files (same analytes, same number of rows required) cause silent failures

## Evidence

- [results] ElemCor MDT/ULS outperform original ULS on 13C/34S CoA metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] NRE (IsoCor-style) correction is least accurate: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [abstract] MDT vs ULS usage guidance: "We recommend MDT for low-mass compounds for cost efficiency in experiments, and ULS for high-mass compounds with relatively large spectral inaccuracy that can be tracked by unlabeled standards."
- [methods] ElemCor workflow step 6 computes enrichment: "Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6."
- [readme] Resolution dependence of correction magnitude: "the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [readme] Test data format and resolutions: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
