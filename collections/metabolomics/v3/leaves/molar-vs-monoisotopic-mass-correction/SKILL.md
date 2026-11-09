---
name: molar-vs-monoisotopic-mass-correction
description: Use when cleaning an annotated MS/MS library where stored parent masses
  are systematically offset from the monoisotopic mass implied by the SMILES/InChI
  annotation, a common annotation error in public libraries.
when_to_use_negative:
- The spectrum is unannotated or lacks SMILES/InChI/InChIKey, so there is no structure
  from which a monoisotopic mass can be recomputed.
- The parent mass is already monoisotopic and consistent with the annotation — no
  correction is warranted, and the spectrum should instead be routed to other filters
  (e.g. adduct/precursor or annotation-match repair).
edam_topics: []
tools:
- name: matchms
  role: Framework providing the Spectrum objects and the filter/pipeline mechanism
    in which the parent-mass repair filter is implemented and ordered.
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads the structural annotations (SMILES, InChI, InChIKey) used to verify
    and recompute the mass consistent with the annotated compound.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:857a66091c0f5ba69ec6a7435fde10e5f25d691eadcc06e51cd975ec395b23e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Molar vs monoisotopic mass correction

## Summary

A matchms metadata-repair filter that detects and corrects spectra whose stored parent mass was computed from the average molar mass instead of the monoisotopic mass, recomputing it from the annotated structure so the spectrum can be retained and further validated rather than discarded.

## When to use

Use when cleaning an annotated MS/MS library where stored parent masses are systematically offset from the monoisotopic mass implied by the SMILES/InChI annotation, a common annotation error in public libraries.

## When NOT to use

- The spectrum is unannotated or lacks SMILES/InChI/InChIKey, so there is no structure from which a monoisotopic mass can be recomputed.
- The parent mass is already monoisotopic and consistent with the annotation — no correction is warranted, and the spectrum should instead be routed to other filters (e.g. adduct/precursor or annotation-match repair).

## Inputs

- matchms Spectrum objects with parent mass, precursor m/z, SMILES/InChI/InChIKey metadata fields
- Annotated MS/MS library spectra (e.g. .msp/.json/.mgf imported into matchms; the paper uses the GNPS public mass spectral library)

## Outputs

- Spectrum objects with parent mass corrected to the monoisotopic mass of the annotated structure
- Per-filter counts of changed spectra collected into a processing report
- A YAML file capturing pipeline settings and filter order for reproducibility

## How to apply

Load the input spectra as matchms (v0.26.4) Spectrum objects exposing parent mass, precursor m/z, SMILES, InChI and InChIKey metadata. Implement the filter as a function receiving a spectrum and returning a spectrum: detect when the stored parent mass was calculated from the molar (average-atomic-weight) mass instead of the monoisotopic mass, and correct this by recomputing the mass as the monoisotopic mass of the annotated structure using the most common isotopes. Apply the filter within the predefined global filter order — e.g. after adduct-derivation filters that retrieve the adduct from the compound name — so the corrected mass feeds into downstream annotation-consistency checks like Repair Not Matching Annotation and Require Valid Annotation. Run it as part of the 'library cleaning' pipeline over an annotated test library (the paper uses the GNPS public mass spectral library downloaded 21-08-2023), and log per-filter change counts in the automatically generated processing report. The corrected parent mass should now equal the monoisotopic mass of the kept annotation, enabling salts/adduct-related filters to evaluate the right ion.

## Related tools

- **matchms** (Framework providing the Spectrum objects and the filter/pipeline mechanism in which the parent-mass repair filter is implemented and ordered.) — https://github.com/matchms/matchms
- **RDKit** (Loads the structural annotations (SMILES, InChI, InChIKey) used to verify and recompute the mass consistent with the annotated compound.)

## Examples

```
from matchms.filtering import repair_parent_mass_is_molar_mass; spec = repair_parent_mass_is_molar_mass(spectrum)
```

## Evaluation signals

- After correction, each modified spectrum has a parent mass equal to the monoisotopic mass of the kept annotation.
- The processing report lists a non-zero, specific count of spectra changed by this filter, distinguishable from other repair filters.
- The filter's count is reproducible across runs of the same pipeline (YAML file captures the same settings and filter order).
- Downstream filters that depend on the corrected parent mass (e.g. salt/adduct/annotation-match checks) operate on a mass consistent with the structure.
- No spectra are removed solely by this filter — it is a repair, not a discard, step.

## Limitations

- The filter only addresses parent masses computed as molar masses; other annotation inconsistencies (salt SMILES, non-matching SMILES/InChI/InChIKey) require separate repair filters in the pipeline.
- It relies on the presence of a structural annotation to recompute the mass, so it cannot act on spectra with incomplete or missing annotations.
- No plausibility checks currently consider both the corrected metadata and the measured fragments, so wrong chemical annotations consistent with the corrected mass still go unnoticed.

## Evidence

- [methods] A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this.: "A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this."
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] This report gives an overview of the number of spectra that were changed in the library by each filter.: "This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] it automatically creates a YAML file containing all settings and filters used.: "it automatically creates a YAML file containing all settings and filters used."
- [results] The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra and is available on Zenodo [5].: "The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra and is available on Zenodo [5]."
- [discussion] Future expansions might include filters that check if the fragments match the given annotation.: "Future expansions might include filters that check if the fragments match the given annotation."
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
