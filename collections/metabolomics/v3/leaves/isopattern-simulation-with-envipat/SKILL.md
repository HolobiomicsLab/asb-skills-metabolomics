---
name: isopattern-simulation-with-envipat
description: Use when processing high-resolution GC-CI-MS (GC-qTOF or GC-Orbitrap)
  stable-isotope-labeling data and the expected isotopologue m/z values and their
  resolvability must be predicted from a derivatized formula prior to isotopologue
  quantification.
when_to_use_negative:
- Data are low-resolution (LRMS, e.g. QqQ in SIM mode), where isoSCAN does not use
  enviPat-based envelope prediction and selected ion clusters suffice
- Formula is unknown or cannot be reconstructed for the derivatized metabolite, so
  no isotopologue envelope can be simulated
- The task is chromatographic deconvolution rather than isotope-envelope prediction
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: enviPat
  role: R package supplying the isotopes table and the isopattern function used to
    simulate the theoretical isotopologue envelope and assess peak resolvability at
    a given resolving power
- name: isoSCAN
  role: R package whose high-resolution data path invokes enviPat-based isotopologue
    m/z prediction from the derivatized formula prior to isotopologue quantification
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:39bd2da37905b44b9a1d196342d28203959baa22d3ada54b79e223146f87e225
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# isopattern-simulation-with-envipat

## Summary

Uses the enviPat R package's isotope table and isopattern function to predict the theoretical isotopologue m/z values and relative abundances of a derivatized metabolite formula, and to assess which neighbouring peaks are resolved at a given mass-analyser resolving power before they are quantified from GC-CI-MS data.

## When to use

Use when processing high-resolution GC-CI-MS (GC-qTOF or GC-Orbitrap) stable-isotope-labeling data and the expected isotopologue m/z values and their resolvability must be predicted from a derivatized formula prior to isotopologue quantification.

## When NOT to use

- Data are low-resolution (LRMS, e.g. QqQ in SIM mode), where isoSCAN does not use enviPat-based envelope prediction and selected ion clusters suffice
- Formula is unknown or cannot be reconstructed for the derivatized metabolite, so no isotopologue envelope can be simulated
- The task is chromatographic deconvolution rather than isotope-envelope prediction

## Inputs

- Derivatized compound chemical formula, e.g. C11H28NO2Si2 for silylated Valine
- Mass-analyser resolving power setting, e.g. 60,000 at 200 m/z (GC-Orbitrap)
- enviPat isotope table (shipped with the enviPat R package)
- Element list present in the derivatized formula (C, H, N, O, Si)

## Outputs

- Data frame of predicted isotopologue m/z values, relative abundances, and per-peak resolvability flags
- CSV file of isotopologue m/z predictions, e.g. isotopologue_mz_predictions.csv
- [M+H]+ monoisotopic m/z for the target metabolite

## How to apply

Load the isotopes table shipped with the enviPat R package and subset the elements present in the derivatized compound formula (e.g. Val derivatized as C11H28NO2Si2: C, H, N, O, Si). Run enviPat's isopattern function on the formula to generate the theoretical [M+H]+ isotopologue pattern, yielding each isotopologue's mass and relative abundance spanning the expected 13C incorporation series (M+0 through M+11 for Val). Compute each predicted isotopologue's exact m/z from the monoisotopic masses in the enviPat isotopes table, emulating the formulaTable-based m/z lookup isoSCAN performs for high-resolution data. Apply enviPat's resolution assessment at the instrument's resolving power (e.g. 60,000 at 200 m/z as configured on the GC-Orbitrap) to flag which neighbouring isotopologue peaks are resolved. Assemble predicted m/z values, relative abundances and resolvability flags into a data frame that downstream isoSCAN-style peak-picking and integration can use to attribute chromatographic peaks to the correct isotopologue.

## Related tools

- **enviPat** (R package supplying the isotopes table and the isopattern function used to simulate the theoretical isotopologue envelope and assess peak resolvability at a given resolving power)
- **isoSCAN** (R package whose high-resolution data path invokes enviPat-based isotopologue m/z prediction from the derivatized formula prior to isotopologue quantification) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
install_github("jcapelladesto/isoSCAN"); library(isoSCAN); library(enviPat); data("isotopes"); iso <- isopattern("C11H28NO2Si2", isotopes = isotopes, resolution = 60000)
```

## Evaluation signals

- Predicted [M+H]+ monoisotopic m/z of the metabolite matches the theoretical value derivable from the derivatized formula within the instrument's resolution tolerance
- Simulated envelope spans the expected 13C incorporation series (e.g. M+0 through M+11 for Val) with relative abundances consistent with natural isotope ratios
- Per-isotopologue resolvability flags are consistent with the configured resolving power (e.g. 60,000 at 200 m/z)
- Predicted m/z values derived from the enviPat isotopes-table monoisotopic masses agree with the formulaTable-based m/z lookup performed by isoSCAN for high-resolution data

## Limitations

- Prediction requires an accurate derivatized formula; errors in the formula propagate to wrong isotopologue m/z and abundances
- Resolvability assessment depends on the resolution setting supplied (e.g. 60,000 at 200 m/z); a mismatched setting misclassifies neighbouring isotopologue peaks
- Applicable to high-resolution GC-CI-MS workflows (GC-qTOF/GC-Orbitrap); low-resolution data do not rely on this envelope prediction

## Evidence

- [other] For high-resolution data, isoSCAN uses the enviPat package to predict the isotopologue envelope of a (derivatized) formula at a given resolution: [evidence span withheld — anchor status: not_found]
- [intro] isoSCAN capable of processing GC-CI-MS data from LRMS and HRMS: [evidence span withheld — anchor status: not_found]
- [abstract] standard full scan methods using high resolution GC-(q)TOF and GC-Orbitrap systems: "standard full scan methods using high resolution GC-(q)TOF and GC-Orbitrap systems"
- [readme] README install_github jcapelladesto/isoSCAN: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
