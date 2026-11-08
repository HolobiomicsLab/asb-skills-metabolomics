---
name: isotopologue-abundance-normalization
description: Use when tracer quantification output (e.g. from TarMet's isotope tracer
  assay on 13C6-glucose samples) provides raw isotopologue peak areas (M+1, M+2, ...)
  and you need per-metabolite fractional labeling to check pathway-position labeling
  patterns.
when_to_use_negative:
- Input is raw EIC chromatograms or unquantified peaks rather than already-quantified
  isotopologue areas from TarMet's isotopologue analysis step
- The goal is absolute targeted metabolite quantification (TarMet's 'Quantitative
  analysis' workflow) rather than relative labeling within an isotopologue envelope
- The tracer experiment does not include parallel samples or a defined expected labeling
  pattern, so the fractional values cannot be cross-validated
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TarMet
  role: Produces the quantified isotopologue areas upstream of this normalization;
    its isotope tracer assay workflow is the data source, and the normalization table
    (isotopologue_fractions.csv) summarizes its output
  repo: https://github.com/hcji/TarMet
- name: shiny
  role: Reactive GUI through which the TarMet isotopologue quantification and tracer
    analysis is driven interactively
  repo: https://github.com/hcji/TarMet
- name: enviPat
  role: Computes the theoretical isotope patterns of the compounds based on the MS
    resolution, defining which isotopologues are expected per metabolite before normalization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:2370be2612894737c56a601decf8b77ce1786c2bac2282f8614960d2d996a6d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - TarMet
  - enviPat
  - shiny
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# isotopologue-abundance-normalization

## Summary

Normalizes per-metabolite 13C isotopologue peak areas from a stable isotope tracer experiment into fractional abundances (relative labeling per isotopologue), so labeling patterns across a metabolic pathway can be compared between parallel samples and validated against expected metabolism rules.

## When to use

Use when tracer quantification output (e.g. from TarMet's isotope tracer assay on 13C6-glucose samples) provides raw isotopologue peak areas (M+1, M+2, ...) and you need per-metabolite fractional labeling to check pathway-position labeling patterns.

## When NOT to use

- Input is raw EIC chromatograms or unquantified peaks rather than already-quantified isotopologue areas from TarMet's isotopologue analysis step
- The goal is absolute targeted metabolite quantification (TarMet's 'Quantitative analysis' workflow) rather than relative labeling within an isotopologue envelope
- The tracer experiment does not include parallel samples or a defined expected labeling pattern, so the fractional values cannot be cross-validated

## Inputs

- Quantified isotopologue peak areas per metabolite per sample from the TarMet targeted quantification / isotopologue analysis step (e.g. 13C6-glucose tracer EIC-derived areas for M+1, M+2, ... isotopologues)
- Metabolite-to-pathway-position mapping (e.g. ordered glycolysis metabolite list: G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso)

## Outputs

- Summary table of fractional labeling (relative abundance) per isotopologue per metabolite, written as isotopologue_fractions.csv
- Per-metabolite total labeled pool (sum of all 13C isotopologue areas) used as the normalization denominator

## How to apply

Load the quantified isotopologue peak areas for the tracer-labeled metabolites from the upstream TarMet tracer workflow output. For each metabolite, sum the areas of all its 13C isotopologues to obtain the total labeled pool, then divide each individual isotopologue area (M+1, M+2, ...) by that total to compute the fractional labeling (relative abundance) per isotopologue. Assemble the fractional labeling values for all metabolites of interest into a summary table (e.g. Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso for the glycolysis pathway) and write it as isotopologue_fractions.csv. The rationale is that raw areas are comparable only within a metabolite, so normalizing within each metabolite's isotopologue envelope lets labeling patterns be compared across parallel samples and across pathway position. Validate that fractional abundances sum to 1 per metabolite and that the expected biology is reproduced — for 13C6-glucose, upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) should be predominantly labeled while downstream ones (Pyr, Ala, Lac) remain unlabeled.

## Related tools

- **TarMet** (Produces the quantified isotopologue areas upstream of this normalization; its isotope tracer assay workflow is the data source, and the normalization table (isotopologue_fractions.csv) summarizes its output) — https://github.com/hcji/TarMet
- **shiny** (Reactive GUI through which the TarMet isotopologue quantification and tracer analysis is driven interactively) — https://github.com/hcji/TarMet
- **enviPat** (Computes the theoretical isotope patterns of the compounds based on the MS resolution, defining which isotopologues are expected per metabolite before normalization)

## Examples

```
library(TarMet); runTarMet()
```

## Evaluation signals

- For every metabolite, the fractional abundances of all its isotopologues (M+1, M+2, ...) must sum to 1 within floating-point tolerance
- Quantitative results across parallel samples must be highly consistent after normalization
- Upstream glycolysis metabolites (G6P, F6P, FBP, DHAP) must be predominantly labeled while downstream ones (Pyr, Ala, Lac) remain unlabeled, matching the expected 13C6-glucose metabolism rules
- The output table must contain the expected metabolite set (Glu, G6P, F6P, FBP, DHAP, PG, PEP, Pyr, Ala, Lac, Cit/Iso) and be written as isotopologue_fractions.csv

## Limitations

- TarMet's listed future work does not yet include MS/MS library verification or data-independent acquisition support, so isotopologue assignments are not independently verified by MS/MS
- Interpretation depends on the assumed pathway order; if a metabolite's position in the labeling cascade is wrong, the upstream-labeled/downstream-unlabeled check is not diagnostic

## Evidence
