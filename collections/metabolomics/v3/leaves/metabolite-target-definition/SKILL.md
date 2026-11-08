---
name: metabolite-target-definition
description: Use when building a targeted LC-MS assay in TarMet and you must define
  target metabolite m/z windows from a molecular formula/adduct, or derive isotopologue
  m/z series for a stable-isotope tracer with a chosen isotope and max isotopologues
  N.
when_to_use_negative:
- Raw EIC processing tasks (baseline removal, peak detection, retention-time alignment)
  — those belong to TarMet's later isotopologues/quantitative analysis steps, not
  target definition
- Tracer m/z series cannot be computed without the user first specifying the tracer
  isotope and maximum number of isotopologues
- Unbiased/untargeted feature discovery — this skill only generates targets for compounds
  with a known formula and adduct
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TarMet
  role: Reactive shiny application whose target-definition step computes theoretical
    isotope patterns and tracer m/z windows via getIsotopicPattern and getMzWithTracer
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Package TarMet relies on to calculate the theoretical isotope pattern of compounds
    from formula/adduct at the MS resolution
- name: shiny
  role: Provides the graphical user interface in which the target compound definition
    (formula, adduct) is loaded and tracer parameters are specified
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:649b80e15e149d09b46cf01a4966e9698b476bd0005840b971efd720b81642c6
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
yes_rate: 0.875
n_claims: 8
---

# metabolite-target-definition

## Summary

Defines MS target m/z windows for a metabolite by computing its theoretical isotope pattern from molecular formula and adduct at a given MS resolution (via enviPat), and, for tracer assays, the m/z series of isotopologues incorporating 0..N tracer atoms. These target tables drive TarMet's downstream isotopologue and quantitative analysis modules.

## When to use

Use when building a targeted LC-MS assay in TarMet and you must define target metabolite m/z windows from a molecular formula/adduct, or derive isotopologue m/z series for a stable-isotope tracer with a chosen isotope and max isotopologues N.

## When NOT to use

- Raw EIC processing tasks (baseline removal, peak detection, retention-time alignment) — those belong to TarMet's later isotopologues/quantitative analysis steps, not target definition
- Tracer m/z series cannot be computed without the user first specifying the tracer isotope and maximum number of isotopologues
- Unbiased/untargeted feature discovery — this skill only generates targets for compounds with a known formula and adduct

## Inputs

- Molecular formula of the target compound
- Adduct specification (ion species observed in the MS run)
- MS instrument resolution
- For tracer assays: tracer isotope identity and maximum number of isotopologues (N)

## Outputs

- Theoretical isotope pattern table for the compound at the given resolution
- m/z-range table with one m/z window per tracer isotopologue (0..N tracer atoms), used by downstream isotopologue and quantitative analysis modules

## How to apply

Load the target compound definition (molecular formula and adduct) into the TarMet shiny environment, then call getIsotopicPattern to calculate the theoretical isotope pattern at the resolution of the MS instrument, relying on the enviPat package for the pattern computation. For tracer assays, first specify the tracer isotope and the maximum number of isotopologues (N), then call getMzWithTracer to compute the m/z series of the isotopologues with 0..N incorporated tracer atoms, yielding an expected m/z window per isotopologue. Write out the theoretical isotope-pattern table and the m/z-range table as the target-definition outputs consumed by the isotopologues analysis and quantitative analysis modules. The m/z shift per tracer atom must be consistent with the enviPat-derived theoretical pattern, so cross-check each isotopologue row against the pattern table before proceeding.

## Related tools

- **TarMet** (Reactive shiny application whose target-definition step computes theoretical isotope patterns and tracer m/z windows via getIsotopicPattern and getMzWithTracer) — https://github.com/hcji/TarMet
- **enviPat** (Package TarMet relies on to calculate the theoretical isotope pattern of compounds from formula/adduct at the MS resolution)
- **shiny** (Provides the graphical user interface in which the target compound definition (formula, adduct) is loaded and tracer parameters are specified)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- The isotope-pattern table and m/z-range table files are produced after target definition
- Each tracer isotopologue row (0..N) carries an expected m/z range
- m/z shifts per incorporated tracer atom are consistent with the theoretical pattern computed from enviPat
- The pattern matches the specified MS instrument resolution (pattern width is resolution-dependent)

## Limitations

- Tracer-assist m/z computation is conditional: the user must first specify the tracer isotope and maximum number of isotopologues before getMzWithTracer can produce windows
- Accuracy of the theoretical pattern depends on a correct molecular formula and adduct specification
- Planned but not yet supported: data-independent acquisition and MS/MS library verification

## Evidence
