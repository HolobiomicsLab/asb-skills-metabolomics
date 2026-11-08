---
name: mass-distribution-vector-normalization
description: Use when LC-MS isotope-labeling FAM vectors must be corrected for natural
  abundance and tracer isotopic impurity to yield an MDV and per-compound isotopic
  enrichment.
when_to_use_negative:
- The FAM vector is already a corrected MDV (i.e. the sample is unlabeled or correction
  has already been applied) — re-solving C x' = z would double-correct.
- The tracer element selected does not actually label the compound (e.g. selecting
  C or S when the data are 15N-labeled), which the README warns makes the algorithm
  report an error because the tracer element is absent from some compound formulas.
- Instrument resolution is extremely low (where ElemCor degenerates to IsoCor's combinatorial
  correction) or extremely high (where it performs no correction at all) and a different
  correction regime is more appropriate.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Provides the total correction matrix and the non-negative linear regression
    routine used to obtain the MDV and isotopic enrichment from FAM.
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Environment in which ElemCor's non-negative regression function is implemented;
    its flux-analysis adaptability (e.g. with FiatFlux or INCA) relies on these MATLAB
    functions.
- name: MDT
  role: Mass difference theory method that constructs the correction matrix C when
    no unlabeled sample is loaded, feeding the regression step.
- name: ULS
  role: Unlabeled-sample method that supplies the correction matrix via deconvolution
    of unlabeled FAM when an unlabeled file is loaded.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:4e56c06e7db2e5df01560c7ac92621618269e43639ef7d9d210038c866170168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-distribution-vector-normalization

## Summary

Obtain the corrected mass distribution vector (MDV) and isotopic enrichment from a measured fractional-abundance vector (FAM) by solving the correction-matrix equation via non-negative linear regression, then normalizing the solution to sum to 1, as implemented in ElemCor for high-resolution LC-MS isotope-labeling data.

## When to use

Use when LC-MS isotope-labeling FAM vectors must be corrected for natural abundance and tracer isotopic impurity to yield an MDV and per-compound isotopic enrichment.

## When NOT to use

- The FAM vector is already a corrected MDV (i.e. the sample is unlabeled or correction has already been applied) — re-solving C x' = z would double-correct.
- The tracer element selected does not actually label the compound (e.g. selecting C or S when the data are 15N-labeled), which the README warns makes the algorithm report an error because the tracer element is absent from some compound formulas.
- Instrument resolution is extremely low (where ElemCor degenerates to IsoCor's combinatorial correction) or extremely high (where it performs no correction at all) and a different correction regime is more appropriate.

## Inputs

- Total correction matrix C (constructed by MDT and/or unlabeled-sample deconvolution, as in ElemCor)
- Measured fractional abundance of measured isotopologs (FAM) vector z for each compound/sample, e.g. from test_labelN_sim.xlsx

## Outputs

- Normalized mass distribution vector (MDV) per compound/sample (non-negative, sums to 1)
- Isotopic enrichment scalar per compound (tracer-count-weighted sum Σ i·x'_i)

## How to apply

Formulate the correction as C x' = z, where C is the total correction matrix (built by ElemCor via mass difference theory and/or unlabeled-sample deconvolution, chained from the preceding MDT construction step) and z is the measured FAM vector extracted from the labeled-sample XLSX file (e.g. test_labelN_sim.xlsx). Solve the system with non-negative linear regression — ElemCor's provided MATLAB function (scipy.optimize.nnls is an equivalent reimplementation) — to obtain a non-negative solution x'. Normalize x' to the sum of 1, exactly as ElemCor does after regression, to yield the MDV. Compute the isotopic enrichment as the tracer-count-weighted sum Σ i·x'_i. Validation invariants: every MDV entry must be non-negative, the MDV must sum to 1 within floating-point tolerance, and the output must contain a single enrichment scalar consistent with the normalized MDV.

## Related tools

- **ElemCor** (Provides the total correction matrix and the non-negative linear regression routine used to obtain the MDV and isotopic enrichment from FAM.) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Environment in which ElemCor's non-negative regression function is implemented; its flux-analysis adaptability (e.g. with FiatFlux or INCA) relies on these MATLAB functions.)
- **MDT** (Mass difference theory method that constructs the correction matrix C when no unlabeled sample is loaded, feeding the regression step.)
- **ULS** (Unlabeled-sample method that supplies the correction matrix via deconvolution of unlabeled FAM when an unlabeled file is loaded.)

## Examples

```
from scipy.optimize import nnls; import numpy as np; x_prime, _ = nnls(C, z); mdv = x_prime / x_prime.sum(); enrichment = float(np.dot(np.arange(len(mdv)), mdv))
```

## Evaluation signals

- All MDV entries are non-negative (no negative abundance values appear in the solution vector).
- The normalized MDV sums to 1 within floating-point tolerance.
- Isotopic enrichment equals Σ i·x'_i using the same normalized MDV entries, i.e. a single non-negative scalar per compound.
- The regression is applied to the same correction matrix C and FAM vector z as loaded/constructed in the preceding ElemCor steps (tracer element, resolution, mass analyzer unchanged between construction and solving).
- For the provided example files (test_labelN_sim.xlsx / test_labelS_sim.xlsx), results are produced within the README-reported runtime (~10 s on a 2.6 GHz Core i7, ~20 s on a 1.5 GHz Core i5) and stored as an additional sheet in the input XLSX file.

## Limitations

- ElemCor was developed under MATLAB (2016b); incorporating source code into other pipelines requires the authors' MATLAB functions, which are shared on request rather than distributed as a standalone library.
- If the selected tracer element is not present in the compound's formula (e.g. choosing C or S for 15N data), the algorithm reports an error and the run fails.
- ULS-based correction requires an additional unlabeled-sample measurement; the article recommends MDT for small metabolites to avoid that cost.
- The current version only prints a general error message; troubleshooting relies on checking tracer selection, file location, and matched analytes between labeled and unlabeled files.

## Evidence

- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6.: "Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6."
- [readme] The final results include, MDV, enrichment, and fold change of pool size for different compounds and samples.: "The final results include, MDV, enrichment, and fold change of pool size for different compounds and samples."
- [readme] the fractional abundances of measured isotopologues (FAM) collected from the instrument must be corrected to obtain mass distribution vectors (MDV): "the fractional abundances of measured isotopologues (FAM) collected from the instrument must be corrected to obtain mass distribution vectors (MDV)"
- [readme] If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly.: "If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly."
