---
name: mass-spectrometry-resolution-aware-peak-shape-modeling
description: Use when a target metabolite's molecular formula and adduct are known
  and you need resolution-aware theoretical isotope patterns or tracer-isotopologue
  m/z windows as target lists for downstream quantification.
when_to_use_negative:
- The aim is raw-peak detection, baseline removal, or retention-time alignment (TarMet
  handles those separately via MassSpecWavelet, airPLS, and PAFFT) — this skill only
  builds the target definitions.
- No molecular formula/adduct is available for the compound, so no theoretical isotope
  pattern can be computed.
- Quantification of already-defined targets without needing new isotopologue m/z windows
  or tracer-assay target lists.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: Shiny application exposing getIsotopicPattern and getMzWithTracer for target
    metabolite definition and tracer m/z-series computation
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Backend package that computes the theoretical isotope pattern of the compound
    from formula/adduct at the MS resolution
- name: shiny
  role: Reactive GUI framework through which the user supplies formula, adduct, resolution,
    and tracer parameters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:e5e19712e2265541b122b4737f5433be15e00fd7d7821e014c93050ab78febef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - enviPat
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectrometry-resolution-aware-peak-shape-modeling

## Summary

Defines MS targets for a metabolite by computing its theoretical isotope pattern from formula/adduct at the instrument's resolution via enviPat (TarMet getIsotopicPattern), and for tracer assays derives the m/z windows of isotopologues with 0..N incorporated tracer atoms (getMzWithTracer).

## When to use

Use when a target metabolite's molecular formula and adduct are known and you need resolution-aware theoretical isotope patterns or tracer-isotopologue m/z windows as target lists for downstream quantification.

## When NOT to use

- The aim is raw-peak detection, baseline removal, or retention-time alignment (TarMet handles those separately via MassSpecWavelet, airPLS, and PAFFT) — this skill only builds the target definitions.
- No molecular formula/adduct is available for the compound, so no theoretical isotope pattern can be computed.
- Quantification of already-defined targets without needing new isotopologue m/z windows or tracer-assay target lists.

## Inputs

- Target compound definition: molecular formula and adduct
- MS instrument resolution
- For tracer assays: tracer isotope label and maximum number of tracer atoms N

## Outputs

- Theoretical isotope-pattern table (m/z and abundance per isotopologue, resolution-dependent)
- m/z-range table: one row per tracer isotopologue (0..N incorporated atoms) with its expected m/z window
- Target-definition outputs consumed by TarMet's isotopologues-analysis and quantitative-analysis modules

## How to apply

Load the target compound definition (molecular formula plus adduct) into the TarMet shiny environment and call getIsotopicPattern, which delegates the isotope-pattern computation to the enviPat package using the resolution of the MS instrument as a parameter so the predicted envelope reflects real resolving power. For stable-isotope tracer assays, specify the tracer isotope (e.g. 13C) and the maximum number of isotopologues N, then call getMzWithTracer to compute the m/z series of isotopologues incorporating 0..N tracer atoms, yielding an expected m/z window for each isotopologue. Write both the theoretical isotope-pattern table and the m/z-range table as the target-definition outputs that feed TarMet's downstream isotopologues-analysis (2.2) and quantitative-analysis (2.3) modules. The rationale is that peaks and isotopologue separations shift with MS resolution, so resolution must be an explicit input rather than an afterthought.

## Related tools

- **TarMet** (Shiny application exposing getIsotopicPattern and getMzWithTracer for target metabolite definition and tracer m/z-series computation) — https://github.com/hcji/TarMet
- **enviPat** (Backend package that computes the theoretical isotope pattern of the compound from formula/adduct at the MS resolution)
- **shiny** (Reactive GUI framework through which the user supplies formula, adduct, resolution, and tracer parameters)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- Both the isotope-pattern table and the m/z-range table files are produced as target-definition outputs.
- Each tracer isotopologue row (0..N) carries an expected m/z range, and the per-tracer-atom m/z shifts are consistent with the enviPat theoretical pattern.
- The computed isotope envelope matches the stated MS resolution (peak spacing/resolution-dependent separation is plausible).
- Downstream TarMet modules (isotopologues analysis 2.2, quantitative analysis 2.3) accept the tables without re-deriving targets.

## Limitations

- Future features not yet supported: data-independent acquisition and MS/MS library verification, so targets are built for regular (non-DIA) targeted MS workflows.
- Requires accurate molecular formula and adduct; the pattern quality depends on the resolution supplied for the instrument.

## Evidence
