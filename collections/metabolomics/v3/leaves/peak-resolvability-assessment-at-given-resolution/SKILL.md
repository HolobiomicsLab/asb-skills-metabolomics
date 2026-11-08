---
name: peak-resolvability-assessment-at-given-resolution
description: Use when processing high-resolution GC-CI-MS stable-isotope labeling
  data and you need to know which isotopologue m/z values of a derivatized formula
  can be distinguished at the instrument's resolution setting before quantifying them.
when_to_use_negative:
- Input is low-resolution GC-CI-MS data (e.g. from an Agilent 7000 QqQ in SIM or narrow-window
  scan mode), where isoSCAN does not use enviPat-based envelope prediction and resolvability
  is not assessed from the formula.
- The analyte's derivatized formula is unknown or incomplete — enviPat cannot predict
  the isotopologue envelope without the full elemental composition including derivatization
  atoms (Si, etc.).
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: enviPat
  role: R package used to generate the theoretical isotopologue pattern (isopattern)
    and assess which neighboring isotopologue peaks are resolved at a given resolution;
    also provides the isotopes table with monoisotopic masses used for the m/z lookup.
- name: isoSCAN
  role: R tool that performs this envelope-prediction and resolvability assessment
    as part of its high-resolution GC-CI-MS processing pipeline, using the formulaTable-based
    m/z lookup.
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:3321ff360e84cbc2457d2126c0e41a044ada4f2d182dfd3183ab7dc7578b7282
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# peak-resolvability-assessment-at-given-resolution

## Summary

Predicts the theoretical isotopologue envelope of a derivatized formula using enviPat in R, computing each isotopologue's exact m/z and flagging whether neighboring isotopologue peaks (e.g. the M+0 through M+11 13C series of [M+H]+ ions) are resolved at the mass analyzer's configured resolving power (e.g. 60,000 at 200 m/z on a GC-Orbitrap).

## When to use

Use when processing high-resolution GC-CI-MS stable-isotope labeling data and you need to know which isotopologue m/z values of a derivatized formula can be distinguished at the instrument's resolution setting before quantifying them.

## When NOT to use

- Input is low-resolution GC-CI-MS data (e.g. from an Agilent 7000 QqQ in SIM or narrow-window scan mode), where isoSCAN does not use enviPat-based envelope prediction and resolvability is not assessed from the formula.
- The analyte's derivatized formula is unknown or incomplete — enviPat cannot predict the isotopologue envelope without the full elemental composition including derivatization atoms (Si, etc.).

## Inputs

- Derivatized compound formula (e.g. C11H28NO2Si2 for methoxyamination/silylation-derivatized valine, including Si from MSTFA + 1% TMCS silylation)
- Mass analyzer resolving-power setting (e.g. 60,000 at 200 m/z from the GC-Orbitrap configuration)
- enviPat isotopes table (shipped with the enviPat R package)

## Outputs

- Data frame of predicted isotopologue exact m/z values, relative abundances, and per-peak resolvability flags at the given resolution
- isotopologue_mz_predictions.csv containing the predicted m/z values, relative abundances, and resolvability flags

## How to apply

Load the enviPat isotopes table and subset the elements of the derivatized formula (e.g. Val as C11H28NO2Si2 contains C, H, N, O, Si). Run enviPat's isopattern function on the formula to generate the theoretical isotopologue pattern, yielding each isotopologue's nominal mass and relative natural-abundance across the expected incorporation series (M+0 through M+11 for 13C tracing). Compute each predicted isotopologue's exact m/z from the monoisotopic masses in the enviPat isotopes table, emulating the formulaTable-based m/z lookup isoSCAN performs for high-resolution data. Apply enviPat's resolution assessment (isopattern / check_chemform with the resolution argument) at the instrument's resolving-power setting — e.g. 60,000 at 200 m/z as configured on the GC-Orbitrap — to flag which neighboring isotopologue peaks are resolved. Assemble predicted m/z values, relative abundances, and resolvability flags into a data frame; validation requires the predicted [M+H]+ monoisotopic m/z to match the theoretical value derivable from the formula within the instrument's tolerance, and the envelope to span the expected 13C incorporation series with abundances consistent with natural isotope ratios.

## Related tools

- **enviPat** (R package used to generate the theoretical isotopologue pattern (isopattern) and assess which neighboring isotopologue peaks are resolved at a given resolution; also provides the isotopes table with monoisotopic masses used for the m/z lookup.)
- **isoSCAN** (R tool that performs this envelope-prediction and resolvability assessment as part of its high-resolution GC-CI-MS processing pipeline, using the formulaTable-based m/z lookup.) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
library(isoSCAN); env <- enviPat::isopattern(isoSCAN::formulaTable("C11H28NO2Si2"), resolution = 60000, plot = FALSE)
```

## Evaluation signals

- The predicted [M+H]+ monoisotopic m/z of the analyte must match the theoretical value derivable from its formula (e.g. C11H28NO2Si2 for Val) within the resolution tolerance of the configured instrument setting.
- The predicted envelope must span the expected isotopologue incorporation series (M+0 through M+11 for a 13C-labeled compound like valine) with relative abundances consistent with natural isotope ratios.
- Neighboring isotopologue peaks flagged as unresolved at the configured resolving power (e.g. 60,000 at 200 m/z) should correspond to peaks whose predicted mass difference falls below the resolution-limited spacing at that m/z.
- The output table should contain columns for predicted exact m/z, relative abundance, and a resolvability flag for every isotopologue in the envelope, and be written to isotopologue_mz_predictions.csv without missing or malformed entries.

## Limitations

- The assessment depends on the accuracy of the derivatized formula; errors in the formula (e.g. missing Si atoms from silylation with MSTFA + 1% TMCS) propagate to incorrect m/z predictions and wrong resolvability flags.
- Resolvability is judged at a single configured resolving-power setting (e.g. 60,000 at 200 m/z); the flags do not generalize to other instruments or resolution modes without re-running the assessment.
- The skill addresses high-resolution GC-CI-MS data only; for low-resolution acquisitions (e.g. QqQ SIM mode) isoSCAN does not perform this enviPat-based envelope prediction.

## Evidence

- [methods] enviPat isopattern on derivatized formula: [evidence span withheld — anchor status: not_found]
- [results] isobutane reagent gas yields [M+H]+ envelope: [evidence span withheld — anchor status: not_found]
- [readme] isoSCAN repository: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
- [abstract] high-resolution GC-Orbitrap / GC-qTOF systems used: "standard full scan methods using high resolution GC-(q)TOF and GC-Orbitrap systems"
