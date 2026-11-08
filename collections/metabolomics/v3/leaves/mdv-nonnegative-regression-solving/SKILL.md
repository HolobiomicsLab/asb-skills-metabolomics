---
name: mdv-nonnegative-regression-solving
description: Use when a fractional-abundance vector (FAM) from an LC-MS isotope labeling
  experiment must be converted to a corrected, non-negative MDV and isotopic enrichment
  via an already-built total correction matrix (MDT and/or ULS).
when_to_use_negative:
- When the tracer element is not actually present in the compound's formula (e.g.
  C or S selected for a 15N-only compound) — the algorithm reports an error
- When the input is raw, uncorrected isotopologue intensities and no correction matrix
  has been built — this skill solves the correction equation, it does not construct
  the MDT/ULS matrix
- When instrument resolution is extremely high, ElemCor performs essentially no correction,
  so regression-based correction adds little
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: ElemCor
  role: Source of the MATLAB function that performs the non-negative regression of
    the correction-matrix equation and normalization to obtain the MDV and enrichment;
    also provides a GUI (Step 6) doing the same for each compound
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory method that supplies the total correction matrix C
    consumed by this regression step
- name: MATLAB
  role: Environment for ElemCor's provided regression function; adaptable into flux
    analysis pipelines such as FiatFlux or INCA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:75afb1e0450ca4d3e8a114d16b6a4ce695e4dbfd356cfc9ef9ef39ed46d58da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - MATLAB
  - MDT
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mdv-nonnegative-regression-solving

## Summary

Solves the correction-matrix equation C x' = z by non-negative linear regression (NNLS), then normalizes x' to sum to 1 to obtain the mass distribution vector (MDV), from which isotopic enrichment is computed as the tracer-count-weighted sum, exactly as ElemCor does for stable isotope labeling LC-MS data.

## When to use

Use when a fractional-abundance vector (FAM) from an LC-MS isotope labeling experiment must be converted to a corrected, non-negative MDV and isotopic enrichment via an already-built total correction matrix (MDT and/or ULS).

## When NOT to use

- When the tracer element is not actually present in the compound's formula (e.g. C or S selected for a 15N-only compound) — the algorithm reports an error
- When the input is raw, uncorrected isotopologue intensities and no correction matrix has been built — this skill solves the correction equation, it does not construct the MDT/ULS matrix
- When instrument resolution is extremely high, ElemCor performs essentially no correction, so regression-based correction adds little

## Inputs

- Total correction matrix C (MDT- and/or ULS-based, resolution-aware) for the compound and tracer element
- Measured fractional-abundance vector (FAM) z from a labeled sample, e.g. a column of test_labelN_sim.xlsx
- Tracer element and number of tracer atoms Nt (defines MDV length Nt + 1 covering M+0 ... M+Nt)

## Outputs

- Normalized MDV vector (non-negative entries summing to 1)
- Isotopic enrichment scalar (tracer-count-weighted sum of the MDV)
- Optional CSV of MDV entries and enrichment per compound (e.g. glutamine_mdv_enrichment.csv); in ElemCor, results are stored in an added sheet of the input XLSX

## How to apply

Load the total correction matrix C (constructed beforehand by MDT and/or ULS, per the article's Eq. 1) and the measured FAM vector z extracted from the labeled-sample spreadsheet (e.g. test_labelN_sim.xlsx). Solve C x' = z with a non-negative least squares solver — ElemCor ships a MATLAB function, and scipy.optimize.nnls is an equivalent reimplementation — because ordinary regression can yield negative isotopologue fractions that are physically meaningless. Normalize x' to sum to 1, as ElemCor does after regression, to yield the MDV over isotopologues M+0 ... M+Nt. Compute isotopic enrichment as the tracer-count-weighted sum, sum over i of i * x'_i. FAM length may differ from Nt + 1; the solver still applies, but the regression must follow, not precede, the resolution-aware correction matrix construction.

## Related tools

- **ElemCor** (Source of the MATLAB function that performs the non-negative regression of the correction-matrix equation and normalization to obtain the MDV and enrichment; also provides a GUI (Step 6) doing the same for each compound) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory method that supplies the total correction matrix C consumed by this regression step)
- **MATLAB** (Environment for ElemCor's provided regression function; adaptable into flux analysis pipelines such as FiatFlux or INCA)

## Examples

```
from scipy.optimize import nnls; x, _ = nnls(C, z); mdv = x / x.sum(); enrichment = sum(i * mdv[i] for i in range(len(mdv)))
```

## Evaluation signals

- Every MDV entry is non-negative (a direct consequence of the NNLS solution)
- The MDV sums to 1 within floating-point tolerance after the normalization step
- The enrichment equals sum(i * x'_i) over the normalized MDV and is a single scalar per compound/sample
- MDV length matches the FAM/correction-matrix dimensionality (typically Nt + 1 covering M+0 ... M+Nt, though other lengths are handled)
- Results match ElemCor's own output when run on the same test files (test_labelN_sim.xlsx with 15N tracer), where corrected MDVs are stored in a new sheet of the input file

## Limitations

- If the wrong tracer is selected (e.g. 15N data loaded but C or S chosen), the software will not run properly because compounds lacking the tracer element cause errors
- The regression depends on a correctly constructed resolution-aware correction matrix; at extremely low instrument resolution ElemCor reduces to IsoCor-style combinatorial correction
- ElemCor was developed under MATLAB (2016b), so reproducing the exact solver outside MATLAB relies on equivalent reimplementations such as scipy.optimize.nnls

## Evidence

- [discussion] Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor: "Non-negative linear regression of Eq. (1) followed by normalization to the sum of 1 is used to obtain MDV in ElemCor"
- [intro] FAM corrected to MDV: "the fractional abundance of measured isotopologs (FAM) collected from the instrument must be corrected to obtain MDV."
- [intro] MATLAB function adaptability: "since popular flux analysis software suites such as FiatFlux [11] and INCA [12] are also MATLAB-based, the provided MATLAB function of ElemCor can easily be adapted"
- [readme] FAM length convention: "FAM typically has a length of Nt + 1 to cover M+0, M+1, ..., M+Nt isotopologues."
- [readme] Enrichment calculated per compound: "Then the loaded data are analyzed and isotopic enrichment is calculated for each compound in Step 6."
- [readme] Wrong tracer failure mode: "If the 15N data are loaded, but C or S is selected as tracer, then the software tool would not run properly."
