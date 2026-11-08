---
name: mass-spectrometry-correction-matrix-construction
description: Use when correcting FAM to MDV for natural abundance and tracer impurity
  on a high-resolution analyzer (e.g., Orbitrap at nominal resolution 100,000–280,000)
  where same-nominal-mass isotopologs may be resolved and full correction matrices
  are no longer justified.
when_to_use_negative:
- When instrument resolution is extremely low, ElemCor becomes identical to IsoCor's
  combinatorial correction without resolution effect — a plain full correction matrix
  suffices
- When the tracer element is not present in the compound formula (e.g., selecting
  C or S as tracer for 15N-only data), the algorithm errors out
- When unlabeled samples are available and spectral inaccuracy is large for high-mass
  compounds, the ULS-based approach may be preferable to MDT alone
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ElemCor
  role: Implements the resolution-dependent correction matrix construction, per-isotope
    exclusion limits, and non-negative linear regression correction of FAM to MDV
  repo: https://github.com/4dsoftware/elemcor
- name: MDT
  role: Mass difference theory underlying the Delta-M resolution criterion used to
    gate which same-nominal-mass isotopologs enter the correction matrix
  repo: https://github.com/4dsoftware/elemcor
- name: IsoCor
  role: Baseline combinatorial correction without resolution effect; coincides with
    ElemCor at extremely low resolution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-2669-9
    title: 'ElemCor: accurate data analysis and enrichment calculation for high-resolution
      LC-MS stable isotope labeling experiments'
  source_content_hash: sha256:be4197f2e69ed841b137654774ad1f9887a3179be29a962892e186f771e37ffb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/4dsoftware/elemcor
  tools:
  - ElemCor
  - IsoCor
  - MDT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-correction-matrix-construction

## Summary

Constructs a resolution-dependent isotope correction matrix for LC-MS stable isotope labeling data using mass difference theory (MDT): high-resolving-power instruments like the Orbitrap can resolve same-nominal-mass isotopologs, so isotopolog inclusion is gated by a mass-difference criterion rather than including all isotopologs as IsoCor does.

## When to use

Use when correcting FAM to MDV for natural abundance and tracer impurity on a high-resolution analyzer (e.g., Orbitrap at nominal resolution 100,000–280,000) where same-nominal-mass isotopologs may be resolved and full correction matrices are no longer justified.

## When NOT to use

- When instrument resolution is extremely low, ElemCor becomes identical to IsoCor's combinatorial correction without resolution effect — a plain full correction matrix suffices
- When the tracer element is not present in the compound formula (e.g., selecting C or S as tracer for 15N-only data), the algorithm errors out
- When unlabeled samples are available and spectral inaccuracy is large for high-mass compounds, the ULS-based approach may be preferable to MDT alone

## Inputs

- Compound molecular formula (e.g., C5H10N2O3 for glutamine)
- Nominal instrument resolution (e.g., 100,000; Orbitrap test files use 140,000 and 280,000) and mass analyzer selection
- Fractional abundances of measured isotopologs (FAM) in XLSX spreadsheet format, length Nt+1 covering M+0..M+Nt
- Isotopic purity of the nutrient (tracer purity)
- Tracer element selection (13C, 2H, 15N, 18O, or 34S)
- Optional: unlabeled-sample FAM (test_unlabelN_sim.xlsx / test_unlabelS_sim.xlsx) for ULS deconvolution

## Outputs

- Per-isotope exclusion limits and smallest resolvable Delta-M (e.g., mdt_correction_limits.csv)
- Resolution-dependent correction matrix containing only isotopologs with cumulative mass differences below the resolvable Delta-M
- Corrected mass distribution vectors (MDV) and isotopic enrichment per compound/sample
- Corrected MDVs written to a new sheet of the input XLSX file

## How to apply

Load the compound formula (e.g., glutamine C5H10N2O3) and the nominal instrument resolution (Step 3 of ElemCor), then compute the smallest resolvable mass difference with the Orbitrap criterion Delta-M >= 1.66*M^1.5/(R*sqrt(M_R)); at the monoisotopic mass of glutamine with R = 100,000 this is ~2.07e-3 Da. For each isotope relevant to the formula (13C, 15N, 2H, 17O, 18O), compute its mass difference from the monoisotopic peak and set the per-isotope exclusion limit to the largest count whose cumulative mass difference stays below the smallest resolvable Delta-M (e.g., limits of 2 for 17O and 0 for 15N, 2H, and 18O). Apply ElemCor's improvement over original MDT: enumerate isotope combinations and sum mass differences over each combination rather than excluding each isotope independently, so combinations like 2x17O + 1x18O whose cumulative Delta-M falls within the resolvable window are retained in the correction matrix. Build the total correction matrix from these included isotopologs (optionally deconvolving unlabeled-sample FAM per ULS), solve the correction via non-negative linear regression followed by normalization to sum 1 to obtain the MDV and isotopic enrichment. ElemCor's correction amount is monotonically dependent on resolution: identical to IsoCor at extremely low resolution and performing no correction at extremely high resolution.

## Related tools

- **ElemCor** (Implements the resolution-dependent correction matrix construction, per-isotope exclusion limits, and non-negative linear regression correction of FAM to MDV) — https://github.com/4dsoftware/elemcor
- **MDT** (Mass difference theory underlying the Delta-M resolution criterion used to gate which same-nominal-mass isotopologs enter the correction matrix) — https://github.com/4dsoftware/elemcor
- **IsoCor** (Baseline combinatorial correction without resolution effect; coincides with ElemCor at extremely low resolution)

## Examples

```
DeltaM_min = 1.66*M_mono**1.5/(R*np.sqrt(M_R))  # e.g. glutamine at R=100000 -> ~2.07e-3 Da, then gate isotopolog combos by cumulative Delta-M
```

## Evaluation signals

- Smallest resolvable Delta-M at glutamine's monoisotopic mass with R = 100,000 must equal ~2.07e-3 Da via Delta-M >= 1.66*M^1.5/(R*sqrt(M_R))
- Per-isotope limits must reproduce 2 for 17O and 0 for 15N, 2H, and 18O on the glutamine reference case
- The 2x17O + 1x18O combination must be included in the correction matrix (as in Fig. 5b) under the sum-of-mass-differences criterion, though 17O alone is limited to 2 and 18O alone to 0
- Amount of correction applied must vary monotonically with resolution: identical to IsoCor at extremely low resolution, no correction at extremely high resolution
- MDV obtained by non-negative linear regression must be normalized to sum 1 before enrichment calculation

## Limitations

- Original MDT's isotope-exclusion criterion was limited to tracers 13C, 2H, and 15N; ElemCor's improvement adds a small mass-difference correction delta balancing all isotopes
- FAM length other than Nt+1 is supported, but the tracer element must be present in every compound formula or the algorithm reports an error
- Selecting the wrong tracer (e.g., C or S for 15N data) causes the software to fail; a warning message prompts verification
- ULS mode requires additional experimental measurement of unlabeled samples, a cost limitation; MDT alone is recommended for low-mass compounds

## Evidence

- [intro] modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified.: "modern mass spectrometers with high resolving power can easily resolve isotopologs with the same nominal mass, and, thus, including all isotopologs in the correction matrix is no longer justified."
- [abstract] The tool is based on mass difference theory (MDT) and information from unlabeled samples (ULS) to account for resolution effects.: "The tool is based on mass difference theory (MDT) and information from unlabeled samples (ULS) to account for resolution effects."
- [methods] In Step 4, the tracer element is selected. In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction.: "In Step 4, the tracer element is selected. In addition to 13C, 2H, and 15N, ElemCor allows 18O and 34S as the tracer element for correction."
- [readme] IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor.: "IsoCor performs correction based on combinatorics without considering ressolution effect. When instrument resolution is extremely low, ElemCor is identical to IsoCor."
