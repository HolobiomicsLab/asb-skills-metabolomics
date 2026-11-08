---
name: unlabeled-sample-correction-matrix-estimation
description: Use when correcting LC-MS isotope labeling data on high-resolving-power
  instruments (e.g. Orbitrap) where an unlabeled sample standard for each compound
  is available and the correction matrix must reflect the instrument's nominal resolution.
when_to_use_negative:
- When no unlabeled-sample measurements are available — without Step 2 data, ElemCor
  runs on mass difference theory (MDT) only, so use MDT instead
- For low-mass compounds where MDT is recommended for cost efficiency, since ULS requires
  additional experimental measurement of unlabeled samples
- When the tracer element is not present in some compounds' formulas — the algorithm
  reports an error if the wrong tracer is selected (e.g. loading 15N data but selecting
  C or S)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Software that implements the unlabeled-sample (ULS) correction via deconvolution
    of unlabeled-sample FAM and computes corrected MDVs and enrichment
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Development/run environment; MATLAB functions of ElemCor can be adapted into
    flux analysis pipelines such as FiatFlux and INCA
  repo: https://github.com/4dsoftware/elemcor
- name: FiatFlux
  role: MATLAB-based flux analysis suite into which the provided ElemCor MATLAB function
    can be adapted
- name: INCA
  role: MATLAB-based flux analysis suite into which the provided ElemCor MATLAB function
    can be adapted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:c5556011f86f34a797d4a67c1d2890d7d1eb34a4e16daf7a686605ee541c4306
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - FiatFlux
  - INCA
  - IsoCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# unlabeled-sample-correction-matrix-estimation

## Summary

Estimate the natural-abundance/tracer-impurity correction matrix by deconvolving the fractional abundances of measured isotopologs (FAM) from unlabeled samples, as implemented in ElemCor's ULS method for high-resolution LC-MS stable isotope labeling data.

## When to use

Use when correcting LC-MS isotope labeling data on high-resolving-power instruments (e.g. Orbitrap) where an unlabeled sample standard for each compound is available and the correction matrix must reflect the instrument's nominal resolution.

## When NOT to use

- When no unlabeled-sample measurements are available — without Step 2 data, ElemCor runs on mass difference theory (MDT) only, so use MDT instead
- For low-mass compounds where MDT is recommended for cost efficiency, since ULS requires additional experimental measurement of unlabeled samples
- When the tracer element is not present in some compounds' formulas — the algorithm reports an error if the wrong tracer is selected (e.g. loading 15N data but selecting C or S)

## Inputs

- FAM column vectors from labeled samples in XLSX format (one column vector per compound per sample, e.g. test_labelS_sim.xlsx)
- FAM column vectors from unlabeled samples in XLSX format (e.g. test_unlabelS_sim.xlsx)
- Isotopic purity of the labeled nutrient
- Nominal instrument resolution (e.g. 140,000 or 280,000) and mass analyzer type (e.g. Orbitrap)
- Tracer element selection (13C, 2H, 15N, 18O, or 34S)

## Outputs

- Correction matrix for natural abundance and isotopic impurity
- Corrected mass distribution vectors (MDV) stored in a new sheet of the input XLSX file
- Isotopic enrichment per compound and sample
- Fold change of pool size per compound and sample

## How to apply

Measure FAM vectors from unlabeled samples for the same compounds, then deconvolve the fractional natural abundance of the tracer element Nt times from the unlabeled-sample FAM to obtain the correction matrix for natural abundance. Combine this with the resolution-dependent MDT component: in ElemCor, load labeled and unlabeled XLSX data (one FAM column vector per compound per sample, e.g. test_labelS_sim.xlsx and test_unlabelS_sim.xlsx), specify isotopic purity of the nutrient and nominal instrument resolution (e.g. 140,000 for the Orbitrap), select the tracer element (13C, 2H, 15N, 18O, or 34S), and run analysis. Apply non-negative linear regression of the correction-matrix equation followed by normalization to the sum of 1 to obtain MDVs and isotopic enrichment. ULS is preferred for high-mass compounds with relatively large spectral inaccuracy that can be tracked by unlabeled standards, whereas the original ULS implementation underperformed for 34S because 32S natural abundance (95.0%) is lower than 14N's (99.6%).

## Related tools

- **ElemCor** (Software that implements the unlabeled-sample (ULS) correction via deconvolution of unlabeled-sample FAM and computes corrected MDVs and enrichment) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Development/run environment; MATLAB functions of ElemCor can be adapted into flux analysis pipelines such as FiatFlux and INCA) — https://github.com/4dsoftware/elemcor
- **FiatFlux** (MATLAB-based flux analysis suite into which the provided ElemCor MATLAB function can be adapted)
- **INCA** (MATLAB-based flux analysis suite into which the provided ElemCor MATLAB function can be adapted)

## Evaluation signals

- Corrected MDVs sum to 1 after normalization (non-negative linear regression followed by normalization to the sum of 1)
- MDV, enrichment, and fold-change results appear in new sheets of the original input XLSX file
- For 34S simulations at resolution 140,000, the modified ULS enrichment RMSE is lower than the NRE (IsoCor-style) RMSE, while the original ULS RMSE exceeds the NRE RMSE
- Processing the bundled example files completes within ~10 seconds on a 2.6GHz Core i7 (or ~20 seconds on a 1.5GHz Core i5); longer runtime suggests the wrong tracer was selected
- Labeled and unlabeled XLSX files contain the same analytes and the same number of rows

## Limitations

- ULS requires additional experimental measurement of unlabeled samples, adding cost
- ElemCor was developed under MATLAB (2016b) and requires MATLAB Compiler Runtime; dependency on the proprietary MATLAB environment may limit reproducibility
- No software version/release identifier or license terms are stated in the source record (GitHub repo lists MIT)
- If the wrong tracer element is selected (e.g. 15N data loaded but C or S selected), the software will not run properly because the algorithm errors when the tracer is absent from a compound
- Original ULS is less accurate than NRE for 34S tracers because sulfur's most abundant isotope has lower fractional abundance (95.0% 32S vs 99.6% 14N)

## Evidence

- [discussion] ULS deconvolution rationale: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [discussion] MDV computation: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] Tracer element options and workflow: "In Step 4, the tracer element is selected. In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [readme] Input format: "The files show standard format of input files. Each column vector correspond to FAM of all compounds in each sample."
- [results] 34S abundance rationale: [evidence span withheld — anchor status: not_found]
