---
name: isotopic-enrichment-correction
description: Use when LC-MS isotopic labeling data (FAM vectors, e.g. XLSX per sample)
  from high-resolving-power instruments like the Orbitrap must be corrected for natural
  abundance and tracer isotopic impurity to obtain MDV and enrichment.
when_to_use_negative:
- The tracer element selected is not present in some compounds' formulas (e.g. 15N
  data loaded but C or S selected) — the algorithm will error
- Instrument resolution is extremely low, where ElemCor is identical to IsoCor and
  the resolution-dependent machinery adds no value
- The goal is flux analysis requiring full source-code integration without access
  to the MATLAB functions (MATLAB-based implementation may limit portability)
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Primary GUI/MATLAB tool that performs the resolution-dependent MDT and ULS
    correction and writes MDV/enrichment results back into the input XLSX
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Development/runtime environment; ElemCor requires MATLAB Compiler Runtime,
    and its MATLAB functions can be adapted into flux analysis pipelines (e.g. FiatFlux,
    INCA)
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Baseline combinatorial (NRE) correction that ignores resolution effects, used
    as a comparison point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:f7d19cd93a3080787546322e6f05c4fcb777c10793f0e259f35eea1e4a4705c1
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# isotopic-enrichment-correction

## Summary

Correct fractional abundances of measured isotopologs (FAM) from high-resolution LC-MS stable isotope labeling experiments into mass distribution vectors (MDV) and isotopic enrichment, using ElemCor's resolution-dependent MDT and ULS methods.

## When to use

Use when LC-MS isotopic labeling data (FAM vectors, e.g. XLSX per sample) from high-resolving-power instruments like the Orbitrap must be corrected for natural abundance and tracer isotopic impurity to obtain MDV and enrichment.

## When NOT to use

- The tracer element selected is not present in some compounds' formulas (e.g. 15N data loaded but C or S selected) — the algorithm will error
- Instrument resolution is extremely low, where ElemCor is identical to IsoCor and the resolution-dependent machinery adds no value
- The goal is flux analysis requiring full source-code integration without access to the MATLAB functions (MATLAB-based implementation may limit portability)

## Inputs

- Labeled-sample FAM data in XLSX format (e.g. test_labelN_sim.xlsx), one column vector of FAM per compound per sample
- Unlabeled-sample FAM data in XLSX format (e.g. test_unlabelN_sim.xlsx), required for ULS correction (optional for MDT-only)
- Tracer element selection (13C, 2H, 15N, 18O, or 34S)
- Tracer isotopic purity specification
- Nominal instrument resolution and mass analyzer type (e.g. Orbitrap at 140,000)

## Outputs

- Corrected mass distribution vectors (MDV) per compound and sample
- Isotopic enrichment per compound and sample
- Fold change of pool size per compound and sample
- Results stored in an additional sheet of the original input XLSX file and previewed in the figure window

## How to apply

Load labeled and (optionally) unlabeled sample data in XLSX format, where each column is the FAM (length Nt+1 covering M+0 through M+Nt) for all compounds per sample. Specify tracer isotopic purity, nominal instrument resolution (e.g. 140,000 or 280,000 on an Orbitrap), tracer element (13C, 2H, 15N, 18O, or 34S), and mass analyzer. Apply the modified MDT correction via non-negative linear regression of the correction matrix followed by normalization to a sum of 1, and/or the modified ULS correction by deconvolving the unlabeled-sample FAM Nt times; both account for the resolution effect that IsoCor's NRE (combinatorial) correction ignores. Prefer MDT for low-mass compounds (no extra experiment needed) and ULS for high-mass compounds with larger spectral inaccuracy trackable by unlabeled standards. Read the corrected MDV, enrichment, and fold change of pool size from the new sheet written back into the input XLSX.

## Related tools

- **ElemCor** (Primary GUI/MATLAB tool that performs the resolution-dependent MDT and ULS correction and writes MDV/enrichment results back into the input XLSX) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Development/runtime environment; ElemCor requires MATLAB Compiler Runtime, and its MATLAB functions can be adapted into flux analysis pipelines (e.g. FiatFlux, INCA)) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Baseline combinatorial (NRE) correction that ignores resolution effects, used as a comparison point)

## Examples

```
elemcor.exe --labeled test_labelN_sim.xlsx --unlabeled test_unlabelN_sim.xlsx --tracer 15N --resolution 140000
```

## Evaluation signals

- Corrected MDV vectors sum to 1 (post-normalization invariant of the non-negative linear regression step)
- For simulated 15N metabolites at 20% theoretical enrichment and 140,000 resolution, MDT and ULS corrected enrichment RMSE should be significantly lower than FAM, NRE, and original MDT/ULS
- Amount of correction applied should vary monotonically with instrument resolution (identical to IsoCor at extremely low resolution; no correction at extremely high resolution)
- Output sheets appear in the same input XLSX file containing MDV, enrichment, and fold change of pool size for all compounds and samples
- Example files process within ~10 seconds on a 2.6 GHz Core i7 (within a minute on typical hardware); failure usually indicates wrong tracer element selected

## Limitations

- ULS requires additional experimental measurement of unlabeled samples, adding cost; MDT is recommended for small metabolites
- Developed under MATLAB (2016b) and dependent on MATLAB Compiler Runtime — MATLAB installation not needed to run but the environment is proprietary
- Current version only prints a general error message; troubleshooting relies on manual checks of tracer selection, file location, and matching analytes/rows between labeled and unlabeled files
- Experimental validation limited to S. cerevisiae grown with 20.0% 15N ammonium

## Evidence

- [abstract] Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner.: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [discussion] Non-negative linear regression followed by normalization: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [discussion] ULS deconvolution of unlabeled-sample FAM: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [readme] Test files simulated from XCalibur at 140,000/280,000 Orbitrap resolution: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively."
- [readme] MDV and enrichment stored in additional XLSX sheets: "The final results include, MDV, enrichment, and fold change of pool size for different compounds and samples."
- [readme] ElemCor vs IsoCor resolution dependence: "Therefore, the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
- [abstract] MDT for low-mass, ULS for high-mass recommendation: "We recommend MDT for low-mass compounds for cost efficiency in experiments, and ULS for high-mass compounds with relatively large spectral inaccuracy that can be tracked by unlabeled standards."
