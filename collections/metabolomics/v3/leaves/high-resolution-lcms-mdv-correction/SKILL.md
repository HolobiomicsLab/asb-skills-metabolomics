---
name: high-resolution-lcms-mdv-correction
description: Use when LC-MS isotope-labeling FAM data (XLSX columns of isotopologue
  fractions) from Orbitrap or similar high-resolution analyzers need natural-abundance/impurity
  correction at a known nominal resolution (e.g. 140,000 or 280,000).
when_to_use_negative:
- The tracer element is not present in a compound's formula — the algorithm will report
  an error (e.g. selecting C or S as tracer for 15N-labeled data)
- Low-resolution data where isotope-resolving power is irrelevant and IsoCor-style
  combinatorial correction suffices
- No unlabeled sample is available AND spectral inaccuracy at high mass makes ULS
  unreliable — MDT should be used instead of forcing ULS
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Standalone GUI/MATLAB tool that performs the resolution-dependent MDT/ULS
    correction and computes MDV and enrichment
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass-difference-theory-based scheme inside ElemCor for correction without
    unlabeled samples
  repo: https://github.com/4dsoftware/elemcor
- name: ULS
  role: Unlabeled-sample-based scheme inside ElemCor for high-mass compounds with
    large spectral inaccuracy
  repo: https://github.com/4dsoftware/elemcor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:c871ec8643689a9271ca04720d742ec075dbc095a8d9a3f800d6de14d0a95ff5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# high-resolution-lcms-mdv-correction

## Summary

Correct fractional abundances of measured isotopologues (FAM) from high-resolution LC-MS stable isotope labeling experiments into mass distribution vectors (MDV) and isotopic enrichment, using ElemCor's MDT and ULS schemes at a specified nominal instrument resolution.

## When to use

Use when LC-MS isotope-labeling FAM data (XLSX columns of isotopologue fractions) from Orbitrap or similar high-resolution analyzers need natural-abundance/impurity correction at a known nominal resolution (e.g. 140,000 or 280,000).

## When NOT to use

- The tracer element is not present in a compound's formula — the algorithm will report an error (e.g. selecting C or S as tracer for 15N-labeled data)
- Low-resolution data where isotope-resolving power is irrelevant and IsoCor-style combinatorial correction suffices
- No unlabeled sample is available AND spectral inaccuracy at high mass makes ULS unreliable — MDT should be used instead of forcing ULS

## Inputs

- XLSX file of labeled-sample FAM (fractional abundances of measured isotopologues), one column per sample
- Optional XLSX file of unlabeled-sample FAM for the ULS correction
- Tracer isotopic purity (e.g. 20% 15N) and nominal instrument resolution (e.g. 140,000 or 280,000)
- Tracer element selection (13C, 2H, 15N, 18O, or 34S) and mass analyzer type (e.g. Orbitrap)

## Outputs

- Corrected mass distribution vectors (MDV) per compound and sample, stored in a new sheet of the input XLSX file
- Isotopic enrichment per compound and sample
- Fold change of pool size per compound and sample

## How to apply

Load the labeled FAM XLSX file (e.g. test_labelN_sim.xlsx) and, if available, the unlabeled-sample file (e.g. test_unlabelN_sim.xlsx); the unlabeled file enables the ULS scheme, otherwise ElemCor runs on mass difference theory (MDT) alone. Specify the tracer's isotopic purity and the nominal instrument resolution (e.g. 140,000 or 280,000 for Orbitrap data), then select the tracer element — 13C, 2H, 15N, 18O, or 34S — and the mass analyzer. Non-negative linear regression of the correction matrix followed by normalization to sum 1 yields the MDV, from which isotopic enrichment and fold change of pool size are computed per compound and sample; results are written into new sheets of the same input file. Choose MDT for low-mass compounds for cost efficiency, and ULS for high-mass compounds (e.g. coenzyme A derivatives) where spectral inaccuracy can be tracked by unlabeled standards.

## Related tools

- **ElemCor** (Standalone GUI/MATLAB tool that performs the resolution-dependent MDT/ULS correction and computes MDV and enrichment) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass-difference-theory-based scheme inside ElemCor for correction without unlabeled samples) — https://github.com/4dsoftware/elemcor
- **ULS** (Unlabeled-sample-based scheme inside ElemCor for high-mass compounds with large spectral inaccuracy) — https://github.com/4dsoftware/elemcor

## Examples

```
double-click ElemCor.exe after installing MATLAB Compiler Runtime via MyAppInstaller_web.exe; load test_labelN_sim.xlsx, select 15N as tracer, set resolution 140,000, then click Step 6
```

## Evaluation signals

- Corrected enrichments should approach the theoretical enrichment (e.g. 20% 15N) — compare RMSE against the no-correction (FAM) baseline; ElemCor should outperform original ULS for 13C and 34S at resolution 280,000
- MDV values must be non-negative and sum to 1 after normalization
- Output XLSX must contain new sheets with MDV, enrichment, and fold change of pool size, matching row count and analyte order of the input
- Processing time should be under ~1 minute for the example files (≤10 s on a 2.6 GHz Core i7); if not, verify the correct tracer was selected

## Limitations

- If the selected tracer element is absent from a compound's formula (e.g. C or S chosen for 15N data), the software will not run properly
- ULS requires the cost of an additional unlabeled-sample experiment; for small metabolites MDT is recommended instead
- The resolution effect correction is monotonically dependent on instrument resolution: at extremely low resolution it equals IsoCor, at extremely high resolution no correction is performed
- Current version only prints a general error message; troubleshooting requires checking tracer selection, file location, and matching analytes/rows between labeled and unlabeled files

## Evidence

- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] Step 3 specifies isotopic purity and nominal instrument resolution; Step 4 selects tracer element; Step 5 selects mass analyzer; Step 6 computes enrichment.: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified."
- [readme] Example input files are simulated XCalibur data at nominal resolutions 140,000 and 280,000 from Orbitrap analyzer.: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
- [readme] Tracer element options include 13C, 2H, 15N, 18O, and 34S.: "In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [abstract] Recommendation: MDT for low-mass compounds for cost efficiency; ULS for high-mass compounds with large spectral inaccuracy tracked by unlabeled standards.: "We recommend MDT for low-mass compounds for cost efficiency in experiments, and ULS for high-mass compounds with relatively large spectral inaccuracy that can be tracked by unlabeled standards."
- [readme] The final results include MDV, enrichment, and fold change of pool size, saved into additional sheets of the original XLSX file.: "The final results include, MDV, enrichment, and fold change of pool size for different compounds and samples."
