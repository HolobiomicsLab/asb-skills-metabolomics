---
name: exact-mass-calculation-from-chemical-formula
description: Use when processing high-resolution GC-CI-MS (e.g. GC-Orbitrap) stable-isotope
  tracing data and you need theoretical m/z values for a derivatized compound formula
  or its isotopologues to match detected peaks.
when_to_use_negative:
- Data are low-resolution (QqQ SIM/NS) isotopologue assignments, where isoSCAN uses
  nominal-mass windows rather than formula-based exact m/z lookups
- Only raw chromatographic peak intensities are needed and no formula-derived m/z
  targets exist
- The compound formula is unknown or the derivatization chemistry is unverified
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: enviPat
  role: R package providing the isotope table and isopattern/resolution functions
    used to predict isotopologue m/z values and their resolvability from the derivatized
    formula
- name: isoSCAN
  role: R tool that performs the formulaTable-based m/z lookup for high-resolution
    GC-CI-MS data, using exact mass calculations to assign isotopologues
  repo: https://github.com/jcapelladesto/isoSCAN
- name: R
  role: Runtime environment for both enviPat and isoSCAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:106475d9e322d78d3ee6c49b007f926de4bd83052fe4247431cc8e80c3af9650
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - enviPat
  - isoSCAN
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Exact-mass calculation from chemical formula

## Summary

Computes the theoretical monoisotopic m/z (and full isotopologue m/z envelope) of a derivatized GC-CI-MS metabolite formula, e.g. Val C11H28NO2Si2, using elemental masses such as those shipped in enviPat's isotopes table, to anchor isoSCAN's high-resolution isotopologue assignments.

## When to use

Use when processing high-resolution GC-CI-MS (e.g. GC-Orbitrap) stable-isotope tracing data and you need theoretical m/z values for a derivatized compound formula or its isotopologues to match detected peaks.

## When NOT to use

- Data are low-resolution (QqQ SIM/NS) isotopologue assignments, where isoSCAN uses nominal-mass windows rather than formula-based exact m/z lookups
- Only raw chromatographic peak intensities are needed and no formula-derived m/z targets exist
- The compound formula is unknown or the derivatization chemistry is unverified

## Inputs

- Derivatized compound chemical formula (e.g. C11H28NO2Si2 for silylated valine)
- Instrument resolving power setting (e.g. 60,000 at 200 m/z)
- Element isotope mass/abundance table (enviPat isotopes table)

## Outputs

- Theoretical monoisotopic m/z of the [M+H]+ ion
- Isotopologue m/z list with relative abundances spanning M+0 to M+n (e.g. M+0 through M+11 for 13C)
- Per-isotopologue resolvability flags at the instrument resolution

## How to apply

Take the derivatized molecular formula of the target compound (e.g. the [M+H]+ form of valine, C11H28NO2Si2, after methoxyamination/silylation) and compute exact m/z from monoisotopic elemental masses, such as those in enviPat's isotopes table. For high-resolution data, isoSCAN uses enviPat's isopattern function to generate the full theoretical isotopologue envelope (each isotopologue's m/z and relative abundance, spanning M+0 through the maximum label incorporation). Then assess resolvability by feeding the resolution setting of the instrument (e.g. 60,000 at 200 m/z on the GC-Orbitrap) into enviPat's resolution check, flagging which neighboring isotopologue peaks are resolved. These theoretical values become the lookup targets that isoSCAN matches against detected peak m/z in HRMS data, analogous to the formulaTable-based m/z lookup it performs internally.

## Related tools

- **enviPat** (R package providing the isotope table and isopattern/resolution functions used to predict isotopologue m/z values and their resolvability from the derivatized formula)
- **isoSCAN** (R tool that performs the formulaTable-based m/z lookup for high-resolution GC-CI-MS data, using exact mass calculations to assign isotopologues) — https://github.com/jcapelladesto/isoSCAN
- **R** (Runtime environment for both enviPat and isoSCAN)

## Examples

```
install_github("jcapelladesto/isoSCAN"); library(isoSCAN)  # then supply the derivatized formula and resolution for enviPat-based isotopologue m/z prediction
```

## Evaluation signals

- The predicted monoisotopic [M+H]+ m/z of valine must match the theoretical value derivable from formula C11H28NO2Si2 within the instrument's resolution tolerance
- The isotopologue envelope must span the expected 13C incorporation series (M+0 through M+11) with abundances consistent with natural isotope ratios
- Neighboring isotopologue peaks flagged as resolvable must be consistent with the configured resolving power (e.g. 60,000 at 200 m/z)
- Computed m/z values should match those found by isoSCAN's formulaTable lookup when run on the same formula

## Limitations

- Exact-mass-based assignment only applies to high-resolution instruments; low-resolution QqQ SIM/NS data rely on nominal-mass windows instead
- Prediction accuracy depends on the correctness of the derivatized formula (methoxyamination + silylation products); an incorrect formula shifts all m/z targets
- Resolvability flags are resolution-dependent and must be recomputed when the instrument resolving power changes

## Evidence

- [results] enviPat-based prediction of isotopologue m/z: [evidence span withheld — anchor status: not_found]
- [intro] isoSCAN is an R tool for GC-CI-MS labeling data: [evidence span withheld — anchor status: not_found]
- [readme] Installation instructions from README: "R Package to automatically process GC-CI-MS data for stable isotope labelling experiments"
