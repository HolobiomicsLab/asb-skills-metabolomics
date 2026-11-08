---
name: stable-isotope-lcms-labeling-data-processing
description: Use when LC-MS data from a stable isotope labeling experiment (13C, 2H,
  15N, 18O, or 34S tracer) are given as FAM column vectors in XLSX form and must be
  resolution-corrected to MDV and per-compound enrichment.
when_to_use_negative:
- The tracer element selected is not present in some compounds' formulas — the algorithm
  reports an error (e.g. loading 15N data but selecting C or S as tracer)
- Instrument resolution is extremely low, where ElemCor becomes identical to IsoCor,
  or extremely high, where it performs no correction at all — the resolution-dependent
  correction amount may not match the experimental design
- The task requires cross-platform or non-MATLAB execution without the compiled runtime
  — ElemCor is a MATLAB stand-alone application and workflows must obtain its MATLAB
  functions for pipeline integration
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: 'Primary tool: performs resolution-dependent correction of FAM to MDV and
    calculates isotopic enrichment; provides a graphical interface and a MATLAB function
    adaptable to flux analysis workflows'
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory method underlying construction of the correction matrix;
    recommended for low-mass compounds for cost efficiency
  repo: https://github.com/4dsoftware/elemcor
- name: ULS
  role: Unlabeled-sample-based method for building the correction matrix; recommended
    for high-mass compounds with relatively large spectral inaccuracy
  repo: https://github.com/4dsoftware/elemcor
- name: MATLAB
  role: Development/runtime environment; ElemCor's MATLAB function can be adapted
    for FiatFlux/INCA flux analysis pipelines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:dd1f3a0632af5d056a71ae760737434c63e04dbe9a97cc085efb445d77f12597
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# stable-isotope-lcms-labeling-data-processing

## Summary

Corrects fractional abundances of measured isotopologs (FAM) from high-resolution LC-MS stable isotope labeling experiments for natural abundance and tracer isotopic impurity using ElemCor's mass difference theory (MDT) and/or unlabeled sample (ULS) correction matrices, then computes mass distribution vectors (MDV) and isotopic enrichment via non-negative linear regression.

## When to use

Use when LC-MS data from a stable isotope labeling experiment (13C, 2H, 15N, 18O, or 34S tracer) are given as FAM column vectors in XLSX form and must be resolution-corrected to MDV and per-compound enrichment.

## When NOT to use

- The tracer element selected is not present in some compounds' formulas — the algorithm reports an error (e.g. loading 15N data but selecting C or S as tracer)
- Instrument resolution is extremely low, where ElemCor becomes identical to IsoCor, or extremely high, where it performs no correction at all — the resolution-dependent correction amount may not match the experimental design
- The task requires cross-platform or non-MATLAB execution without the compiled runtime — ElemCor is a MATLAB stand-alone application and workflows must obtain its MATLAB functions for pipeline integration

## Inputs

- Labeled-sample FAM vectors in XLSX format (e.g. test_labelN_sim.xlsx), one column per sample, one column vector per compound
- Optional unlabeled-sample FAM XLSX file (e.g. test_unlabelN_sim.xlsx) for ULS-based correction
- Isotopic purity of the nutrient (e.g. 20.0% 15N in ammonium) and nominal instrument resolution
- Tracer element selection (13C, 2H, 15N, 18O, or 34S) and mass analyzer type (e.g. Orbitrap)

## Outputs

- Corrected MDV for each compound and sample, written to a new sheet in the same input XLSX file
- Isotopic enrichment for each compound
- Fold change of pool size for each compound and sample
- Figure-window preview of FAM and MDV before and after correction for a selected compound/sample

## How to apply

Load the labeled FAM data from an XLSX file (optionally unlabeled-sample data; if omitted, correction runs on mass difference theory only). Specify the isotopic purity of the nutrient, the nominal instrument resolution (e.g. Orbitrap 140,000 or 280,000 in the test files), the tracer element, and the mass analyzer; the tracer must be present in every compound's formula or the algorithm will error. Build the total correction matrix via MDT and/or ULS, then solve the correction matrix equation by non-negative linear regression followed by normalization to the sum of 1 to obtain the MDV. Calculate isotopic enrichment as the tracer-count-weighted sum over the normalized MDV entries; corrected MDVs, enrichment, and fold change of pool size are written into additional sheets of the original XLSX file. FAM length need not equal Nt+1 (Nt = number of tracer atoms) — ElemCor handles other lengths.

## Related tools

- **ElemCor** (Primary tool: performs resolution-dependent correction of FAM to MDV and calculates isotopic enrichment; provides a graphical interface and a MATLAB function adaptable to flux analysis workflows) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory method underlying construction of the correction matrix; recommended for low-mass compounds for cost efficiency) — https://github.com/4dsoftware/elemcor
- **ULS** (Unlabeled-sample-based method for building the correction matrix; recommended for high-mass compounds with relatively large spectral inaccuracy) — https://github.com/4dsoftware/elemcor
- **MATLAB** (Development/runtime environment; ElemCor's MATLAB function can be adapted for FiatFlux/INCA flux analysis pipelines)

## Examples

```
x = scipy.optimize.nnls(C, z); mdv = x / x.sum(); enrichment = sum(i * mdv[i] for i in range(len(mdv)))
```

## Evaluation signals

- Every MDV entry is non-negative (a direct consequence of the non-negative linear regression step)
- MDV entries sum to 1 within floating-point tolerance after the normalization step
- Enrichment equals the tracer-count-weighted sum sum_i i * x'_i over the normalized MDV and is consistent with it
- Corrected results (MDV, enrichment, fold change of pool size) appear in additional sheets of the original XLSX file matching the figure-window preview
- Example files process within ~10 s on a 2.6 GHz Core i7 (20 s on 1.5 GHz Core i5); failure to finish within a minute usually indicates the wrong tracer element was selected

## Limitations

- ElemCor was developed under MATLAB (2016b) and requires MATLAB Compiler Runtime; the dependency on a proprietary MATLAB environment may limit reproducibility
- ULS requires additional experimental measurement of unlabeled samples, a cost limitation; MDT is recommended for small metabolites
- Experimental validation was limited to a single organism (S. cerevisiae) and a single tracer (20.0% 15N ammonium)
- If the 15N data are loaded but C or S is selected as tracer, the software will not run properly because some compounds lack C or S in their formula
- The current version only prints a general error message; troubleshooting requires checking tracer selection, file directory, and matching analytes/rows between labeled and unlabeled files

## Evidence

- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [abstract] ElemCor performs resolution-dependent correction: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [readme] README tutorial steps: "In Step 3, isotopic purity of nutrient and nominal instrument resolution are specified. In Step 4, the tracer element is selected."
- [readme] README Q&A on output location: "They are automatically saved into additional sheets in the original XLSX file. The final results include, MDV, enrichment, and fold change of pool size"
- [readme] README on wrong tracer: "If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly."
