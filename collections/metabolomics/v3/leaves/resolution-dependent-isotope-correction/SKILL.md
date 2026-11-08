---
name: resolution-dependent-isotope-correction
description: Use when correcting LC-MS isotope labeling data (13C, 2H, 15N, 18O, or
  34S tracers) acquired on Orbitrap-class analyzers where resolution-dependent isotopolog
  overlap makes combinatorial correction (IsoCor-style NRE) inaccurate.
when_to_use_negative:
- The tracer element is not present in some compounds' formulas (e.g., selecting C
  or S as tracer for 15N-only data) — the algorithm will report an error and the software
  will not run properly
- Instrument resolution is extremely low, where ElemCor becomes identical to IsoCor's
  combinatorial correction and the resolution-aware machinery adds no value
- Only FAM-to-MDV conversion without correction is needed — this skill is specifically
  the natural-abundance/impurity correction step, not downstream flux fitting
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ElemCor
  role: Primary software implementing the modified MDT and ULS resolution-dependent
    corrections with a graphical interface and MATLAB functions
  repo: github.com/4dsoftware/elemcor
- name: MATLAB
  role: Environment for the ElemCor MATLAB functions, adaptable into metabolic flux
    analysis pipelines (e.g., with FiatFlux or INCA)
- name: IsoCor
  role: Combinatorial (no-resolution-effect, NRE) correction used as the lower-accuracy
    baseline for comparison
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:c8adcffc1291f1571f50fe0cf58caa460fcc29079912eadf78a4ec4d18e0191f
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# resolution-dependent-isotope-correction

## Summary

Apply ElemCor's modified mass difference theory (MDT) and unlabeled-sample (ULS) corrections to convert measured fractional abundances of isotopologs (FAM) into mass distribution vectors (MDV) and isotopic enrichment, explicitly accounting for instrument resolution effects on high-resolving-power LC-MS data.

## When to use

Use when correcting LC-MS isotope labeling data (13C, 2H, 15N, 18O, or 34S tracers) acquired on Orbitrap-class analyzers where resolution-dependent isotopolog overlap makes combinatorial correction (IsoCor-style NRE) inaccurate.

## When NOT to use

- The tracer element is not present in some compounds' formulas (e.g., selecting C or S as tracer for 15N-only data) — the algorithm will report an error and the software will not run properly
- Instrument resolution is extremely low, where ElemCor becomes identical to IsoCor's combinatorial correction and the resolution-aware machinery adds no value
- Only FAM-to-MDV conversion without correction is needed — this skill is specifically the natural-abundance/impurity correction step, not downstream flux fitting

## Inputs

- Labeled-sample FAM XLSX file (e.g., test_labelN_sim.xlsx or test_labelS_sim.xlsx); each column is the FAM of all compounds in one sample
- Unlabeled-sample FAM XLSX file (e.g., test_unlabelN_sim.xlsx or test_unlabelS_sim.xlsx); optional, required only for ULS
- Tracer element selection (13C, 2H, 15N, 18O, or 34S)
- Isotopic purity of the nutrient and nominal instrument resolution (e.g., 140,000 or 280,000)
- Mass analyzer selection (e.g., Orbitrap)

## Outputs

- Corrected mass distribution vectors (MDV) per compound and sample, stored in a new sheet of the input XLSX
- Isotopic enrichment per compound and sample
- Fold change of pool size for different compounds and samples
- Per-method comparison metrics such as enrichment RMSE against theoretical enrichment (when run on simulated data of known enrichment)

## How to apply

Load labeled and (optionally) unlabeled FAM data from XLSX files where each column vector is the FAM of all compounds in a sample (e.g., test_labelN_sim.xlsx and test_unlabelN_sim.xlsx). Specify the isotopic purity of the nutrient and the nominal instrument resolution (e.g., 140,000), select the tracer element (13C, 2H, 15N, 18O, or 34S — the correct tracer must be selected or the algorithm errors on compounds lacking the element), and select the mass analyzer (Orbitrap). ElemCor builds a total correction matrix via MDT and/or ULS — the latter by deconvolution of the unlabeled-sample FAM for the tracer element Nt times — then obtains the MDV by non-negative linear regression of the correction-matrix equation followed by normalization to a sum of 1, and computes isotopic enrichment per compound. Corrected MDVs, enrichment, and pool-size fold change are written to additional sheets in the original XLSX file. Choose MDT for low-mass compounds (no extra unlabeled measurement needed) and ULS for high-mass compounds with relatively large spectral inaccuracy trackable by unlabeled standards.

## Related tools

- **ElemCor** (Primary software implementing the modified MDT and ULS resolution-dependent corrections with a graphical interface and MATLAB functions) — github.com/4dsoftware/elemcor
- **MATLAB** (Environment for the ElemCor MATLAB functions, adaptable into metabolic flux analysis pipelines (e.g., with FiatFlux or INCA))
- **IsoCor** (Combinatorial (no-resolution-effect, NRE) correction used as the lower-accuracy baseline for comparison)

## Examples

```
ElemCor.exe  # load test_labelN_sim.xlsx and test_unlabelN_sim.xlsx, set isotopic purity, resolution=140000, tracer=15N, analyzer=Orbitrap, then analyze
```

## Evaluation signals

- Corrected MDVs sum to 1 after non-negative linear regression normalization
- On simulated data at known enrichment (e.g., 24 15N metabolites at 20% theoretical enrichment, resolution 140,000), modified MDT and ULS yield significantly lower enrichment RMSE than raw FAM, NRE, and the original MDT/ULS implementations
- For 34S simulations at resolution 140,000, original ULS RMSE exceeds the NRE RMSE because 32S natural abundance is 95.0% versus 99.6% for 14N
- Outputs are automatically written as additional sheets (MDV, enrichment, pool-size fold change) in the original XLSX input file
- Example files process within roughly 10 seconds on a 2.6 GHz Core i7; results taking longer than a minute usually indicate the wrong tracer was selected

## Limitations

- Developed and distributed under MATLAB (MATLAB 2016b environment); running the compiled executable requires installing MATLAB Compiler Runtime, which may limit reproducibility
- ULS requires an additional experimental measurement of unlabeled samples, so MDT is recommended for small metabolites on cost grounds
- Skipping the unlabeled-data step means ElemCor runs based on mass difference theory only (MDT without ULS)
- Original ULS underperforms for tracers whose most abundant isotope has lower fractional abundance (e.g., 32S at 95.0%), so the modified implementation must be used
- Labeled and unlabeled XLSX files must contain the same analytes and the same number of rows, and must be placed in the same directory as the executable

## Evidence

- [abstract] Resolution-dependent correction on high-res analyzers: "Here we present a software tool, ElemCor, to perform correction of such data in a resolution-dependent manner."
- [discussion] MDV computation scheme: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [discussion] ULS deconvolution definition: "one can obtain the correction matrix for natural abundance by deconvolution of the fractional natural abundance of the tracer element Nt times from the FAM from an unlabeled sample."
- [readme] README test files and simulation provenance: "They are simulated from XCalibur with N and S as trace elements, and at nominal instrument resolutions of 140,000 and 280,000, respectively. Both are from Orbitrap analyzer."
- [readme] README input format: "The software tool takes data in a spreadsheet (XLSX) format. In Steps 1 and 2, labeled and unlabeled data are loaded."
- [readme] IsoCor comparison rationale: "IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor."
