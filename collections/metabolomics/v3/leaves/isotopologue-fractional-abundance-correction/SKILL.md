---
name: isotopologue-fractional-abundance-correction
description: Use when LC-MS data from isotope labeling experiments (FAM vectors per
  compound/sample, e.g. XLSX input) must be corrected for natural abundance, tracer
  isotopic impurity, and resolution effects before deriving MDVs and enrichment.
when_to_use_negative:
- When the tracer element selected is not actually present in the compound's formula
  (e.g. loading 15N data but selecting C or S as tracer) — the algorithm reports an
  error
- When input is already corrected MDV data or a flux-analysis result rather than measured
  FAM from isotopolog intensities
- When correction without resolution effect is desired at extremely low instrument
  resolution — ElemCor then behaves identically to IsoCor, so a combinatorial corrector
  suffices
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ElemCor
  role: Provides the resolution-dependent correction matrix and the non-negative-regression
    solver used to convert FAM into MDV and enrichment
  repo: github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory method used to build the total correction matrix in
    a resolution-dependent manner
  repo: github.com/4dsoftware/elemcor
- name: scipy.optimize.nnls
  role: Equivalent Python reimplementation of the non-negative linear regression step
    solving C x' = z
- name: MATLAB
  role: Environment hosting ElemCor's provided MATLAB function, adaptable into flux-analysis
    suites such as FiatFlux and INCA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:9a095cdbf336baef00b785c4074409f2161aba4913edb40a34238731d3ce00c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemCor
  - MATLAB
  - MDT
  - scipy.optimize.nnls
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# isotopologue-fractional-abundance-correction

## Summary

Corrects fractional abundances of measured isotopologs (FAM) from high-resolution LC-MS stable isotope labeling experiments into mass distribution vectors (MDV) via non-negative linear regression of a resolution-dependent total correction matrix, then computes tracer isotopic enrichment per compound.

## When to use

Use when LC-MS data from isotope labeling experiments (FAM vectors per compound/sample, e.g. XLSX input) must be corrected for natural abundance, tracer isotopic impurity, and resolution effects before deriving MDVs and enrichment.

## When NOT to use

- When the tracer element selected is not actually present in the compound's formula (e.g. loading 15N data but selecting C or S as tracer) — the algorithm reports an error
- When input is already corrected MDV data or a flux-analysis result rather than measured FAM from isotopolog intensities
- When correction without resolution effect is desired at extremely low instrument resolution — ElemCor then behaves identically to IsoCor, so a combinatorial corrector suffices

## Inputs

- FAM (fractional abundance of measured isotopologs) column vectors per compound/sample, typically from an XLSX spreadsheet (e.g. test_labelN_sim.xlsx, test_labelS_sim.xlsx)
- Total correction matrix built by MDT (mass difference theory), optionally chained with ULS (unlabeled sample) deconvolution
- Isotopic purity of the nutrient, nominal instrument resolution, tracer element choice, and mass analyzer type

## Outputs

- Corrected mass distribution vector (MDV) per compound/sample, normalized to sum 1
- Isotopic enrichment scalar per compound (sum over i of i * x'_i)
- Pool-size fold change per compound/sample; results stored in a new sheet of the input XLSX file

## How to apply

Construct a total correction matrix C using mass difference theory (MDT) — optionally chained with unlabeled-sample (ULS) deconvolution — that accounts for natural isotope abundance, tracer isotopic impurity, and instrument resolution (ElemCor allows 13C, 2H, 15N, 18O, 34S as tracer elements and specifies nominal instrument resolution, e.g. 140,000–280,000 on an Orbitrap). Load the measured FAM vector z from labeled-sample data (length typically Nt+1, covering M+0..M+Nt isotopologs, though other lengths are supported). Solve C x' = z by non-negative linear regression — ElemCor's provided MATLAB function or an equivalent reimplementation such as scipy.optimize.nnls — to obtain a non-negative solution. Normalize x' to sum to 1, exactly as ElemCor does after regression, yielding the MDV; compute isotopic enrichment as the tracer-count-weighted sum sum_i i * x'_i for each compound. ElemCor additionally stores corrected MDVs, enrichment, and pool-size fold change into a new sheet of the original XLSX input file.

## Related tools

- **ElemCor** (Provides the resolution-dependent correction matrix and the non-negative-regression solver used to convert FAM into MDV and enrichment) — github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory method used to build the total correction matrix in a resolution-dependent manner) — github.com/4dsoftware/elemcor
- **scipy.optimize.nnls** (Equivalent Python reimplementation of the non-negative linear regression step solving C x' = z)
- **MATLAB** (Environment hosting ElemCor's provided MATLAB function, adaptable into flux-analysis suites such as FiatFlux and INCA)

## Examples

```
from scipy.optimize import nnls; x_prime, _ = nnls(C_matrix, fam_vector); mdv = x_prime / x_prime.sum(); enrichment = sum(i * x for i, x in enumerate(mdv))
```

## Evaluation signals

- Every MDV entry is non-negative (a direct consequence of the NNLS solution)
- The normalized MDV sums to 1 within floating-point tolerance
- The enrichment scalar equals the tracer-count-weighted sum sum_i i * x'_i computed from the normalized MDV
- Uncorrected FAM vs corrected MDV comparison should show resolution-dependent shrinkage of spurious high-mass isotopolog abundance; correction amount is monotonically dependent on instrument resolution (identical to IsoCor at extremely low resolution, no correction at extremely high resolution)
- Output contains a single enrichment scalar per compound/sample consistent with the corrected MDV

## Limitations

- ULS-based correction requires additional experimental measurement of unlabeled samples; MDT alone is recommended for small metabolites to avoid this cost
- ElemCor was developed under MATLAB (2016b), so reproducing the exact solver depends on the proprietary MATLAB environment
- Only a general error message is printed on failure; troubleshooting requires checking tracer selection, file location, and that labeled/unlabeled files share the same analytes and row count
- No software version/release identifier or license terms are stated in the article record

## Evidence

- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] Enrichment computed per compound in step 6: "Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6."
- [intro] FAM must be corrected to obtain MDV: "the fractional abundance of measured isotopologs (FAM) collected from the instrument must be corrected to obtain MDV."
- [readme] Results stored in a new sheet of the input XLSX file: "They are automatically saved into additional sheets in the original XLSX file. The final results include, MDV, enrichment, and fold change of pool size for different compounds and samples."
- [readme] Resolution dependence relative to IsoCor: "the amount of correction perform by ElemCor is monotonically dependent on instrument resolution."
