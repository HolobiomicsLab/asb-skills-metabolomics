---
name: unlabeled-sample-based-correction
description: Use when correcting LC-MS isotope-labeling FAM data from a high-resolution
  analyzer (e.g., Orbitrap at 140,000/280,000 nominal resolution) and an unlabeled-sample
  spectrum is available, especially for high-mass metabolites where resolution effects
  are strong.
when_to_use_negative:
- No unlabeled-sample measurement is available — ULS requires an additional experimental
  measurement; use the MDT-only path instead (ElemCor runs on mass difference theory
  when unlabeled data are not loaded).
- Small/low-mass metabolites — the article recommends MDT for low-mass compounds for
  cost efficiency, reserving ULS for high-mass compounds.
- The tracer element selected is not present in some compounds' formulas (e.g., selecting
  C or S as tracer for 15N data), which makes the algorithm report errors.
tools:
- name: ElemCor
  role: Implements the corrected ULS deconvolution scheme with a GUI/MATLAB function;
    loads labeled and unlabeled XLSX FAM files, applies resolution-dependent correction,
    and writes MDV, enrichment, and pool-size fold change back into the input file.
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Development/runtime environment (MATLAB 2016b + MATLAB Compiler Runtime);
    the provided MATLAB function can be adapted into flux analysis pipelines such
    as FiatFlux and INCA.
- name: FluxFix
  role: Prior unlabeled-sample-based correction method whose numerical scheme ElemCor
    corrects and improves.
- name: IsoCor
  role: Combinatorial correction baseline that ignores the resolution effect (NRE);
    used as the comparison point showing why ULS with resolution awareness is needed.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:f89c71e06e27e1c4c710b75bd6ff6e19184cbb84519fe307ea856322159d5752
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - FluxFix
  - IsoCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# unlabeled-sample-based-correction

## Summary

The ULS correction method (as improved in ElemCor) builds the natural-abundance correction matrix by deconvolving the fractional abundance of measured isotopologs (FAM) of an unlabeled sample for the tracer element Nt times, then applies non-negative linear regression to recover the mass distribution vector (MDV) and isotopic enrichment at a specified instrument resolution.

## When to use

Use when correcting LC-MS isotope-labeling FAM data from a high-resolution analyzer (e.g., Orbitrap at 140,000/280,000 nominal resolution) and an unlabeled-sample spectrum is available, especially for high-mass metabolites where resolution effects are strong.

## When NOT to use

- No unlabeled-sample measurement is available — ULS requires an additional experimental measurement; use the MDT-only path instead (ElemCor runs on mass difference theory when unlabeled data are not loaded).
- Small/low-mass metabolites — the article recommends MDT for low-mass compounds for cost efficiency, reserving ULS for high-mass compounds.
- The tracer element selected is not present in some compounds' formulas (e.g., selecting C or S as tracer for 15N data), which makes the algorithm report errors.

## Inputs

- XLSX file of FAM column vectors from labeled samples (e.g., test_labelN_sim.xlsx, one column per compound per sample)
- XLSX file of FAM column vectors from unlabeled samples (e.g., test_unlabelN_sim.xlsx) required for ULS deconvolution
- Isotopic purity of the nutrient (e.g., 20.0% 15N ammonium tracer purity specification)
- Nominal instrument resolution and mass analyzer (e.g., Orbitrap at 140,000)
- Tracer element selection (15N, 13C, 2H, 18O, or 34S)

## Outputs

- Corrected mass distribution vectors (MDV) per compound per sample, stored in a new sheet of the input XLSX file
- Isotopic enrichment per compound per sample (also stored in the XLSX)
- Fold change of pool size per compound per sample
- Per-metabolite, per-method accuracy metrics such as RMSE versus theoretical enrichment (e.g., rmse_15n_correction_methods.csv)

## How to apply

Load the labeled and unlabeled data as XLSX column vectors (one per compound per sample, FAM of length Nt+1 covering M+0 through M+Nt; ElemCor can also handle other lengths). Obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the unlabeled-sample FAM, then combine with tracer isotopic impurity and resolution effects. Solve for the MDV via non-negative linear regression of the correction matrix equation followed by normalization to a sum of 1, and compute isotopic enrichment per compound. Specify the isotopic purity of the nutrient and the nominal instrument resolution (e.g., 140,000 for the 15N Orbitrap test files) and select the correct tracer element (15N, 13C, 2H, 18O, or 34S) before running, since compounds lacking the tracer element cause algorithm errors. In validation, ULS-based enrichment should show lower RMSE from the theoretical enrichment than raw FAM, the no-resolution-effect (IsoCor-style combinatorial) correction, and the original ULS implementation.

## Related tools

- **ElemCor** (Implements the corrected ULS deconvolution scheme with a GUI/MATLAB function; loads labeled and unlabeled XLSX FAM files, applies resolution-dependent correction, and writes MDV, enrichment, and pool-size fold change back into the input file.) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Development/runtime environment (MATLAB 2016b + MATLAB Compiler Runtime); the provided MATLAB function can be adapted into flux analysis pipelines such as FiatFlux and INCA.)
- **FluxFix** (Prior unlabeled-sample-based correction method whose numerical scheme ElemCor corrects and improves.)
- **IsoCor** (Combinatorial correction baseline that ignores the resolution effect (NRE); used as the comparison point showing why ULS with resolution awareness is needed.)

## Evaluation signals

- Corrected MDV rows sum to 1 after non-negative linear regression normalization.
- Isotopic enrichment of simulated 15N metabolites at 20% theoretical enrichment falls closer to 20% than raw FAM, NRE (IsoCor-style), and the original ULS implementation — significantly lower RMSE.
- ElemCor's correction strength is monotonically dependent on instrument resolution: identical to IsoCor at extremely low resolution and performs no correction at extremely high resolution — results should behave this way when resolution is varied.
- Output MDVs, enrichment, and fold-change appear as new sheets inside the original input XLSX for every compound and sample.
- Labeled and unlabeled files have the same analytes and the same number of rows; the selected tracer element is present in all compound formulas.

## Limitations

- ULS requires an additional experimental measurement of unlabeled samples, which the article cites as a cost limitation; MDT is recommended for small metabolites.
- ElemCor was developed under MATLAB (2016b) and requires MATLAB Compiler Runtime to run the packaged executable, which may limit reproducibility.
- Experimental validation was limited to a single organism (S. cerevisiae) and a single tracer (15N ammonium at 20.0% enrichment).
- If the wrong tracer element is selected (e.g., 15N data loaded but C or S selected), the software will not run properly because some compounds lack that element in their formulas.
- The current version only prints a general error message, complicating troubleshooting.

## Evidence

- [discussion] ULS deconvolution basis: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [discussion] Non-negative regression: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [abstract] ULS use case: "We recommend MDT for low-mass compounds for cost efficiency in experiments, and ULS for high-mass compounds with relatively large spectral inaccuracy that can be tracked by unlabeled standards."
- [readme] Test data files: "The package includes example data files from labeled samples "test_labelN_sim.xlsx" and "test_labelS_sim.xlsx", and the associated files from unlabeled samples are "test_unlabelN_sim.xlsx""
- [readme] Resolution dependence: "Therefore, the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [discussion] IsoCor comparison: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [readme] Tracer selection risk: "If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly."
