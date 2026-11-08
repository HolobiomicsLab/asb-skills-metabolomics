---
name: isotopic-enrichment-calculation
description: Use when a measured FAM vector (fractional abundances of M+0..M+Nt isotopologs)
  from a stable isotope labeling LC-MS experiment must be corrected for natural abundance
  and tracer isotopic impurity and converted to an MDV plus a single enrichment scalar
  per compound.
when_to_use_negative:
- The selected tracer element is not present in the compound's formula — the algorithm
  reports an error (e.g., loading 15N data but selecting C or S as tracer)
- The labeled and unlabeled XLSX files have different analytes or different numbers
  of rows, since they must be aligned for correction
- You only need nominal-mass correction at extremely low instrument resolution, where
  ElemCor reduces to IsoCor's no-resolution-effect correction rather than a distinct
  method
tools:
- name: ElemCor
  role: Provides the non-negative regression solver and GUI that compute MDV and enrichment
    from the correction matrix; developed in MATLAB (2016b)
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory method used to construct the total correction matrix
    that the regression is solved against, accounting for resolution effects without
    unlabeled samples
  repo: https://github.com/4dsoftware/elemcor
- name: scipy.optimize.nnls
  role: Equivalent non-negative least squares reimplementation of ElemCor's regression
    step (Cx' = z) for use outside MATLAB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:f0a6d481f5a224c15ae1b7a53bacf56fb8d2af8bc89a3ea994bc6dc91777ad77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MATLAB
  - MDT
  - scipy.optimize.nnls
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# isotopic-enrichment-calculation

## Summary

Computes corrected mass distribution vectors (MDV) and isotopic enrichment from measured fractional abundances of measured isotopologs (FAM) by non-negative linear regression against a total correction matrix, as implemented in ElemCor for high-resolution LC-MS stable isotope labeling data.

## When to use

Use when a measured FAM vector (fractional abundances of M+0..M+Nt isotopologs) from a stable isotope labeling LC-MS experiment must be corrected for natural abundance and tracer isotopic impurity and converted to an MDV plus a single enrichment scalar per compound.

## When NOT to use

- The selected tracer element is not present in the compound's formula — the algorithm reports an error (e.g., loading 15N data but selecting C or S as tracer)
- The labeled and unlabeled XLSX files have different analytes or different numbers of rows, since they must be aligned for correction
- You only need nominal-mass correction at extremely low instrument resolution, where ElemCor reduces to IsoCor's no-resolution-effect correction rather than a distinct method

## Inputs

- Total correction matrix C (e.g., MDT-constructed, resolution-dependent, for a specific compound and tracer element)
- Measured FAM column vector z for each compound/sample from a labeled-sample XLSX file (e.g., test_labelN_sim.xlsx, test_labelS_sim.xlsx)
- Tracer element specification (13C, 2H, 15N, 18O, or 34S) and isotopic purity of the nutrient
- Nominal instrument resolution and mass analyzer (e.g., Orbitrap at 140,000 or 280,000)

## Outputs

- Corrected MDV vector (non-negative, normalized to sum of 1) per compound per sample
- Isotopic enrichment scalar per compound (tracer-count-weighted sum of the MDV)
- Results written to a CSV (e.g., glutamine_mdv_enrichment.csv) or an added sheet in the original XLSX input file

## How to apply

Load the total correction matrix C (e.g., built by mass difference theory (MDT) for a compound such as glutamine with a 15N tracer) and the FAM column vector z extracted from the labeled sample spreadsheet (e.g., test_labelN_sim.xlsx). Solve C x' = z by non-negative linear regression — ElemCor's provided MATLAB function, equivalently scipy.optimize.nnls — because the corrected isotopolog fractions must be non-negative; unregularized least squares can produce negative entries. Normalize x' to the sum of 1, exactly as ElemCor does after regression, to obtain the MDV. Compute isotopic enrichment as the tracer-count-weighted sum, sum over i of i * x'_i, and write the MDV entries and enrichment per compound (ElemCor stores these in a new sheet of the input XLSX file). FAM may have a length other than Nt+1 (number of tracer atoms + 1); the solver handles both cases.

## Related tools

- **ElemCor** (Provides the non-negative regression solver and GUI that compute MDV and enrichment from the correction matrix; developed in MATLAB (2016b)) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory method used to construct the total correction matrix that the regression is solved against, accounting for resolution effects without unlabeled samples) — https://github.com/4dsoftware/elemcor
- **scipy.optimize.nnls** (Equivalent non-negative least squares reimplementation of ElemCor's regression step (Cx' = z) for use outside MATLAB)

## Examples

```
from scipy.optimize import nnls; x,_ = nnls(C, z); x = x/x.sum(); enrichment = sum(i*xi for i,xi in enumerate(x))
```

## Evaluation signals

- Every MDV entry in the solution x' is non-negative after NNLS solving
- The normalized MDV sums to 1 within floating-point tolerance
- The enrichment scalar equals the tracer-count-weighted sum sum_i(i * x'_i) computed from the normalized MDV
- Output contains exactly one enrichment value per compound per sample, matching ElemCor's stored results in the added sheet of the input file
- Regression correctly handles FAM vectors of length Nt+1 (M+0..M+Nt isotopologs) as well as other lengths

## Limitations

- ElemCor was developed under the proprietary MATLAB (2016b) environment, which may limit reproducibility; running the executable requires MATLAB Compiler Runtime
- The result depends on resolution: at extremely low resolution ElemCor is identical to IsoCor (no resolution-effect correction), and at extremely high resolution it performs no correction at all
- Selecting the wrong tracer element causes the software to fail or run improperly, since compounds lacking the tracer element in their formula trigger an error
- Validation in the paper was limited to a single organism (S. cerevisiae) and a single tracer (20.0% 15N ammonium), alongside simulated data

## Evidence

- [discussion] Non-negative linear regression followed by normalization to compute MDV and isotopic enrichment: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [intro] FAM must be corrected to obtain MDV: "the fractional abundance of measured isotopologs (FAM) collected from the instrument must be corrected to obtain MDV."
- [readme] ElemCor stores corrected MDVs in a new sheet of the input file: "After correction, the MDVs of corresponding samples are shown in the figure window in ElemCor and also stored in a different sheet in the same input file."
- [readme] FAM length flexibility: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues. Here Nt stands for the number of tracer atoms."
- [readme] Resolution-dependent behavior relative to IsoCor: "When instrument resolution is extremely low, ElemCor is identical to IsoCor. When instrument resolution is extremely high, ElemCor performs no correction at all."
