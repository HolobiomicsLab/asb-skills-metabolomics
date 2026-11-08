---
name: spectral-agreement-icc-benchmarking
description: Use when validating that an accelerated or reimplemented MRS basis-set
  simulation produces metabolite basis functions statistically equivalent to a reference
  baseline, requiring per-metabolite ICC thresholds before acceptance.
when_to_use_negative:
- Comparing a simulated basis set to the LCModel phantom-acquired vendor basis set
  as a pass/fail test — ICCs there are expected to be lower (0.68–0.96) because waveforms
  and timings are undocumented
- The sequence of interest is not covered by MRSCloud (e.g., STEAM or SPECIAL localization,
  which are not implemented), so no matched baseline spectra can be generated
- Input is raw in-vivo MRS data rather than simulated metabolite basis functions —
  ICC benchmarking applies to simulated basis spectra, not measured spectra
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MRSCloud
  role: Generates the candidate density-matrix-simulated basis set whose spectra are
    ICC-benchmarked against baselines
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Elementwise simulation toolbox providing the reference baseline basis spectra
    for the ICC >= 0.98 comparison
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Alternative simulator used as a second baseline for ICC comparison (ICCs >=
    0.96)
- name: LCModel
  role: Source of the experimentally measured phantom basis set used as an external
    agreement comparator
- name: R
  role: Statistical environment in which per-metabolite ICCs are computed in RStudio
- name: RStudio
  role: IDE used to perform the ICC statistical analysis in R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:9835e2e0e7db1e251bee4f64c33f1760ecd60b34517fce7ae2b039636dcaaa62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shui5/MRSCloud
  tools:
  - FID-A
  - LCModel
  - MARSS
  - MRSCloud
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-agreement-icc-benchmarking

## Summary

Quantifies agreement between density-matrix-simulated metabolite basis spectra (e.g., from MRSCloud) and baseline simulated or experimentally acquired basis sets by computing per-metabolite intraclass correlation coefficients (ICCs) in R/RStudio, using an ICC >= 0.98 acceptance threshold against the FID-A elementwise baseline.

## When to use

Use when validating that an accelerated or reimplemented MRS basis-set simulation produces metabolite basis functions statistically equivalent to a reference baseline, requiring per-metabolite ICC thresholds before acceptance.

## When NOT to use

- Comparing a simulated basis set to the LCModel phantom-acquired vendor basis set as a pass/fail test — ICCs there are expected to be lower (0.68–0.96) because waveforms and timings are undocumented
- The sequence of interest is not covered by MRSCloud (e.g., STEAM or SPECIAL localization, which are not implemented), so no matched baseline spectra can be generated
- Input is raw in-vivo MRS data rather than simulated metabolite basis functions — ICC benchmarking applies to simulated basis spectra, not measured spectra

## Inputs

- Simulated metabolite basis spectra from MRSCloud (run_simulations_cloud.m output, e.g., PRESS/sLASER 3T basis set)
- Baseline simulated basis spectra from FID-A elementwise simulation (CIC-methods/FID-A) or MARSS
- Optionally, an experimentally acquired phantom basis set (e.g., LCModel press_te30_3t_01a.basis) for comparison
- Sequence configuration from simMRS.json (metabolite list, vendor, TE, editTp, spatial_points)

## Outputs

- Per-metabolite ICC table (e.g., propagator_benchmark.csv) with ICCs and runtimes for each implementation
- Accept/reject decision per metabolite against the ICC >= 0.98 (vs FID-A) threshold
- Optional QA PDF image of the spectra bundled with each zipped MRSCloud basis set

## How to apply

Generate the candidate basis set (e.g., run run_simulations_cloud.m from MRSCloud, which uses the 1D projection method, coherence order filtering, and pre-calculated propagators) and the baseline spectra (elementwise FID-A or MARSS) with matched parameters such as editTp = 14 ms for MEGA-PRESS/HERMES or 20 ms for HERCULES, TE = 68/80 ms for HERMES/HERCULES, and spatial_points between 41 (acceptable) and 101 (ideal). Align the simulated metabolite basis functions across the two implementations per metabolite, then perform statistical analysis in R in RStudio, computing intraclass correlation coefficients per metabolite (e.g., Gln, Glu, NAA, GABA, Cystat, HCar, Lys, Glc) and writing a per-metabolite ICC/runtime table such as propagator_benchmark.csv. Accept the accelerated implementation only if ICCs versus the FID-A baseline reach at least 0.98 for every metabolite; treat 0.96 (the MRSCloud-vs-MARSS level) as the reference scale for alternative baselines, and interpret lower ICCs (e.g., 0.68 for Gln against the LCModel phantom-measured basis set) as evidence of parameter mismatch rather than simulation error.

## Related tools

- **MRSCloud** (Generates the candidate density-matrix-simulated basis set whose spectra are ICC-benchmarked against baselines) — https://github.com/shui5/MRSCloud
- **FID-A** (Elementwise simulation toolbox providing the reference baseline basis spectra for the ICC >= 0.98 comparison) — https://github.com/CIC-methods/FID-A
- **MARSS** (Alternative simulator used as a second baseline for ICC comparison (ICCs >= 0.96))
- **LCModel** (Source of the experimentally measured phantom basis set used as an external agreement comparator)
- **R** (Statistical environment in which per-metabolite ICCs are computed in RStudio)
- **RStudio** (IDE used to perform the ICC statistical analysis in R)

## Examples

```
Rscript -e 'library(irr); d <- read.csv("propagator_benchmark.csv"); icc(d[,c("MRSCloud","FIDA")], model="twoway", type="agreement")'
```

## Evaluation signals

- ICCs between the candidate basis spectra and the FID-A elementwise baseline are at least 0.98 for every metabolite
- ICCs against MARSS are at least 0.96, matching the reported MRSCloud-vs-MARSS agreement level
- Per-metabolite ICCs and runtimes are recorded in propagator_benchmark.csv covering all metabolites in the configured simMRS.json list
- ICCs against the LCModel acquired basis set fall within the reported plausible range (lowest Gln 0.68, highest NAA 0.96), not used as a pass/fail criterion
- Each zipped downloaded basis set includes a PDF image of the spectra for quality assurance

## Limitations

- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, and especially Glc) dominate simulation time, so benchmarking runs over these metabolites are slow
- Comparison with the LCModel vendor-provided basis set is hindered because its waveforms and timings are not well documented and it is likely phantom-measured rather than simulated
- MRSCloud supports only 3T PRESS/sLASER simulations with a limited parameter set, so agreement benchmarks are valid only within that scope
- Spin-system parameters rest largely on the work of Govindaraju et al. with minor updates, so ICC agreement reflects shared parameter assumptions rather than independent validation

## Evidence
