---
name: spectral-metadata-repair-filtering
description: Use when an annotated spectral library (e.g. the GNPS public mass spectral
  library, downloaded 21-08-2023) contains entries whose parent mass, SMILES, InChI,
  InChIKey, or adduct metadata are inconsistent and need repair before require-step
  filtering.
when_to_use_negative:
- Spectra already validated and needing no metadata repair — the library cleaning
  pipeline 'requires complete annotations' and will discard inconsistent entries;
  do not use as a blanket re-annotation tool via PubChem lookup for spectra that already
  have complete annotations.
- Spectra whose fragments need plausibility checking against the annotation — fragment-matching
  filters are only a possible future expansion, and wrong chemical annotations consistent
  with the measured mass go unnoticed.
- Standardizing instrument type or collision energies metadata — those fields are
  not yet cleaned/standardized by current matchms filters.
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Core library hosting the Spectrum class and the repair filters (Repair Smiles
    Of Salts, Repair Parent Mass Is Molar Mass, Repair Not Matching Annotation, Repair
    Adduct And Parent Mass Based On Smiles) within the library cleaning pipeline;
    version 0.26.4 was used.
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads SMILES, InChI and InChIKey annotations, parses salt-ion components from
    SMILES, and compares annotation fields to detect non-matching metadata.
- name: Python
  role: Runtime language in which each filter is implemented as a function receiving
    a spectrum and returning a spectrum.
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:38460bd6e47cea8f8d1950886f6ef2c73dd49bf3d555ff3f32140360d745012c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-metadata-repair-filtering

## Summary

A matchms-based repair stage of the MS/MS library cleaning pipeline that fixes annotation metadata errors — molar-mass parent masses, salt SMILES, and non-matching SMILES/InChI/InChIKey annotations — before discarding spectra that still fail validation.

## When to use

Use when an annotated spectral library (e.g. the GNPS public mass spectral library, downloaded 21-08-2023) contains entries whose parent mass, SMILES, InChI, InChIKey, or adduct metadata are inconsistent and need repair before require-step filtering.

## When NOT to use

- Spectra already validated and needing no metadata repair — the library cleaning pipeline 'requires complete annotations' and will discard inconsistent entries; do not use as a blanket re-annotation tool via PubChem lookup for spectra that already have complete annotations.
- Spectra whose fragments need plausibility checking against the annotation — fragment-matching filters are only a possible future expansion, and wrong chemical annotations consistent with the measured mass go unnoticed.
- Standardizing instrument type or collision energies metadata — those fields are not yet cleaned/standardized by current matchms filters.

## Inputs

- Annotated mass spectral library (e.g. GNPS public mass spectral library, MGF/msp-style entries with SMILES, InChI, InChIKey, parent mass, precursor m/z, adduct, ionmode metadata)
- MassBank library spectra (matchms pipeline also run on it)
- MoNA (Massbank of North America) library spectra

## Outputs

- Cleaned spectral library with repaired annotation metadata
- Per-filter processing report listing the number of spectra changed by each filter
- Automatically generated YAML file containing all settings and filters used
- Discard list of spectra whose SMILES, InChI, and InChIKey do not match (Require Valid Annotation)

## How to apply

Load the input spectra into matchms (v0.26.4) Spectrum objects so each spectrum exposes its parent mass, precursor m/z, adduct, ionmode, SMILES, InChI, and InChIKey metadata fields. Apply Repair Parent Mass Is Molar Mass to detect parent masses computed from the molar mass (average atomic weights) instead of the monoisotopic mass and recompute them using the most common isotopes. Apply Repair Smiles Of Salts by parsing the SMILES (e.g. 'C1=NC2=NC=NC(=C2N1)N.Cl') into separate salt-ion components with RDKit, checking whether the stored parent mass matches any single component or a combination of components, and replacing the SMILES with the matching ion portion; this filter runs after the adduct-derivation filter that retrieves the adduct from the compound name. Apply Repair Not Matching Annotation when the SMILES, InChI, and InChIKey fields describe different compounds: load all three with RDKit, keep the field whose compound matches the stored parent mass, and delete the non-matching fields. Repair Adduct And Parent Mass Based On Smiles corrects the adduct of a spectrum based on the SMILES and precursor m/z. Each filter is a function receiving a spectrum and returning a spectrum, respecting the predefined global filter order, and the library cleaning pipeline additionally runs Require Valid Annotation, which loads SMILES, InChI and InChIKey with RDKit, compares them to each other, and removes the spectrum if one is not complete or does not match. Run the pipeline to automatically generate a YAML file containing all settings and filters used, plus a processing report giving an overview of the number of spectra changed by each filter.

## Related tools

- **matchms** (Core library hosting the Spectrum class and the repair filters (Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Not Matching Annotation, Repair Adduct And Parent Mass Based On Smiles) within the library cleaning pipeline; version 0.26.4 was used.) — https://github.com/matchms/matchms
- **RDKit** (Loads SMILES, InChI and InChIKey annotations, parses salt-ion components from SMILES, and compares annotation fields to detect non-matching metadata.)
- **Python** (Runtime language in which each filter is implemented as a function receiving a spectrum and returning a spectrum.)

## Examples

```
from matchms import Pipeline; pipeline = Pipeline(); pipeline.clean_library(input_spectra, filters=["Repair Parent Mass Is Molar Mass", "Repair Smiles Of Salts", "Repair Not Matching Annotation", "Require Valid Annotation"])
```

## Evaluation signals

- Processing report shows a non-zero count of spectra changed by each repair filter, and the final cleaned GNPS library contains 448,485 curated mass spectra from the 500,569 input spectra.
- Each repaired spectrum has a parent mass equal to the monoisotopic mass of the kept annotation and a SMILES free of unmeasured salt ions.
- Require Valid Annotation removes spectra whose SMILES, InChI and InChIKey — compared via RDKit — are not complete or do not match.
- Automatically generated YAML file lists exactly the filters and settings used, enabling pipeline replay and reproducibility.
- Error-rate estimate check: re-deriving SMILES from compound names on the 413,314-spectrum GNPS error-estimation subset recovers annotations for 72.4% of spectra, fails for 27.6%, and yields a different 2D structure for 1.62% of successfully annotated spectra.

## Limitations

- Wrong chemical annotations that are consistent with the measured mass go unnoticed — current filters lack plausibility checks that consider both metadata and measured fragments.
- Instrument type and collision energies are not yet cleaned or standardized by current matchms filters.
- Repair filters operate on annotation metadata consistency; a validator tool for automatic curation during upload to public libraries remains proposed future work.
- Pipeline runtime is substantial — running the library cleaning pipeline with the filters in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.

## Evidence

- [methods] We repair this, by checking if the parent mass matches any of the separate or combinations of salt ions given in the SMILES.: [evidence span withheld — anchor status: not_found]
- [methods] A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this.: [evidence span withheld — anchor status: not_found]
- [methods] If the different metadata fields for an annotation do not match, this filter will try to correct it.: [evidence span withheld — anchor status: not_found]
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other.: [evidence span withheld — anchor status: not_found]
- [methods] This report gives an overview of the number of spectra that were changed in the library by each filter.: [evidence span withheld — anchor status: not_found]
- [methods] it automatically creates a YAML file containing all settings and filters used.: [evidence span withheld — anchor status: not_found]
- [discussion] they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed: [evidence span withheld — anchor status: not_found]
- [results] Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.: [evidence span withheld — anchor status: not_found]
- [readme] To make typical matchms workflows (data import, processing, score computations) more accessible to users, matchms now offers a `Pipeline` class to handle complex workflows.: [evidence span withheld — anchor status: not_found]
