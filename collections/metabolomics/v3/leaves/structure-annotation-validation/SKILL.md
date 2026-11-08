---
name: structure-annotation-validation
description: Use when MS/MS library entries carry structural annotations (SMILES/InChI/InChIKey)
  whose consistency against adduct, precursor m/z, parent mass and ionmode must be
  checked or repaired before library searching or ML training.
when_to_use_negative:
- The goal is to verify that measured fragments match the given annotation — fragment-matching
  plausibility filters are not implemented and wrong annotations consistent with the
  measured mass go unnoticed
- The fields needing cleanup are instrument type or collision energies — these are
  not yet standardized by current matchms filters
- Spectra lack any structure annotation or derivable compound name — validation can
  neither repair nor confirm them and they will be removed by Require Valid Annotation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the pipeline (SpectrumProcessor/Pipeline, v0.26.4) and the repair/require
    filters (Derive Annotation From Compound Name, Repair Smiles Of Salts, Repair
    Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair
    Not Matching Annotation, Require Valid Annotation, Require Parent Mass Match Smiles,
    Require Matching Adduct And Ionmode)
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and pairwise-compares SMILES, InChI and InChIKey to decide annotation
    completeness and consistency (Require Valid Annotation)
- name: PubChem
  role: Source from which canonical SMILES, InChI and InChIKey are derived from compound
    names, accepted only when they match the given parent mass
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:74f4f5827375ad33961b02f5607ec646727dd3b6bdd00d6a2c7d750d097a856e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# structure-annotation-validation

## Summary

Validate and repair chemical structure annotations (SMILES, InChI, InChIKey) of MS/MS library spectra for internal consistency with adduct, precursor m/z and parent mass, using matchms filters backed by RDKit and PubChem, removing spectra with irreparable annotations.

## When to use

Use when MS/MS library entries carry structural annotations (SMILES/InChI/InChIKey) whose consistency against adduct, precursor m/z, parent mass and ionmode must be checked or repaired before library searching or ML training.

## When NOT to use

- The goal is to verify that measured fragments match the given annotation — fragment-matching plausibility filters are not implemented and wrong annotations consistent with the measured mass go unnoticed
- The fields needing cleanup are instrument type or collision energies — these are not yet standardized by current matchms filters
- Spectra lack any structure annotation or derivable compound name — validation can neither repair nor confirm them and they will be removed by Require Valid Annotation

## Inputs

- Mass spectral library file (e.g. GNPS public no-propagated .msp snapshot, 21-08-2023) with per-spectrum metadata: SMILES, InChI, InChIKey, compound name, adduct, precursor m/z, parent mass, ionmode
- Other public libraries with native metadata styles (MassBank, MoNA)

## Outputs

- Cleaned spectral library (e.g. 448,485 curated GNPS spectra, deposited on Zenodo)
- Auto-generated YAML file with all settings and filters used (rerunnable)
- Processing report counting per filter how many spectra were removed, had metadata changed, or had peaks changed

## How to apply

Build the matchms (v0.26.4) library-cleaning pipeline so repair filters run before requirement filters: Derive Annotation From Compound Name (fetches canonical SMILES/InChI/InChIKey from PubChem, kept only if they match the parent mass), Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass (fixes parent masses computed from molar instead of monoisotopic mass), Repair Adduct And Parent Mass Based On Smiles, and Repair Not Matching Annotation. Then apply validation requirements: Require Valid Annotation loads SMILES/InChI/InChIKey with RDKit and compares them pairwise, removing spectra where one field is incomplete or inconsistent; Require Parent Mass Match Smiles, Require Matching Adduct/Precursor m/z/Parent Mass, and Require Matching Adduct And Ionmode enforce metadata-structure agreement. Collect the auto-generated YAML file and the per-filter processing report; on the 21-08-2023 GNPS no-propagated snapshot (500,569 spectra) this retained 448,485 spectra and removed 31,758 (52,084 repaired vs 83,843 removed without repair).

## Related tools

- **matchms** (Provides the pipeline (SpectrumProcessor/Pipeline, v0.26.4) and the repair/require filters (Derive Annotation From Compound Name, Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, Require Valid Annotation, Require Parent Mass Match Smiles, Require Matching Adduct And Ionmode)) — https://github.com/matchms/matchms
- **RDKit** (Loads and pairwise-compares SMILES, InChI and InChIKey to decide annotation completeness and consistency (Require Valid Annotation))
- **PubChem** (Source from which canonical SMILES, InChI and InChIKey are derived from compound names, accepted only when they match the given parent mass)

## Examples

```
from matchms.pipeline import Pipeline; pipeline = Pipeline("cleaning_workflow.yaml"); cleaned_library = pipeline.run("gnps_library.msp")
```

## Evaluation signals

- Processing report counts match expected flow: 500,569 input GNPS spectra, 31,758 removed with repair functions enabled, 448,485 in the cleaned library (52,084 spectra had metadata repaired vs 83,843 removed without repair)
- For every retained spectrum, SMILES, InChI and InChIKey loaded in RDKit are complete and mutually consistent (Require Valid Annotation passes)
- Parent mass equals the monoisotopic mass implied by the SMILES and is consistent with the stated adduct and precursor m/z (molar-mass mistakes corrected)
- Adduct and ionmode agree with precursor m/z and parent mass after Repair Adduct And Parent Mass Based On Smiles
- Rerunning the auto-generated YAML file reproduces the same cleaned library and per-filter counts

## Limitations

- Plausibility checks that consider both metadata and measured fragments are lacking; wrong chemical annotations that are consistent with the measured mass go unnoticed
- Fragment-matching filters (checking whether fragments match the annotation) are only a possible future expansion, not implemented
- Instrument type and collision energies are important metadata fields that might require additional standardization but are not yet cleaned
- Run time is substantial: 6 h 45 min for the 500,569-spectrum GNPS library in matchms 0.26.4

## Evidence

- [methods] Require Valid Annotation loads and compares SMILES/InChI/InChIKey via RDKit, removing incomplete or mismatching spectra: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] The parent mass filter corrects the common molar-mass-instead-of-monoisotopic-mass mistake: "A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this."
- [results] Before cleaning, the GNPS library contained 500,569 spectra; the cleaned library holds 448,485 curated spectra: "Before cleaning, the GNPS library contained 500,569 spectra."
- [discussion] Current filters lack plausibility checks combining metadata and measured fragments: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [readme] matchms is a Python package for importing, processing, cleaning and comparing MS/MS data: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
