---
name: spectral-nrms-benchmark-comparison
description: Use when a newly simulated vendor-/sequence-specific MRS basis set (e.g.
  Philips PRESS TE 30 ms) must be quantitatively benchmarked against an existing reference
  basis set before use in LCModel-style metabolite quantification.
when_to_use_negative:
- When the goal is absolute metabolite quantification from in vivo spectra rather
  than basis-set-to-basis-set validation — ICC/NRMS here quantify agreement between
  simulated basis functions, not fitting accuracy.
- 'When the reference basis set''s acquisition parameters are undocumented (e.g. LCModel
  vendor-provided press_te30_3t_01a.basis): the article notes comparison is challenging
  because waveforms and timings are not well documented and likely differ from the
  MRSCloud simulation.'
- When the target sequence is not supported by MRSCloud (e.g. STEAM or SPECIAL localization
  are not implemented; only 3T PRESS/sLASER UnEdited/MEGA/HERMES/HERCULES variants).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: MRSCloud
  role: Generates the test density-matrix-simulated basis set via run_simulations_cloud.m
    from FID-A-based cloud simulation
  repo: https://github.com/shui5/MRSCloud
- name: FID-A
  role: Underlying MATLAB toolbox providing the simulation functionality and basis-set
    data structures on which MRSCloud is built
  repo: https://github.com/CIC-methods/FID-A
- name: MARSS
  role: Source of the reference (benchmark) metabolite basis functions used for ICC/NRMS
    comparison
- name: R
  role: Statistical computation of per-metabolite ICCs and normalized RMS differences
- name: RStudio
  role: Environment in which the R statistical analysis is performed
- name: LCModel
  role: Provider of an experimentally measured (phantom-acquired) basis set used as
    an alternative comparison target
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.29370
    title: '<scp>MRSCloud</scp>: A cloud‐based<scp>MRS</scp>tool for basis set simulation'
  source_content_hash: sha256:b44cc388a97c8e70191295784ae7e701f37b0a385b21c3863470d5da9710f678
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# spectral-nrms-benchmark-comparison

## Summary

Quantitatively validate a simulated MRS metabolite basis set (e.g. MRSCloud, built on FID-A) against a reference basis set (e.g. MARSS or an acquired LCModel set) by computing per-metabolite intraclass correlation coefficients (ICCs) and normalized root-mean-square differences (NRMS) on aligned spectral grids.

## When to use

Use when a newly simulated vendor-/sequence-specific MRS basis set (e.g. Philips PRESS TE 30 ms) must be quantitatively benchmarked against an existing reference basis set before use in LCModel-style metabolite quantification.

## When NOT to use

- When the goal is absolute metabolite quantification from in vivo spectra rather than basis-set-to-basis-set validation — ICC/NRMS here quantify agreement between simulated basis functions, not fitting accuracy.
- When the reference basis set's acquisition parameters are undocumented (e.g. LCModel vendor-provided press_te30_3t_01a.basis): the article notes comparison is challenging because waveforms and timings are not well documented and likely differ from the MRSCloud simulation.
- When the target sequence is not supported by MRSCloud (e.g. STEAM or SPECIAL localization are not implemented; only 3T PRESS/sLASER UnEdited/MEGA/HERMES/HERCULES variants).

## Inputs

- MRSCloud-simulated metabolite basis set (FID-A format; e.g. unedited Philips PRESS, TE 30 ms)
- Reference basis set (e.g. MARSS-generated Philips PRESS TE 30 ms basis set, or LCModel .basis file such as press_te30_3t_01a.basis)
- Metabolite list from simMRS. (e.g. Asc, Asp, Cr, EA, GABA, GPC, GSH, Gln, Glu, Gly, H2O, Lac, mI, NAA, NAAG, PCh, PCr, PE, Ser, sI, Tau)

## Outputs

- Per-metabolite ICC values (e.g. mrscloud_marss_icc_nrms.csv)
- Per-metabolite normalized root-mean-square difference (NRMS) values
- Pass/fail assessment per metabolite against ICC >= 0.96 and NRMS < 9% thresholds

## How to apply

Simulate the test basis set with MRSCloud's run_simulations_cloud.m (configure simMRS.: vendor, UnEdited, PRESS, TE 30 ms, spatial_points 101 ideal / 41 acceptable, private parameters unchanged), and obtain the reference set (e.g. MARSS Philips PRESS TE 30 ms basis set from the Juchem lab site). Load both sets, align each metabolite's simulated basis function onto a common spectral grid in MATLAB/FID-A, then compute per-metabolite ICCs and normalized RMS differences in R/RStudio. Accept the comparison when every metabolite reaches ICC >= 0.96 (MRSCloud vs MARSS threshold; >= 0.98 vs FID-A) and NRMS < 9%, tolerating larger discrepancies (NRMS 5-8%) for strongly J-coupled spins such as GABA, Gln, and Glu. Write the per-metabolite ICC and NRMS values to a CSV (e.g. mrscloud_marss_icc_nrms.csv) for reproducibility.

## Related tools

- **MRSCloud** (Generates the test density-matrix-simulated basis set via run_simulations_cloud.m from FID-A-based cloud simulation) — https://github.com/shui5/MRSCloud
- **FID-A** (Underlying MATLAB toolbox providing the simulation functionality and basis-set data structures on which MRSCloud is built) — https://github.com/CIC-methods/FID-A
- **MARSS** (Source of the reference (benchmark) metabolite basis functions used for ICC/NRMS comparison)
- **R** (Statistical computation of per-metabolite ICCs and normalized RMS differences)
- **RStudio** (Environment in which the R statistical analysis is performed)
- **LCModel** (Provider of an experimentally measured (phantom-acquired) basis set used as an alternative comparison target)

## Examples

```
MATLAB: edit simMRS. (vendor=Philips, mega_or_hadam=UnEdited, localization=PRESS, TE=30, spatial_points=101) then run run_simulations_cloud.m; R: write.csv(icc_nrms, "mrscloud_marss_icc_nrms.csv")
```

## Evaluation signals

- Every metabolite ICC reaches at least 0.96 for MRSCloud vs MARSS (0.98 vs FID-A); values below these thresholds indicate a configuration mismatch.
- All metabolite NRMS values fall below 9%; NRMS of 5-8% is expected and tolerated only for strongly J-coupled spins (GABA, Gln, Glu).
- Simulation was run with spatial_points set to 101 (ideal) or at minimum 41, with private parameters left unchanged.
- Output CSV (mrscloud_marss_icc_nrms.csv) contains one ICC and one NRMS row per metabolite across the full metabolite list.
- Downloaded MRSCloud basis set includes the PDF image of the spectra (generated via the Osprey-integrated plot function) for quality assurance.

## Limitations

- MRSCloud balances ease of use against flexibility by limiting the number of user-specified parameters; simulations capture only the most commonly used vendor parameter set and cannot accurately capture all possible vendor experiments or hardware/protocol variations.
- Spin-system parameters rely largely on Govindaraju et al. with only minor updates (e.g. to GABA), so benchmark agreement is bounded by shared spin-parameter assumptions.
- Complex J-coupled spin systems (Cystat, HCar, iLe, Lys, Glc) dramatically increase simulation time; Glc takes extremely long.
- Comparison against the LCModel-measured basis set is inherently limited: waveforms and timings are not well documented, limited acquisition information is provided, and ICCs were lowest for Gln at 0.68 and highest for NAA at 0.96 — below the simulated-vs-simulated thresholds.
- MRSCloud was only focused and tested on brain MRS data at 3T.

## Evidence
