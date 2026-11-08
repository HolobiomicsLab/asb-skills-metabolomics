---
name: isotope-pattern-prediction-from-molecular-formula
description: Use when defining target metabolites for MS-based targeted metabolic
  or stable-isotope tracer analysis and you need theoretical isotope patterns and
  per-isotopologue m/z ranges from a formula/adduct/resolution.
when_to_use_negative:
- Raw EIC data already acquired for known targets where quantification (airPLS baseline
  removal, MassSpecWavelet peak detection, PAFFT alignment) is the next step rather
  than target definition
- Analyses lacking a defined molecular formula/adduct, since enviPat requires a formula
  to compute the pattern
- Isotopologue extraction or quantitative analysis of previously defined targets —
  those are TarMet steps 2.2 and 2.3, not target definition
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TarMet
  role: Shiny application providing getIsotopicPattern and getMzWithTracer to define
    target metabolite isotope patterns and tracer isotopologue m/z ranges
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Package used by TarMet to calculate the theoretical isotope patterns of compounds
    from the MS resolution
- name: shiny
  role: Reactive GUI framework hosting the TarMet target-definition workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:f220c9f10f1cdb2c9175f4ffe691dacb6a41f0826142a36e4b498fbfbf1a38e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# isotope-pattern-prediction-from-molecular-formula

## Summary

Compute the theoretical isotope pattern of a target metabolite from its molecular formula and adduct at the MS instrument's resolution, plus tracer-assay m/z windows for isotopologues with 0..N incorporated tracer atoms, using TarMet's getIsotopicPattern and getMzWithTracer backed by the enviPat package.

## When to use

Use when defining target metabolites for MS-based targeted metabolic or stable-isotope tracer analysis and you need theoretical isotope patterns and per-isotopologue m/z ranges from a formula/adduct/resolution.

## When NOT to use

- Raw EIC data already acquired for known targets where quantification (airPLS baseline removal, MassSpecWavelet peak detection, PAFFT alignment) is the next step rather than target definition
- Analyses lacking a defined molecular formula/adduct, since enviPat requires a formula to compute the pattern
- Isotopologue extraction or quantitative analysis of previously defined targets — those are TarMet steps 2.2 and 2.3, not target definition

## Inputs

- Target compound definition (molecular formula and adduct)
- MS instrument resolution
- Tracer isotope specification and maximum number of isotopologues (N) for tracer assays

## Outputs

- Theoretical isotope-pattern table for the compound at the given resolution
- m/z-range table listing each tracer isotopologue (0..N incorporated tracer atoms) with its expected m/z window

## How to apply

Load the target compound definition (molecular formula and adduct) into the TarMet shiny environment, then call getIsotopicPattern to calculate the theoretical isotope pattern of the compound at the resolution of the MS instrument, relying on the enviPat package for the pattern computation. For tracer assays, specify the tracer isotope and the maximum number of isotopologues, then use getMzWithTracer to compute the m/z series of the isotopologues with 0..N incorporated tracer atoms, deriving expected m/z windows for each isotopologue. Write out the theoretical isotope-pattern table and the m/z-range table as the target-definition outputs consumed by the downstream isotopologues analysis (2.2) and quantitative analysis (2.3) modules. Validate that each tracer isotopologue row carries an m/z range whose per-tracer-atom shift is consistent with the enviPat theoretical pattern.

## Related tools

- **TarMet** (Shiny application providing getIsotopicPattern and getMzWithTracer to define target metabolite isotope patterns and tracer isotopologue m/z ranges) — https://github.com/hcji/TarMet
- **enviPat** (Package used by TarMet to calculate the theoretical isotope patterns of compounds from the MS resolution)
- **shiny** (Reactive GUI framework hosting the TarMet target-definition workflow)

## Examples

```
library(TarMet); runTarMet()  # then in the shiny GUI load formula/adduct and call getIsotopicPattern / getMzWithTracer
```

## Evaluation signals

- The isotope-pattern table and m/z-range table files are both produced as target-definition outputs
- Each tracer isotopologue row carries the expected m/z range
- m/z shifts per incorporated tracer atom are consistent with the theoretical pattern computed by enviPat
- Pattern is computed at the resolution of the actual MS instrument (resolution mismatch would shift expected windows)
- Isotopologue series spans exactly 0..N incorporated tracer atoms as specified by the user

## Limitations

- Tracer m/z computation requires the user to first specify the tracer isotope and maximum number of isotopologues; it cannot proceed without those choices
- Planned but not yet supported features include data-independent acquisition and MS/MS library verification
- TarMet is a GUI-centric (shiny) workflow; command-line use is documented separately in the package vignettes

## Evidence
