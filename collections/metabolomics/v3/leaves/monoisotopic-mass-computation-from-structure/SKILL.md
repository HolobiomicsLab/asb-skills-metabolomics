---
name: monoisotopic-mass-computation-from-structure
description: Use when you need to verify or predict the theoretical m/z of a derivatized
  metabolite from its chemical structure — e.g., confirming that Met-ID hits for FMP-10-derivatized
  dopamine (1A, 2A, 2B) correspond to expected [M+H]+ ions within a 5 ppm mass-error
  window.
when_to_use_negative:
- When the goal is annotation of observed m/z features from MALDI-MSI data rather
  than computing theoretical masses from a known structure — use metid.identify()
  directly instead
- When discrimination between derivatized forms relies on MS2 fragmentation patterns
  (e.g., distinguishing dopamine derivatives that retain vs. lose a methyl group)
  — mass computation alone cannot resolve this
- When the metabolite has no structure/name entry in the HMDB- or LIPID MAPS-derived
  Met-ID database
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: RDKit
  role: Computes the neutral monoisotopic (exact) mass of the metabolite structure
    from which derivatized [M+H]+ m/z values are derived
  repo: https://www.rdkit.org/
- name: Met-ID
  role: Supplies the metabolite structure (search_by_name) and validates computed
    m/z values against the identification workflow via identify() with mass_error=5
    ppm on the FMP-10/HMDB(All) configuration
  repo: https://github.com/pbjarterot/Met-ID/releases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:4e8159b32c7be0107aca1d711cdc0157365359a2622d690d3b13dfaaac90ed37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# monoisotopic-mass-computation-from-structure

## Summary

Computes the neutral monoisotopic mass of a metabolite from its structure (via RDKit) and derives theoretical [M+H]+ m/z values for single and double FMP-10 derivatization states, enabling validation of Met-ID annotations such as dopamine peaks at m/z 421.1911, 674.2802, and 688.2959.

## When to use

Use when you need to verify or predict the theoretical m/z of a derivatized metabolite from its chemical structure — e.g., confirming that Met-ID hits for FMP-10-derivatized dopamine (1A, 2A, 2B) correspond to expected [M+H]+ ions within a 5 ppm mass-error window.

## When NOT to use

- When the goal is annotation of observed m/z features from MALDI-MSI data rather than computing theoretical masses from a known structure — use metid.identify() directly instead
- When discrimination between derivatized forms relies on MS2 fragmentation patterns (e.g., distinguishing dopamine derivatives that retain vs. lose a methyl group) — mass computation alone cannot resolve this
- When the metabolite has no structure/name entry in the HMDB- or LIPID MAPS-derived Met-ID database

## Inputs

- Metabolite structure or name from the Met-ID metabolite database (e.g., 'dopamine', retrieved via search_by_name)
- Neutral monoisotopic mass computed by RDKit from the metabolite structure
- FMP-10 derivatization increment definitions (single tag, double tag with demethylation, double tag with dehydrogenation)
- Reference m/z values reported for the derivatized species (421.1911, 674.2802, 688.2959)

## Outputs

- Theoretical [M+H]+ m/z values for the single-derivatized form 1A and double-derivatized forms 2A (demethylated) and 2B (dehydrogenated)
- PPM deviations (dPPM) between computed and reported m/z values
- Comparison table written to CSV (e.g., derivatized_dopamine_mz_check.csv)

## How to apply

Retrieve the metabolite structure from the Met-ID database with search_by_name('dopamine') and compute its neutral monoisotopic mass with RDKit's exact-mass calculator. For the single-derivatized species 1A, add one FMP-10 derivatization increment plus a proton to the neutral mass to obtain the [M+H]+ ion. For the double-derivatized species, add two FMP-10 increments with the two reaction variants: the demethylated form 2A (loss of a methyl group) and the dehydrogenated form 2B (loss of hydrogen), again as [M+H]+ ions. Cross-check at least one computed value against the Met-ID identification workflow by calling metid.identify() with metabolome='HMDB(All)', matrix='FMP-10', typ='All', adducts='All', mass_error=5, mz=421.1911, mzwindow=5, ccs=-1. Express all computed-vs-reported deviations as ppm differences (dPPM) against the reported values 421.1911, 674.2802, and 688.2959, and write the comparison table to a CSV file (e.g., derivatized_dopamine_mz_check.csv). The rationale is that derivatizing matrices like FMP-10 produce ions beyond the common [M+H]+/[M-H]- adducts, so structure-based mass computation is needed to assign and validate multiple derivatization sites.

## Related tools

- **RDKit** (Computes the neutral monoisotopic (exact) mass of the metabolite structure from which derivatized [M+H]+ m/z values are derived) — https://www.rdkit.org/
- **Met-ID** (Supplies the metabolite structure (search_by_name) and validates computed m/z values against the identification workflow via identify() with mass_error=5 ppm on the FMP-10/HMDB(All) configuration) — https://github.com/pbjarterot/Met-ID/releases

## Examples

```
from rdkit import Chem; from rdkit.Chem import Descriptors; m = Chem.MolFromSmiles('NCCc1ccc(O)c(O)c1'); mono = Descriptors.ExactMolWt(m)
```

## Evaluation signals

- Computed m/z for 1A, 2A, and 2B agree with the reported 421.1911, 674.2802, and 688.2959 within the 5 ppm mass error used by the Met-ID identification workflow
- metid.identify() called with mz=421.1911, metabolome='HMDB(All)', matrix='FMP-10', mass_error=5 returns a hit consistent with the single-derivatized dopamine species
- PPM deviations (dPPM) are computed for all three species and reported in the output table
- Comparison table is successfully written to derivatized_dopamine_mz_check.csv

## Limitations

- rdkit installation is described as nontrivial and no pinned environment or dependency versions are specified for Met-ID, rdkit, or Tauri
- FMP-10 was developed in-house, so its distribution and derivatization increment definitions may not be readily available to other users
- Reinstalling newer versions of Met-ID does not update the bundled database files; the 'com.farmbio.metid' application-data folder must be removed manually before reinstalling

## Evidence

- [readme] Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode"
- [discussion] Met-ID depends on rdkit: [evidence span withheld — anchor status: not_found]
- [methods] identify() accepts m/z values: "``identify`` accepts one or more m/z values and optional corresponding CCS values."
- [methods] search_by_name searches the database: "``search_by_name(name, ccs=False)`` searches by name."
