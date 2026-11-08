---
name: mass-spectral-adduct-repair
description: Use when annotated MS/MS spectra (e.g. from GNPS or MoNA libraries) carry
  an adduct or parent mass inconsistent with their SMILES annotation and precursor
  m/z, and the annotation needs repair before library matching or ML training.
when_to_use_negative:
- Spectra lack any SMILES (or other structure annotation) — the filter cannot test
  candidate adducts against a structure, so Derive Annotation From Compound Name via
  PubChem should run first or the spectrum should be discarded by Require Valid Annotation.
- Wrong chemical annotations that happen to be consistent with the measured precursor
  mass — the article notes such annotations go unnoticed because plausibility checks
  considering measured fragments are not yet implemented.
- Fragment-level validation is required (checking whether measured fragments match
  the annotation) — that is only a proposed future expansion in matchms, not part
  of this filter.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the repair_adduct_based_on_smiles filter and the Pipeline class to
    run it as part of the library-cleaning workflow (version 0.26.4 used in the article).
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Used by matchms to load and cross-compare SMILES, InChI and InChIKey structure
    annotations after adduct repair.
- name: Python
  role: Runtime language in which matchms and the pipeline filters are executed.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:2c48d1614f15feb4661cf2259d77c0f5a297dd021b4c30531ea3b63fe25763b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectral-adduct-repair

## Summary

Corrects a spectrum's adduct annotation and precursor parent mass by testing which adduct explains the observed precursor m/z given the spectrum's SMILES structure, as implemented in the matchms library-cleaning pipeline for GNPS and other public MS/MS libraries.

## When to use

Use when annotated MS/MS spectra (e.g. from GNPS or MoNA libraries) carry an adduct or parent mass inconsistent with their SMILES annotation and precursor m/z, and the annotation needs repair before library matching or ML training.

## When NOT to use

- Spectra lack any SMILES (or other structure annotation) — the filter cannot test candidate adducts against a structure, so Derive Annotation From Compound Name via PubChem should run first or the spectrum should be discarded by Require Valid Annotation.
- Wrong chemical annotations that happen to be consistent with the measured precursor mass — the article notes such annotations go unnoticed because plausibility checks considering measured fragments are not yet implemented.
- Fragment-level validation is required (checking whether measured fragments match the annotation) — that is only a proposed future expansion in matchms, not part of this filter.

## Inputs

- Annotated MS/MS spectra (GNPS public mass spectral library, MoNA, MassBank) in mzML, mzXML, msp, MGF, JSON or metabolomics-USI format
- Per-spectrum metadata: precursor m/z, adduct, SMILES (and ideally InChI/InChIKey), compound name

## Outputs

- Spectra with corrected adduct and parent mass consistent with the SMILES annotation and precursor m/z
- Processing report counting how many spectra the adduct-repair filter changed
- YAML file recording all settings and filters used for reproducibility

## How to apply

Load annotated spectra (mzML, mzXML, msp, MGF or JSON are supported by matchms) and run the repair_adduct_based_on_smiles filter: for each spectrum, the filter tests common adducts against the SMILES-derived monoisotopic mass and the spectrum's precursor m/z, selecting the adduct that best explains the precursor. If a distinct adduct explains the precursor m/z better than the current annotation, the adduct is corrected and the parent mass is repaired accordingly. Run this filter after annotation-harmonization steps like Repair Smiles Of Salts and Repair Parent Mass Is Molar Mass, so that salt-decomposed SMILES and molar-mass-vs-monoisotopic-mass errors are already fixed and the adduct search has a correct structure mass to work with. Follow up with Repair Not Matching Annotation and Require Valid Annotation (SMILES, InChI, InChIKey loaded by RDKit and cross-compared) to catch residual inconsistencies. Matchms 0.26.4 was used for these pipelines; the filter runs as part of the library-cleaning pipeline which also auto-generates a YAML settings file and a processing report recording how many spectra each filter changed.

## Related tools

- **matchms** (Provides the repair_adduct_based_on_smiles filter and the Pipeline class to run it as part of the library-cleaning workflow (version 0.26.4 used in the article).) — https://github.com/matchms/matchms
- **RDKit** (Used by matchms to load and cross-compare SMILES, InChI and InChIKey structure annotations after adduct repair.)
- **Python** (Runtime language in which matchms and the pipeline filters are executed.)

## Examples

```
from matchms.filtering import repair_adduct_based_on_smiles; s = repair_adduct_based_on_smiles(s)
```

## Evaluation signals

- After repair, the adduct should explain the precursor m/z: recomputing the parent mass from the SMILES-derived monoisotopic mass plus the chosen adduct should match the spectrum's precursor m/z.
- Error-rate estimates for the GNPS error-estimation subset: adduct could not be derived for ~0.02% of spectra, and of the remaining 99.98% of spectra 0.024% had an incorrect adduct after the filter — your run should land in the same order of magnitude.
- Processing report should show a non-zero count of spectra changed by the adduct-repair filter; zero changes over a large annotated library suggests the filter did not run or metadata was already harmonized.
- Downstream Require Valid Annotation should not remove spectra because SMILES and InChIKey disagree — if it does, the adduct repair may have corrupted the annotation linkage.
- YAML settings file auto-generated by the pipeline should list the repair filter with the parameters used, matching the intended workflow.

## Limitations

- Cannot repair adducts when no SMILES (or structure) annotation is present; annotations must be complete or derived first.
- Does not catch wrong chemical annotations that are consistent with the measured mass — no plausibility checks considering both metadata and measured fragments exist in the current pipeline.
- Fragment-matching filters (checking if fragments match the given annotation) are only proposed as future expansion, not implemented.
- Other metadata such as instrument type or collision energies are not yet standardized by current matchms filters.

## Evidence

- [methods] Corrects the adduct of a spectrum based on the SMILES and precursor m/z.: [evidence span withheld — anchor status: not_found]
- [intro] structure annotation validation through adduct, precursor m/z, and annotation comparison and harmonization: "structure annotation validation through adduct, precursor m/z, and annotation comparison and harmonization"
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [discussion] no plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
