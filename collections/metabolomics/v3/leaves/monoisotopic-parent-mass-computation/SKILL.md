---
name: monoisotopic-parent-mass-computation
description: Use when spectra carry a structure annotation plus adduct and precursor
  m/z but the parent mass may be wrong (e.
when_to_use_negative:
- When the spectrum has no structure annotation at all — the SMILES/InChI-derived
  monoisotopic mass cannot be computed, so first derive annotations via Derive Annotation
  From Compound Name (PubChem) or drop the spectrum; parent-mass cross-checking against
  PubChem-derived SMILES only applies if it matches the given parent mass.
- When metadata harmonization (ionmode, basic fields) has not yet been run — the library
  cleaning pipeline expects all default filters to have run before the repair/require
  filters.
- When wrong chemical annotations are consistent with the measured mass — the article
  notes these checks 'currently still lack plausibility checks that consider both
  metadata and measured fragments', so fragment-consistent wrong annotations will
  go unnoticed.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass
    Based On Smiles and Repair Smiles Of Salts filters and the require_* validators
    executed inside the library cleaning pipeline
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads SMILES/InChI/InChIKey annotations to obtain the monoisotopic mass of
    the annotated structure used for parent-mass cross-checks
- name: PubChem
  role: Source of canonical SMILES/InChI/InChIKey derived from compound names when
    annotations are missing; parent-mass matching gates whether the derived SMILES
    is accepted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:8c969458153ccb73125d831e81f47ee1b8c4dc0ff582513853863b7e93148554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# monoisotopic-parent-mass-computation

## Summary

Recompute and cross-check a spectrum's parent mass from the monoisotopic neutral mass of the annotated structure (via SMILES/InChI) together with its adduct, using RDKit-backed matchms filters so that molar-mass mistakes, adduct/parent-mass mismatches, and salt adducts are corrected or flagged before library spectra are trusted.

## When to use

Use when spectra carry a structure annotation plus adduct and precursor m/z but the parent mass may be wrong (e.g. computed from average molar mass instead of monoisotopic mass), or when adduct, precursor m/z and parent mass are mutually inconsistent and must be reconciled before keeping the spectrum.

## When NOT to use

- When the spectrum has no structure annotation at all — the SMILES/InChI-derived monoisotopic mass cannot be computed, so first derive annotations via Derive Annotation From Compound Name (PubChem) or drop the spectrum; parent-mass cross-checking against PubChem-derived SMILES only applies if it matches the given parent mass.
- When metadata harmonization (ionmode, basic fields) has not yet been run — the library cleaning pipeline expects all default filters to have run before the repair/require filters.
- When wrong chemical annotations are consistent with the measured mass — the article notes these checks 'currently still lack plausibility checks that consider both metadata and measured fragments', so fragment-consistent wrong annotations will go unnoticed.

## Inputs

- matchms Spectrum objects with metadata fields: adduct, precursor m/z, parent mass, SMILES (and optionally InChI/InChIKey)
- Raw spectra imported from msp, MGF, mzML, mzXML or JSON (e.g. the GNPS public mass spectral library downloaded 21-08-2023)

## Outputs

- Spectra with corrected parent mass (monoisotopic, adduct-adjusted), adduct, or salt-annotated SMILES
- Per-spectrum keep/discard decisions for require_matching_adduct_precursor_mz_and_parent_mass and require_parent_mass_match_smiles
- Processing report rows quantifying how many spectra each mass-consistency filter changed or removed

## How to apply

Load each spectrum as a matchms Spectrum with metadata fields adduct, precursor m/z, parent mass and SMILES/InChI/InChIKey. First run Repair Parent Mass Is Molar Mass: it detects the common mistake that the parent mass was calculated from the molar mass instead of the monoisotopic mass and corrects it. Then run Repair Adduct And Parent Mass Based On Smiles: it corrects the adduct based on the SMILES and precursor m/z, and for salt adducts checks whether the parent mass matches any separate or combined salt ions given in the SMILES (Repair Smiles Of Salts). RDKit loads the structure annotations to obtain the monoisotopic mass used in these cross-checks. Where a filter cannot reconcile the fields, the spectrum fails the corresponding require-style filter (e.g. require_matching_adduct_precursor_mz_and_parent_mass, require_parent_mass_match_smiles) and is discarded, since the library cleaning pipeline requires complete, consistent annotations. Record per-filter decisions rather than silently deleting spectra so the number of spectra removed by each mass-consistency filter can be audited in the processing report.

## Related tools

- **matchms** (Provides the Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles and Repair Smiles Of Salts filters and the require_* validators executed inside the library cleaning pipeline) — https://github.com/matchms/matchms
- **RDKit** (Loads SMILES/InChI/InChIKey annotations to obtain the monoisotopic mass of the annotated structure used for parent-mass cross-checks)
- **PubChem** (Source of canonical SMILES/InChI/InChIKey derived from compound names when annotations are missing; parent-mass matching gates whether the derived SMILES is accepted)

## Examples

```
python -m matchms.library_cleaning --input gnps_library.msp --filters repair_parent_mass_is_molar_mass,repair_adduct_and_parent_mass_based_on_smiles,repair_smiles_of_salts --output cleaned.msp --report report.json
```

## Evaluation signals

- After Repair Parent Mass Is Molar Mass, the corrected parent mass equals monoisotopic neutral mass + adduct mass adjustment, not the average molar mass.
- For kept spectra, adduct, precursor m/z and parent mass are mutually consistent within matchms' tolerance (require_matching_adduct_precursor_mz_and_parent_mass passes).
- For kept spectra, parent mass matches the SMILES-derived monoisotopic mass (require_parent_mass_match_smiles passes).
- Processing report shows non-zero (and auditable) counts of spectra changed by each repair filter and removed by each require filter; overall removal scale should be consistent with the GNPS run (500,569 spectra before cleaning down to 448,485 after the full pipeline).
- Every spectrum in the output carries a decision for each of the five validation filters (no silently dropped rows).

## Limitations

- These mass-consistency checks cannot detect wrong chemical annotations that are consistent with the measured mass; fragment-matching filters are only a possible future expansion.
- Repair filters can only correct errors that follow from the SMILES/precursor m/z pair; if the annotation itself is wrong (e.g. PubChem-derived SMILES not matching the parent mass), the spectrum is discarded rather than repaired.
- Running the full library cleaning pipeline (including these filters) on the GNPS library of 500,569 spectra took 6 h and 45 min — not a lightweight per-file operation.
- The pipeline relies on metadata standardization that public libraries lack; many entries are not computer-interpretable, so repair filters depend on upstream derivation steps being correct.

## Evidence

- [methods] Molar-mass repair rationale: "A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this."
- [methods] Adduct repair from SMILES: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [methods] Salt repair: "We repair this, by checking if the parent mass matches any of the separate or combinations of salt ions given in the SMILES."
- [methods] Annotation validation via RDKit: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [discussion] Limitation: no fragment plausibility: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [intro] Pipeline scope: "we here introduce a comprehensive pipeline for library cleaning within the matchms framework"
- [results] GNPS scale: "Before cleaning, the GNPS library contained 500,569 spectra."
