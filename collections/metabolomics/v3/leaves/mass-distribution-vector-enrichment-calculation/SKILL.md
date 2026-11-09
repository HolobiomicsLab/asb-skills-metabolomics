---
name: mass-distribution-vector-enrichment-calculation
description: Use when LC-MS isotope-labeling FAM data (XLSX column vectors of M+0..M+Nt
  isotopologs) must be corrected for natural abundance, isotopic impurity, and high-resolution
  effects (e.g. Orbitrap at 140,000/280,000) before enrichment can be reported.
when_to_use_negative:
- The compound formula does not contain the selected tracer element (e.g. 15N data
  loaded but C or S selected) — the algorithm will error and the tool will not run
  properly
- Instrument resolution is extremely low, where the resolution effect is negligible
  and IsoCor-style combinatorial NRE correction is equivalent — the extra MDT/ULS
  machinery adds no accuracy
- No unlabeled standards are available and the compounds are small — MDT-only mode
  must be used instead of ULS, since ULS requires additional measurement of unlabeled
  samples
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: ElemCor
  role: 'Primary tool: loads XLSX FAM data, builds resolution-dependent correction
    matrices (MDT/ULS), and computes MDVs and enrichment via its GUI or MATLAB functions'
  repo: github.com/4dsoftware/elemcor
- name: IsoCor
  role: Reference NRE combinatorial correction used as a lower-accuracy baseline for
    comparison at high resolution
- name: MATLAB
  role: Environment for the ElemCor MATLAB function that can be embedded into metabolic
    flux analysis pipelines (e.g. FiatFlux, INCA)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:670af0765e4abb2a353fdf6ae762c9941b35615482e0fcf6d480e1ba48d1dfd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemCor
  - IsoCor
  - MATLAB
  - MDT
  - ULS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-distribution-vector-enrichment-calculation

## Summary

Compute corrected mass distribution vectors (MDV) and isotopic enrichment from measured fractional abundances of measured isotopologs (FAM) by non-negative linear regression against a resolution-dependent correction matrix, as implemented in ElemCor (MDT/ULS) for high-resolution LC-MS isotope tracing.

## When to use

Use when LC-MS isotope-labeling FAM data (XLSX column vectors of M+0..M+Nt isotopologs) must be corrected for natural abundance, isotopic impurity, and high-resolution effects (e.g. Orbitrap at 140,000/280,000) before enrichment can be reported.

## When NOT to use

- The compound formula does not contain the selected tracer element (e.g. 15N data loaded but C or S selected) — the algorithm will error and the tool will not run properly
- Instrument resolution is extremely low, where the resolution effect is negligible and IsoCor-style combinatorial NRE correction is equivalent — the extra MDT/ULS machinery adds no accuracy
- No unlabeled standards are available and the compounds are small — MDT-only mode must be used instead of ULS, since ULS requires additional measurement of unlabeled samples

## Inputs

- Labeled-sample FAM table (XLSX, e.g. test_labelS_sim.xlsx), one column vector per sample with FAM of all compounds
- Optional unlabeled-sample FAM table (XLSX, e.g. test_unlabelS_sim.xlsx) with same analytes and rows
- Tracer element selection (13C, 2H, 15N, 18O, or 34S)
- Nutrient isotopic purity and nominal instrument resolution (e.g. 140,000) and mass analyzer (e.g. Orbitrap)

## Outputs

- Corrected MDVs stored in a new sheet of the same input XLSX file and previewed in the ElemCor figure window
- Isotopic enrichment per compound and sample
- Fold change of pool size per compound and sample

## How to apply

Load labeled and (optionally) unlabeled FAM tables in XLSX format where each column vector holds FAM for all compounds in one sample (FAM length is typically Nt+1, though other lengths are handled). Build the total correction matrix via mass difference theory (MDT) and/or deconvolution of the unlabeled-sample FAM (ULS), specifying tracer element (13C, 2H, 15N, 18O, or 34S), nutrient isotopic purity, mass analyzer (e.g. Orbitrap), and nominal resolution (e.g. 140,000). Obtain the MDV by non-negative linear regression of the correction-matrix equation followed by normalization to sum of 1, then compute isotopic enrichment per compound. If unlabeled data are skipped, ElemCor runs on MDT only; if the tracer element is absent from a compound formula the algorithm errors, so verify tracer selection first. As a baseline or cross-check, compare against the no-resolution-effect (NRE) combinatorial correction used by IsoCor — at very low resolution ElemCor is identical to IsoCor, and at very high resolution it performs almost no correction, so the correction magnitude is monotonically dependent on instrument resolution.

## Related tools

- **ElemCor** (Primary tool: loads XLSX FAM data, builds resolution-dependent correction matrices (MDT/ULS), and computes MDVs and enrichment via its GUI or MATLAB functions) — github.com/4dsoftware/elemcor
- **IsoCor** (Reference NRE combinatorial correction used as a lower-accuracy baseline for comparison at high resolution)
- **MATLAB** (Environment for the ElemCor MATLAB function that can be embedded into metabolic flux analysis pipelines (e.g. FiatFlux, INCA))

## Evaluation signals

- Normalized MDVs must sum to 1 after non-negative linear regression of the correction-matrix equation
- Corrected MDVs appear in a new sheet of the original input XLSX and match the values previewed in the figure window
- For 34S data at resolution 140,000, enrichment RMSE ordering should be: modified MDT and modified ULS < NRE < original ULS (original ULS is less accurate than NRE because 32S natural abundance is 95.0% vs 99.6% for 14N)
- Example files process within ~10 s on a 2.6 GHz Core i7 (20 s on 1.5 GHz Core i5); results taking >1 min usually indicate the wrong tracer was selected
- The tool should emit a warning if the wrong tracer is selected (e.g. 15N data loaded with C or S as tracer)

## Limitations

- The original ULS implementation is mathematically defective for tracers whose most abundant isotope has lower fractional abundance (32S at 95.0% vs 14N at 99.6%), making it less accurate than NRE for 34S; use ElemCor's modified ULS
- Requires MATLAB or MATLAB Compiler Runtime (developed under MATLAB 2016b); no software version/release identifier or license terms are stated in the article
- Only a general error message is printed; troubleshooting relies on checking tracer selection, XLSX location, and that labeled/unlabeled files have the same analytes and number of rows
- Experimental validation was limited to S. cerevisiae with 15N ammonium (20.0% 15N) on Orbitrap data

## Evidence

- [discussion] MDV via NNLS regression: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [methods] enrichment calculation step: "Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6."
- [readme] results stored in input file: "After correction, the MDVs of corresponding samples are shown in the figure window in ElemCor and also stored in a different sheet in the same input file."
- [readme] input format: "The files show standard format of input files. Each column vector correspond to FAM of all compounds in each sample."
- [readme] ElemCor vs IsoCor resolution dependence: "When instrument resolution is extremely low, ElemCor is identical to IsoCor. When instrument resolution is extremely high, ElemCor performs no correction at all."
