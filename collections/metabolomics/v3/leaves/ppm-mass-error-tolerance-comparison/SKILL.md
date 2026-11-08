---
name: ppm-mass-error-tolerance-comparison
description: Use when you need to verify that a computed theoretical m/z for a derivatized
  or adducted species matches a reported/observed peak within a defined ppm tolerance,
  or to derive expected m/z values for single- and multi-site derivatization products.
when_to_use_negative:
- Input is already a confirmed identification from Met-ID's MS2 spectral database
  rather than a theoretical m/z that needs verification
- The analyte is underivatized and detected as a conventional [M+H]+/[M-H]- ion, so
  no derivatization increments need to be modeled
- Observed peaks have mass errors exceeding the configured tolerance (e.g., >5 ppm),
  which the Met-ID mass-match filter would reject as false positives
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Met-ID
  role: Provides the identify() mass-match workflow with a user-defined mass_error
    (5 ppm), the HMDB/FMP-10 matrix database queried via search_by_name(), and recalibration
    based on known m/z ratios
  repo: https://github.com/pbjarterot/Met-ID/releases
- name: RDKit
  role: Computes the neutral monoisotopic mass of the metabolite structure from which
    theoretical derivatized m/z values are derived
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:ad12ae02d879604c866dcb5d89e990ccd17533bdbd85107d6b13c618bd844548
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ppm-mass-error-tolerance-comparison

## Summary

Computes theoretical m/z values for derivatized metabolites (e.g., FMP-10-tagged dopamine) and compares them against reported or observed values as ppm differences, validating agreement within a user-specified mass error (e.g., 5 ppm) used by Met-ID identification.

## When to use

Use when you need to verify that a computed theoretical m/z for a derivatized or adducted species matches a reported/observed peak within a defined ppm tolerance, or to derive expected m/z values for single- and multi-site derivatization products.

## When NOT to use

- Input is already a confirmed identification from Met-ID's MS2 spectral database rather than a theoretical m/z that needs verification
- The analyte is underivatized and detected as a conventional [M+H]+/[M-H]- ion, so no derivatization increments need to be modeled
- Observed peaks have mass errors exceeding the configured tolerance (e.g., >5 ppm), which the Met-ID mass-match filter would reject as false positives

## Inputs

- Metabolite structure/name resolvable via Met-ID database (e.g., search_by_name("dopamine"))
- Derivatization agent increment(s) (FMP-10; single and double, with demethylation and dehydrogenation variants)
- Reported or observed m/z values (e.g., 421.1911, 674.2802, 688.2959)
- Mass error tolerance in ppm (e.g., mass_error=5)

## Outputs

- Computed theoretical [M+H]+ m/z values for each derivatization species (1A, 2A, 2B)
- PPM deviation table (dPPM) comparing computed vs. reported m/z values
- Comparison CSV file (e.g., derivatized_dopamine_mz_check.csv)
- Met-ID identify() match results at the chosen ppm tolerance

## How to apply

Retrieve the metabolite structure (e.g., search_by_name("dopamine") in the Met-ID HMDB database) and compute its neutral monoisotopic mass with RDKit. Build the theoretical ion m/z by adding derivatization increments and the proton: a single FMP-10 derivatization gives the [M+H]+ form (e.g., dopamine 1A at m/z 421.1911), while double derivatization requires two variants — demethylation (loss of a methyl group, form 2A, m/z 674.2802) or dehydrogenation (loss of hydrogen, form 2B, m/z 688.2959). Cross-check computed values by calling metid.identify() with mass_error=5 and mz set to the computed value, then express computed-vs-reported deviations as dPPM. Accept the assignment only when all deviations fall within the 5 ppm mass error used by the Met-ID workflow.

## Related tools

- **Met-ID** (Provides the identify() mass-match workflow with a user-defined mass_error (5 ppm), the HMDB/FMP-10 matrix database queried via search_by_name(), and recalibration based on known m/z ratios) — https://github.com/pbjarterot/Met-ID/releases
- **RDKit** (Computes the neutral monoisotopic mass of the metabolite structure from which theoretical derivatized m/z values are derived)

## Examples

```
metid.identify(metabolome="HMDB(All)", matrix="FMP-10", typ="All", adducts="All", mass_error=5, mz=421.1911, mzwindow=5, ccs=-1)
```

## Evaluation signals

- All computed m/z values (1A, 2A, 2B) agree with reported values within 5 ppm (dPPM within ±5)
- metid.identify() with mz=421.1911, mass_error=5, mzwindow=5, metabolome="HMDB(All)", matrix="FMP-10" returns the expected dopamine derivative match
- The double-derivatized variants are separable: demethylated 2A (m/z 674.2802) differs from dehydrogenated 2B (m/z 688.2959) by ~14 Da (CH2 vs H loss)
- Comparison table CSV is written with computed, reported, and dPPM columns populated for all species

## Limitations

- Mass matching alone can produce false positives; Met-ID provides recalibration with known m/z ratios and MS2 similarity comparison to boost confidence
- Met-ID's bundled MS2 spectral database is a preconfigured finite set; no matching MS2 spectrum may exist for a given derivatized standard
- Met-ID depends on rdkit, whose installation is nontrivial and works most reliably via anaconda; no pinned dependency versions are stated

## Evidence

- [discussion] Met-ID can distinguish between dopamine derivatives that retain or lose a methyl group based on their MS2 fragmentation patterns: "Met-ID can distinguish between dopamine derivatives that retain or lose a methyl group based on their MS2 fragmentation patterns"
- [methods] identify accepts m/z values and optional CCS: "``identify`` accepts one or more m/z values and optional corresponding CCS values."
- [methods] defaults select HMDB(All) and FMP-10 matrix: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [abstract] Met-ID identifies multiple derivatization sites exemplified by FMP-10: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [abstract] recalibrate with known m/z ratios boosts confidence: "Met-ID also permits users to recalibrate data with known m/z ratios, boosting confidence in mass match results."
