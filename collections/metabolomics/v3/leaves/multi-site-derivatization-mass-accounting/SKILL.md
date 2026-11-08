---
name: multi-site-derivatization-mass-accounting
description: Use when annotating MALDI-MSI peaks from a derivatizing matrix (e.g.
  FMP-10) and the analyte has multiple taggable functional groups, so single- and
  double-derivatized species (forms A and B) must be mass-predicted separately.
when_to_use_negative:
- The analyte has only one taggable functional group, so only the singly derivatized
  species exists and multi-site mass accounting adds no variants
- The data were acquired with a conventional (non-derivatizing) matrix, where ions
  follow standard [M+H]+/[M-H]- rules and no derivatization increments apply
- Input is already a list of Met-ID annotation results; this skill predicts masses
  rather than annotating them
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Met-ID
  role: Provides the metabolite database (HMDB), search_by_name structure lookup,
    identify() mass matching against observed m/z with mass_error=5 ppm, and built-in
    FMP-10 matrix functional-group definitions
  repo: https://github.com/pbjarterot/Met-ID/releases
- name: RDKit
  role: Computes the neutral monoisotopic mass of the parent analyte (e.g. dopamine)
    that derivatization increments are added to
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:3bd26418fd880e50947cad5377e1289ddf57c3bdd17d8f9c56e7eaa2d9228c33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multi-site-derivatization-mass-accounting

## Summary

Computes theoretical [M+H]+ m/z values for metabolites derivatized one or more times by a reactive matrix such as FMP-10, accounting for the two double-derivatization variants (demethylated form A via methyl loss and dehydrogenated form B via hydrogen loss). Rationale: FMP-10 can tag multiple functional groups on one molecule, so predicted masses must be generated per derivatization count and variant before checking against observed peaks.

## When to use

Use when annotating MALDI-MSI peaks from a derivatizing matrix (e.g. FMP-10) and the analyte has multiple taggable functional groups, so single- and double-derivatized species (forms A and B) must be mass-predicted separately.

## When NOT to use

- The analyte has only one taggable functional group, so only the singly derivatized species exists and multi-site mass accounting adds no variants
- The data were acquired with a conventional (non-derivatizing) matrix, where ions follow standard [M+H]+/[M-H]- rules and no derivatization increments apply
- Input is already a list of Met-ID annotation results; this skill predicts masses rather than annotating them

## Inputs

- Analyte name or structure resolvable via Met-ID search_by_name (e.g. 'dopamine') with neutral monoisotopic mass from RDKit
- FMP-10 derivatization increment(s), including demethylated (form A) and dehydrogenated (form B) double-derivatization variants
- Observed peak m/z values from MALDI-MSI data processed with the FMP-10 matrix

## Outputs

- Theoretical [M+H]+ m/z for each derivatization state (1A, 2A, 2B), e.g. 421.1911, 674.2802, 688.2959 for dopamine
- Computed-vs-reported comparison with dPPM deviations (e.g. derivatized_dopamine_mz_check.csv)
- Met-ID identification hits for the computed m/z values within the 5 ppm window

## How to apply

Retrieve the analyte structure from the Met-ID database (search_by_name, e.g. 'dopamine') and compute its neutral monoisotopic mass with RDKit. Compute the singly derivatized [M+H]+ ion by adding one FMP-10 derivatization increment plus a proton (for dopamine this gives m/z 421.1911). For the doubly derivatized species, add two increments with the demethylation variant (form A, m/z 674.2802 for dopamine) and the dehydrogenation variant (form B, m/z 688.2959), both as [M+H]+. Cross-check computed values against observed peaks using Met-ID's identify() with a 5 ppm tolerance (mass_error=5), then express computed-vs-reported deviations as dPPM. Validation: all variants must agree with reported values within the 5 ppm mass error used by the Met-ID identification workflow.

## Related tools

- **Met-ID** (Provides the metabolite database (HMDB), search_by_name structure lookup, identify() mass matching against observed m/z with mass_error=5 ppm, and built-in FMP-10 matrix functional-group definitions) — https://github.com/pbjarterot/Met-ID/releases
- **RDKit** (Computes the neutral monoisotopic mass of the parent analyte (e.g. dopamine) that derivatization increments are added to)

## Examples

```
metid.identify(metabolome="HMDB(All)", matrix="FMP-10", typ="All", adducts="All", mass_error=5, mz=421.1911, mzwindow=5, ccs=-1)
```

## Evaluation signals

- Computed 1A, 2A, 2B m/z values match reported peaks (421.1911, 674.2802, 688.2959 for dopamine) within 5 ppm dPPM
- identify() call with matrix='FMP-10' and mass_error=5 returns a hit for the singly derivatized m/z 421.1911 against metabolome='HMDB(All)'
- Form A (demethylated) and form B (dehydrogenated) double-derivatization masses differ by the expected methyl-vs-hydrogen loss mass offset (15.0157 Da difference between 674.2802 and 688.2959 corresponds to CH4 vs H2 loss variants)
- Proton addition is consistently applied ([M+H]+ mode) across all computed variants
- Output CSV contains computed m/z, reported m/z, and dPPM columns for all derivatization variants

## Limitations

- FMP-10 derivatizing matrix was developed in-house; its availability/distribution to other users is not stated
- Met-ID distinguishes dopamine derivatives retaining or losing a methyl group only via MS2 fragmentation patterns, so mass matching alone may not fully resolve form A from form B
- Double-derivatization chemistry is matrix-specific; increments must be redefined for other derivatizing matrices

## Evidence

- [abstract] Multiple derivatization sites on one molecule are the core capability: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [methods] identify() accepts one or more m/z values: "``identify`` accepts one or more m/z values and optional corresponding CCS values."
- [readme] Met-ID focuses on derivatizing matrices yielding non-standard ions: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
- [methods] Default matrix and metabolome configuration for FMP-10 workflows: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [discussion] MS2-level discrimination between methyl-retaining and methyl-losing dopamine derivatives: "Met-ID can distinguish between dopamine derivatives that retain or lose a methyl group based on their MS2 fragmentation patterns"
