---
name: derivatized-metabolite-theoretical-mz-calculation
description: Use when annotating MALDI-MSI peaks produced with a derivatizing matrix
  such as FMP-10, where double derivatization yields demethylated (loss of CH3) or
  dehydrogenated (loss of H) variants and theoretical m/z values are needed for identification.
when_to_use_negative:
- Data were acquired with a conventional (non-derivatizing) matrix so only common
  [M+H]+ / [M-H]- ions are expected and no derivatization increments apply
- The target metabolite lacks the functional groups specific to the derivatization
  agent, so Met-ID's target subset generation would return no candidates
- The goal is MS2-based discrimination of derivative variants (e.g. methyl-retaining
  vs methyl-losing fragments) rather than MS1 theoretical m/z computation
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: Met-ID
  role: Provides the metabolome/matrix/adduct database (HMDB, FMP-10 functional groups),
    search_by_name() for structure retrieval, and identify() for m/z matching at mass_error=5
  repo: https://github.com/pbjarterot/Met-ID
- name: RDKit
  role: Computes the neutral monoisotopic mass of the retrieved metabolite structure
    used as the basis for adding derivatization increments
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:cbf6c884879b049bf272a8b6c8a5a24c0b0359ecc98c02a67e31b648978ac4d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# derivatized-metabolite-theoretical-mz-calculation

## Summary

Computes theoretical [M+H]+ m/z values for singly and multiply derivatized metabolites (e.g. FMP-10-tagged dopamine forms 1A, 2A, 2B) from the neutral monoisotopic mass plus derivatization increments, and validates them against reported values within a 5 ppm mass error via the Met-ID identify() workflow.

## When to use

Use when annotating MALDI-MSI peaks produced with a derivatizing matrix such as FMP-10, where double derivatization yields demethylated (loss of CH3) or dehydrogenated (loss of H) variants and theoretical m/z values are needed for identification.

## When NOT to use

- Data were acquired with a conventional (non-derivatizing) matrix so only common [M+H]+ / [M-H]- ions are expected and no derivatization increments apply
- The target metabolite lacks the functional groups specific to the derivatization agent, so Met-ID's target subset generation would return no candidates
- The goal is MS2-based discrimination of derivative variants (e.g. methyl-retaining vs methyl-losing fragments) rather than MS1 theoretical m/z computation

## Inputs

- Metabolite structure/name retrieved from the Met-ID metabolome database (e.g. search_by_name("dopamine") against HMDB(All))
- Neutral monoisotopic mass computed with RDKit from the metabolite structure
- FMP-10 derivatization increments (single, double-demethylated, double-dehydrogenated)
- List of target m/z values to validate (e.g. 421.1911, 674.2802, 688.2959)

## Outputs

- Theoretical [M+H]+ m/z values for single-derivatized form 1A and double-derivatized forms 2A (demethylated) and 2B (dehydrogenated)
- Computed-vs-reported comparison table with dPPM deviations written to derivatized_dopamine_mz_check.csv
- Met-ID identify() match results confirming the m/z falls within the 5 ppm mass error window

## How to apply

Retrieve the target metabolite structure from the Met-ID database with search_by_name (e.g. 'dopamine') and compute its neutral monoisotopic mass with RDKit. For single derivatization (form 1A), add one FMP-10 derivatization increment plus a proton to the neutral monoisotopic mass to get the [M+H]+ m/z; for double derivatization, add two increments with the demethylation variant (form 2A, loss of a methyl group) or the dehydrogenation variant (form 2B, loss of hydrogen), again as [M+H]+ ions. Cross-check at least one computed value by calling metid.identify() with metabolome='HMDB(All)', matrix='FMP-10', typ='All', adducts='All', mass_error=5, mz=the computed value, mzwindow=5, ccs=-1. Express computed-vs-reported deviations as ppm differences (dPPM) and write the comparison table to a CSV (e.g. derivatized_dopamine_mz_check.csv). The rationale is that derivatizing matrices like FMP-10 tag multiple functional groups, so a single metabolite can generate several characteristic ions beyond the common [M+H]+.

## Related tools

- **Met-ID** (Provides the metabolome/matrix/adduct database (HMDB, FMP-10 functional groups), search_by_name() for structure retrieval, and identify() for m/z matching at mass_error=5) — https://github.com/pbjarterot/Met-ID
- **RDKit** (Computes the neutral monoisotopic mass of the retrieved metabolite structure used as the basis for adding derivatization increments) — https://www.rdkit.org/

## Examples

```
metid.identify(metabolome="HMDB(All)", matrix="FMP-10", typ="All", adducts="All", mass_error=5, mz=421.1911, mzwindow=5, ccs=-1)
```

## Evaluation signals

- Computed [M+H]+ m/z for 1A, 2A, and 2B (421.1911, 674.2802, 688.2959) must agree with reported values within the 5 ppm mass error used by Met-ID identification
- metid.identify() with mz=421.1911, mass_error=5, mzwindow=5 on metabolome='HMDB(All)', matrix='FMP-10' must return a dopamine match
- dPPM column in derivatized_dopamine_mz_check.csv shows |deviation| <= 5 ppm for all three forms
- Double-derivatized 2A vs 2B differ by ~14 Da as expected from the methyl-loss vs hydrogen-loss variants (688.2959 - 674.2802 ≈ 14.0157)

## Limitations

- As the FMP-10 derivatizing matrix was developed in-house, its data sets were used as benchmarks during the software's development; availability to other users is not stated
- The installation of rdkit is slightly more difficult than an average PYPI project but often works with anaconda; no pinned dependency versions are specified
- Reinstalling newer versions of Met-ID does not update the bundled database files, which must be removed manually before reinstalling
- Only theoretical MS1 m/z values are computed here; distinguishing methyl-retaining vs methyl-losing derivatives requires MS2 fragmentation patterns

## Evidence

- [abstract] The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [methods] identify accepts m/z values: "``identify`` accepts one or more m/z values and optional corresponding CCS values."
- [methods] Met-ID defaults to HMDB and FMP-10: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [readme] README: derivatizing matrices focus: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
