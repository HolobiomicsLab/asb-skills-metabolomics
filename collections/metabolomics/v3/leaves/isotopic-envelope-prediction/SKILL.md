---
name: isotopic-envelope-prediction
description: Use when processing high-resolution (HRMS) GC-CI-MS stable-isotope labeling
  data (e.g. GC-Orbitrap) and you need the theoretical [M+H]+ isotopologue m/z values,
  abundances, and resolvability for a derivatized formula before quantifying isotopologues
  with isoSCAN.
when_to_use_negative:
- When working with low-resolution (LRMS) GC-CI-MS data such as triple quadrupole
  (QqQ) SIM/narrow-window scan acquisitions — isoSCAN handles these LRMS datasets
  directly without the enviPat envelope-prediction step
- When the input is already a measured peak list or chromatogram from the MS run rather
  than a chemical formula — this skill predicts the theoretical envelope from a formula
  and is not a peak-picking or quantification tool
- When the compound is un-derivatized and its formula does not include the silyl/methoxyamino
  moieties introduced by the derivatization protocol — the predicted m/z values will
  not match the measured [M+H]+ ions
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: enviPat
  role: R package used by isoSCAN to compute the theoretical isotopologue pattern
    (isopattern) and assess peak resolvability at the instrument resolution for the
    derivatized formula
- name: isoSCAN
  role: R package that calls the enviPat envelope prediction as part of its processing
    of GC-CI-MS HRMS data and assembles predicted m/z values, abundances, and resolvability
    flags
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment in which isoSCAN and enviPat are executed to generate
    the isotopologue m/z predictions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:a9b4f41993fcf79b4ef5e956b5ca593b2e4594801a1be60e35f8cd380ba2dadd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# isotopic-envelope-prediction

## Summary

Predicts the theoretical isotopologue m/z series and relative abundances for a derivatized metabolite formula using enviPat, and flags which neighboring peaks are resolvable at a given GC-Orbitrap resolving power, as implemented in isoSCAN for high-resolution GC-CI-MS stable-isotope labeling data.

## When to use

Use when processing high-resolution (HRMS) GC-CI-MS stable-isotope labeling data (e.g. GC-Orbitrap) and you need the theoretical [M+H]+ isotopologue m/z values, abundances, and resolvability for a derivatized formula before quantifying isotopologues with isoSCAN.

## When NOT to use

- When working with low-resolution (LRMS) GC-CI-MS data such as triple quadrupole (QqQ) SIM/narrow-window scan acquisitions — isoSCAN handles these LRMS datasets directly without the enviPat envelope-prediction step
- When the input is already a measured peak list or chromatogram from the MS run rather than a chemical formula — this skill predicts the theoretical envelope from a formula and is not a peak-picking or quantification tool
- When the compound is un-derivatized and its formula does not include the silyl/methoxyamino moieties introduced by the derivatization protocol — the predicted m/z values will not match the measured [M+H]+ ions

## Inputs

- Derivatized metabolite molecular formula as a string (e.g. 'C11H28NO2Si2' for the silylated valine derivative Val, accounting for methoxyamination and MSTFA+1%TMCS silylation)
- List of chemical elements present in the formula (e.g. C, H, N, O, Si) for subsetting the enviPat isotopes table
- Mass analyzer resolving power setting (e.g. 60,000 at 200 m/z for the GC-Orbitrap high-resolution configuration)

## Outputs

- Table of predicted isotopologue m/z values spanning the 13C incorporation series (e.g. M+0 through M+11) with relative abundances
- Flags indicating which neighboring isotopologue peaks are resolved by the mass analyzer at the specified resolving power
- Data frame written to a file such as isotopologue_mz_predictions.csv for downstream isoSCAN quantification

## How to apply

Load the isotopes table shipped with the enviPat R package and subset it to the elements present in the derivatized compound formula (e.g. Val C11H28NO2Si2 → C, H, N, O, Si). Run enviPat's isopattern function on the formula to generate the theoretical isotopologue pattern, yielding each isotopologue's nominal mass and relative abundance for the [M+H]+ envelope. Compute the exact m/z of each predicted isotopologue using the monoisotopic masses in the enviPat isotopes table, emulating the formulaTable-based m/z lookup isoSCAN performs for high-resolution data. Apply enviPat's resolution assessment at the instrument resolving power (e.g. 60,000 at 200 m/z on the GC-Orbitrap) to flag which neighboring isotopologue peaks are resolved. The rationale is that CI with isobutane concentrates metabolite abundance in the intact [M+H]+ isotopic envelope, so the predicted envelope defines which m/z values isoSCAN integrates from the HRMS full-scan data.

## Related tools

- **enviPat** (R package used by isoSCAN to compute the theoretical isotopologue pattern (isopattern) and assess peak resolvability at the instrument resolution for the derivatized formula)
- **isoSCAN** (R package that calls the enviPat envelope prediction as part of its processing of GC-CI-MS HRMS data and assembles predicted m/z values, abundances, and resolvability flags) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment in which isoSCAN and enviPat are executed to generate the isotopologue m/z predictions)

## Examples

```
install.packages("devtools", dependencies=TRUE); library(devtools); install_github("jcapelladesto/isoSCAN"); library(isoSCAN)
```

## Evaluation signals

- The predicted [M+H]+ monoisotopic m/z of a reference compound (e.g. Val, formula C11H28NO2Si2) must match the theoretical value derivable from the formula within the resolution tolerance of the instrument setting
- The predicted envelope must span the expected 13C incorporation series (M+0 through M+11 for Val) with abundances consistent with natural isotope ratios from the enviPat isotopes table
- Resolvability flags at the GC-Orbitrap setting (e.g. 60,000 at 200 m/z) must correctly separate isotopologues that differ by ~1 Da at that resolving power
- Predicted m/z values should align with the isotopologue positions that isoSCAN subsequently quantifies from the HRMS full-scan data
- The output data frame must contain one row per predicted isotopologue with columns for m/z, relative abundance, and resolvability flag, with no missing values for the elements present in the formula

## Limitations

- Envelope prediction depends on the correctness of the derivatized formula — if the methoxyamination (methoxyamine in pyridine, 45 min at 60 °C) and silylation (MSTFA + 1% TMCS, 30 min at 60 °C) stoichiometry is wrong, the predicted m/z series will be offset
- The resolvability assessment is specific to the configured resolving power (e.g. 60,000 at 200 m/z on the GC-Orbitrap); different instruments or settings require re-running the resolution check
- This prediction is designed for high-resolution GC-CI-MS workflows where the [M+H]+ ion is the dominant species under isobutane chemical ionization; it does not account for in-source fragmentation seen under EI at 70 eV

## Evidence

- [readme] enviPat as the isotopologue envelope predictor: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
- [intro] isoSCAN processes GC-CI-MS HRMS data: [evidence span withheld — anchor status: not_found]
- [results] Isobutane CI yields intact [M+H]+ ions: [evidence span withheld — anchor status: not_found]
- [results] IsoSCAN uses enviPat for high-resolution envelope prediction: [evidence span withheld — anchor status: not_found]
- [methods] Instrument resolving power setting: "high resolution GC-(q)TOF and GC-Orbitrap systems"
