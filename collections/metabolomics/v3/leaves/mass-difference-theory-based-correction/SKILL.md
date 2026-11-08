---
name: mass-difference-theory-based-correction
description: Use when correcting FAM/MDV from high-resolution LC-MS stable isotope
  labeling data (e.g., 13C, 2H, 15N, 18O, 34S tracers on an Orbitrap at nominal resolutions
  like 140,000 or 280,000) to compute isotopic enrichment.
when_to_use_negative:
- The tracer element is not present in a compound's formula — the algorithm will report
  an error (e.g., selecting C or S as tracer on 15N data)
- Instrument resolution is extremely low, where ElemCor reduces to IsoCor's NRE combinatorial
  correction and resolution-aware MDT offers no advantage
- The compound is high-mass with relatively large spectral inaccuracy, where the ULS
  correction (using unlabeled samples) is recommended over MDT
tools:
- name: ElemCor
  role: Software tool implementing the MDT correction matrix, non-negative linear
    regression, and enrichment calculation for high-resolution LC-MS labeling data
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Combinatorial correction baseline (NRE) that ignores resolution effects; used
    as comparison showing MDT is significantly more accurate on high-resolution data
- name: MATLAB
  role: Development environment; ElemCor's MATLAB functions can be incorporated into
    flux-analysis pipelines (e.g., FiatFlux, INCA)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:7ed21b94756745f34ce1c4645af2716955c88d91e42fec90733bb48ea28918b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# mass-difference-theory-based-correction

## Summary

Resolution-dependent natural-abundance and tracer-impurity correction of LC-MS fractional abundances of measured isotopologs (FAM) using mass difference theory (MDT), implemented in ElemCor. Unlike IsoCor-style combinatorial correction (NRE), MDT scales the correction with nominal instrument resolution, which matters on high-resolving-power Orbitrap data.

## When to use

Use when correcting FAM/MDV from high-resolution LC-MS stable isotope labeling data (e.g., 13C, 2H, 15N, 18O, 34S tracers on an Orbitrap at nominal resolutions like 140,000 or 280,000) to compute isotopic enrichment.

## When NOT to use

- The tracer element is not present in a compound's formula — the algorithm will report an error (e.g., selecting C or S as tracer on 15N data)
- Instrument resolution is extremely low, where ElemCor reduces to IsoCor's NRE combinatorial correction and resolution-aware MDT offers no advantage
- The compound is high-mass with relatively large spectral inaccuracy, where the ULS correction (using unlabeled samples) is recommended over MDT

## Inputs

- Labeled-sample FAM spreadsheet in XLSX format (e.g., test_labelN_sim.xlsx), one column vector per compound per sample, typically length Nt+1 covering M+0..M+Nt
- Nominal instrument resolution (e.g., 140,000) and mass analyzer type (Orbitrap)
- Isotopic purity of the tracer nutrient
- Tracer element selection (13C, 2H, 15N, 18O, or 34S); the tracer element must be present in every compound formula

## Outputs

- Corrected mass distribution vectors (MDV) per compound per sample
- Isotopic enrichment per compound per sample
- Fold change of pool size per compound per sample
- Results written as new sheets in the original input XLSX file and previewed in the figure window

## How to apply

Build the correction matrix from mass difference theory so that only isotopologs unresolvable at the specified nominal instrument resolution (e.g., 140,000 for 15N Orbitrap data) contribute, since high-resolving-power instruments can resolve same-nominal-mass isotopologs and including all of them is no longer justified. Compute the mass distribution vector (MDV) by non-negative linear regression of the correction matrix equation followed by normalization to a sum of 1. Run MDT-only (skip unlabeled data loading) for low-mass metabolites to avoid the extra cost of unlabeled measurements, and switch to the ULS correction for high-mass compounds with large spectral inaccuracy that unlabeled standards can track. In ElemCor's workflow: load labeled data (optional unlabeled data), specify isotopic purity of nutrient and nominal resolution, select the correct tracer element and mass analyzer, then calculate isotopic enrichment per compound. Validate against the 20% theoretical enrichment of simulated 15N metabolites by RMSE, where modified MDT/ULS should beat raw FAM, NRE (IsoCor-style combinatorial correction), and the original defective MDT/ULS implementations.

## Related tools

- **ElemCor** (Software tool implementing the MDT correction matrix, non-negative linear regression, and enrichment calculation for high-resolution LC-MS labeling data) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Combinatorial correction baseline (NRE) that ignores resolution effects; used as comparison showing MDT is significantly more accurate on high-resolution data)
- **MATLAB** (Development environment; ElemCor's MATLAB functions can be incorporated into flux-analysis pipelines (e.g., FiatFlux, INCA))

## Evaluation signals

- MDV entries are non-negative and normalize to a sum of 1 after non-negative linear regression of the correction matrix
- Isotopic enrichment of simulated 15N metabolites (20% theoretical enrichment, Orbitrap resolution 140,000) shows significantly lower RMSE for modified MDT/ULS than for raw FAM, NRE, and the original MDT/ULS implementations
- Amount of correction scales monotonically with instrument resolution: identical to IsoCor at extremely low resolution, no correction at extremely high resolution
- Corrected MDVs appear in a new sheet of the input XLSX and match the figure-window preview for the selected compound/sample cell
- Example files process within ~10 seconds on a 2.6GHz Core i7; results not appearing within a minute usually indicate a wrong tracer selection

## Limitations

- Original MDT/ULS implementations had mathematical defects; ElemCor's modified schemes must be used instead of the older versions
- ElemCor was developed under MATLAB (2016b); running the executable requires MATLAB Compiler Runtime (network connection during install), though MATLAB itself is not required at run time
- No ElemCor software version/release identifier or license terms are stated in the source record
- Experimental validation is limited to S. cerevisiae with 15N ammonium (20.0% 15N); simulated files cover 15N and 34S at 140,000 and 280,000 nominal resolution
- Current version prints only a general error message; troubleshooting relies on checking tracer selection, file directory, and matching analytes/row counts between labeled and unlabeled files

## Evidence

- [abstract] Resolution-dependent MDT correction: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [discussion] MDV computation via NNLS and normalization: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [discussion] NRE (IsoCor) less accurate: "correction without considering resolution effect (NRE, used in IsoCor) was significantly less accurate than other correction methods"
- [intro] Including all isotopologs not justified at high resolving power: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
- [readme] Test data format and resolutions: "The package includes example data files from labeled samples "test_labelN_sim.xlsx" and "test_labelS_sim.xlsx""
- [readme] Monotonic resolution dependence vs IsoCor: "Therefore, the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [readme] Tracer element selection requirement: "If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly."
