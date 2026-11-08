---
name: stable-isotope-tracer-mz-series-calculation
description: 'Use when defining target lists for targeted LC-MS metabolite quantification
  or stable-isotope tracer assays: you have a molecular formula plus adduct and need
  isotope-pattern m/z windows or per-isotopologue m/z series for up to N tracer atoms.'
when_to_use_negative:
- The input is already a feature table or extracted-ion-chromatogram peak list — this
  skill only defines targets, not extracts signals (downstream isotopologues/quantitative
  analysis steps handle that).
- The target lacks a known molecular formula or adduct, since both are required inputs
  to getIsotopicPattern/enviPat.
- The analysis is untargeted metabolomics without predefined targets — TarMet targets
  are defined per compound before analysis.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TarMet
  role: Shiny application providing getIsotopicPattern (theoretical isotope pattern
    from formula/adduct/resolution) and getMzWithTracer (m/z series of isotopologues
    with 0..N tracer atoms) for target metabolite definition
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: R package that computes the theoretical isotope patterns of compounds based
    on the resolution of the MS
- name: shiny
  role: Reactive GUI framework through which TarMet exposes target metabolite definition
    (2.1) and tracer m/z computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:d8f085ea7852c15fac56aa2e5e5380a818cdde7e245e865bc9dd950db2310d50
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
yes_rate: 0.6666666666666666
n_claims: 15
---

# stable-isotope-tracer-mz-series-calculation

## Summary

Compute theoretical isotope patterns from a metabolite's molecular formula and adduct at the MS instrument resolution via enviPat (TarMet getIsotopicPattern), and for tracer assays derive the m/z series of isotopologues incorporating 0..N tracer atoms (getMzWithTracer).

## When to use

Use when defining target lists for targeted LC-MS metabolite quantification or stable-isotope tracer assays: you have a molecular formula plus adduct and need isotope-pattern m/z windows or per-isotopologue m/z series for up to N tracer atoms.

## When NOT to use

- The input is already a feature table or extracted-ion-chromatogram peak list — this skill only defines targets, not extracts signals (downstream isotopologues/quantitative analysis steps handle that).
- The target lacks a known molecular formula or adduct, since both are required inputs to getIsotopicPattern/enviPat.
- The analysis is untargeted metabolomics without predefined targets — TarMet targets are defined per compound before analysis.

## Inputs

- Target compound definition: molecular formula and adduct
- MS instrument resolution
- Tracer assay specification: tracer isotope and maximum number of isotopologues (N tracer atoms)

## Outputs

- Theoretical isotope-pattern table (formula/adduct/resolution, computed via enviPat)
- m/z-range table with one row per tracer isotopologue (0..N incorporated tracer atoms), each carrying its expected m/z range

## How to apply

Load the target compound definition (molecular formula and adduct) into the TarMet shiny environment, then call getIsotopicPattern to calculate the theoretical isotope pattern of the compound at the resolution of the MS instrument, delegating the pattern computation to the enviPat package. For tracer assays, first specify the tracer isotope and the maximum number of isotopologues, then use getMzWithTracer to compute the m/z series of the isotopologues with 0..N incorporated tracer atoms. Derive the expected m/z windows for each isotopologue from these values and write out both the theoretical isotope-pattern table and the m/z-range table as the target-definition outputs feeding the downstream isotopologues analysis (2.2) and quantitative analysis (2.3) modules. The instrument resolution must match the actual MS acquisition, because enviPat bases the pattern on that resolution.

## Related tools

- **TarMet** (Shiny application providing getIsotopicPattern (theoretical isotope pattern from formula/adduct/resolution) and getMzWithTracer (m/z series of isotopologues with 0..N tracer atoms) for target metabolite definition) — https://github.com/hcji/TarMet
- **enviPat** (R package that computes the theoretical isotope patterns of compounds based on the resolution of the MS)
- **shiny** (Reactive GUI framework through which TarMet exposes target metabolite definition (2.1) and tracer m/z computation)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- Both the isotope-pattern table and the m/z-range table files are produced as target-definition outputs
- Each tracer isotopologue row carries an expected m/z range
- m/z shifts per tracer atom are consistent with the theoretical pattern computed by enviPat
- Isotopologue series spans 0..N incorporated tracer atoms as specified by the user-supplied maximum
- The theoretical isotope pattern matches the compound's formula and adduct at the declared MS resolution

## Limitations

- Tracer m/z computation requires the user to first specify the tracer isotope and the maximum number of isotopologues; the tool cannot infer these
- Isotope-pattern accuracy depends on supplying the correct MS instrument resolution
- MS/MS library verification and data-independent acquisition support are only planned future features, not yet available

## Evidence

- [other] Tracer assay m/z computation: [evidence span withheld — anchor status: not_found]
