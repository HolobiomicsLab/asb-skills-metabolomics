---
name: adduct-based-parent-mass-verification
description: Use when cleaning annotated MS/MS library spectra whose adduct, parent
  mass, or precursor m/z may be inconsistent with the SMILES annotation — e.g. GNPS
  entries where parent mass was computed from molar mass or the adduct was misassigned.
when_to_use_negative:
- When you need plausibility checks that consider both metadata and measured fragments
  — wrong chemical annotations consistent with the measured mass will go unnoticed
  in this pipeline
- When spectra lack any resolvable structure annotation (no SMILES and no compound
  name matchable via PubChem) — the verification filters depend on a valid annotation
  and the spectra will simply be removed by Require Valid Annotation
- When you need standardized instrument-type or collision-energy metadata — current
  matchms filters do not clean or standardize these fields
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the Pipeline class and the repair/requirement filters (Repair Parent
    Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Require Parent
    Mass Match Smiles, Require Matching Adduct And Ionmode); version 0.26.4 used to
    run the pipelines
  repo: github.com/matchms/matchms
- name: PubChem
  role: Source of canonical SMILES, InChI and InChIKey derived from compound names
    so the parent mass and adduct can be verified against a structure
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey annotations to validate annotation
    consistency
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:45a7fb8a4da7cf43796cde6e3c1f32bb02dbb7ecda42e518f588a19623f4a01a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Adduct-based parent mass verification

## Summary

A matchms filter sequence that checks and repairs the consistency between a spectrum's structure annotation (SMILES), adduct, precursor m/z, and parent mass, correcting masses computed from molar rather than monoisotopic mass and repairing or removing spectra whose adduct/mass metadata disagrees with the SMILES.

## When to use

Use when cleaning annotated MS/MS library spectra whose adduct, parent mass, or precursor m/z may be inconsistent with the SMILES annotation — e.g. GNPS entries where parent mass was computed from molar mass or the adduct was misassigned.

## When NOT to use

- When you need plausibility checks that consider both metadata and measured fragments — wrong chemical annotations consistent with the measured mass will go unnoticed in this pipeline
- When spectra lack any resolvable structure annotation (no SMILES and no compound name matchable via PubChem) — the verification filters depend on a valid annotation and the spectra will simply be removed by Require Valid Annotation
- When you need standardized instrument-type or collision-energy metadata — current matchms filters do not clean or standardize these fields

## Inputs

- MS/MS spectra with structure annotations (SMILES, InChI, InChIKey), adduct, precursor m/z, and parent mass metadata (e.g. GNPS public mass spectral library, 21-08-2023 no-propagated snapshot; supports mzML, mzXML, msp, MGF, JSON formats)
- Library spectra with compound-name-only annotations that can be resolved to canonical SMILES via PubChem
- A YAML pipeline settings file specifying the filter order and parameters (auto-generated for reproducibility)

## Outputs

- Cleaned spectral library with repaired parent-mass/adduct metadata and removal of spectra failing annotation-consistency requirements
- Processing report counting, per filter, how many spectra were removed, had metadata changed, or had peaks changed
- Auto-generated YAML file containing all settings and filters used, rerunnable for reproducibility

## How to apply

In matchms 0.26.4, build the library cleaning pipeline with the create_workflow function so filters run in the predefined global order, ensuring dependent filters run after their prerequisites. First run Derive Annotation From Compound Name, which fetches canonical SMILES/InChI/InChIKey from PubChem (only accepting a hit if the derived SMILES matches the given parent mass). Then apply the repair filters: Repair Parent Mass Is Molar Mass corrects a parent mass calculated from molar mass instead of monoisotopic mass, Repair Smiles Of Salts checks whether the parent mass matches separate or combined salt ions in the SMILES, and Repair Adduct And Parent Mass Based On Smiles corrects the adduct from the SMILES and precursor m/z. Finally enforce Require Parent Mass Match Smiles and Require Matching Adduct And Ionmode, which remove spectra whose adduct/precursor m/z/parent mass still disagree with the annotation, collecting the auto-generated YAML settings file and the processing report that counts per filter how many spectra were removed or had metadata changed.

## Related tools

- **matchms** (Provides the Pipeline class and the repair/requirement filters (Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Require Parent Mass Match Smiles, Require Matching Adduct And Ionmode); version 0.26.4 used to run the pipelines) — github.com/matchms/matchms
- **PubChem** (Source of canonical SMILES, InChI and InChIKey derived from compound names so the parent mass and adduct can be verified against a structure)
- **RDKit** (Loads and compares SMILES, InChI and InChIKey annotations to validate annotation consistency)

## Examples

```
from matchms.pipeline import Pipeline; pipeline = Pipeline("library_cleaning.yaml")  # matchms >= 0.18.0 Pipeline class supports yaml-defined workflows; run with matchms 0.26.4
```

## Evaluation signals

- On the GNPS public library (500,569 input spectra), the pipeline with repair functions enabled removed exactly 31,758 spectra (versus 83,843 without repair), repairing metadata of 52,084 spectra and yielding 448,485 curated spectra
- The processing report must attribute metadata changes per filter — Repair Parent Mass Is Molar Mass and Repair Adduct And Parent Mass Based On Smiles should show repaired (not removed) counts for correctable entries
- Post-run, every retained spectrum's parent mass must equal the monoisotopic mass (not the molar mass) implied by its SMILES, and its adduct must be consistent with the precursor m/z
- The auto-generated YAML file must reproduce the identical filter set and order when rerun
- Reported runtime on the 500,569-spectrum GNPS library was 6 h 45 min with matchms 0.26.4 — large deviations warrant checking filter configuration

## Limitations

- The pipeline lacks plausibility checks that consider both metadata and measured fragments, so wrong chemical annotations that are consistent with the measured mass go unnoticed
- Fragment-matching filters (checking if fragments match the given annotation) are only a possible future expansion, not implemented
- SMILES is only derived from PubChem based on a compound name if it matches the given parent mass, so unresolvable names leave annotations incomplete and the spectra are ultimately removed

## Evidence

- [methods] A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this.: "A common mistake is that the parent mass is calculated from the molar mass instead of the monoisotopic mass. This filter corrects this."
- [methods] Corrects the adduct of a spectrum based on the SMILES and precursor m/z.: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [discussion] Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed: "Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [results] Before cleaning, the GNPS library contained 500,569 spectra.: "Before cleaning, the GNPS library contained 500,569 spectra."
- [readme] Matchms offers an array of tools for metadata cleaning and validation, alongside basic peak filtering, to ensure data accuracy and integrity.: [evidence span withheld — anchor status: external_doc]
