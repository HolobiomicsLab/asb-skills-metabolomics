---
name: compound-name-to-smiles-derivation
description: Use when library spectra carry only a compound name (no SMILES/InChI)
  and machine-readable structures must be recovered, e.g. many GNPS-library annotations
  are compound names without SMILES.
when_to_use_negative:
- Annotations already carry a valid SMILES/InChI — the filter targets spectra that
  are only compound names without SMILES
- The compound name is absent, ambiguous, or the parent mass is unreliable; the PubChem
  lookup will not be able to confirm the correct structure via the mass-match check
- You need to verify fragments match the annotation — the article notes fragment-matching
  filters are only a possible future expansion, so this skill cannot detect mass-consistent
  but structurally wrong annotations
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: matchms
  role: Provides the 'derive annotation from compound name' filter executed within
    the library-cleaning pipeline (v0.26.4)
  repo: https://github.com/matchms/matchms
- name: PubChem
  role: Source of canonical SMILES, InChI and InChIKey for compound-name lookup, gated
    by a parent-mass match
- name: RDKit
  role: Loads the recovered SMILES, InChI and InChIKey and compares them to flag incorrect
    or incomplete 2D structures
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:6974e889e408fb32fc500e605fe2acdf89d2090ceed851e668e3b0ebdb3b9056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# compound-name-to-smiles-derivation

## Summary

Derives canonical SMILES, InChI and InChIKey from compound-name-only annotations in mass spectral libraries via PubChem lookup, accepting only structures whose PubChem mass matches the spectrum's parent mass — a matchms library-cleaning filter for annotating otherwise unannotated spectra.

## When to use

Use when library spectra carry only a compound name (no SMILES/InChI) and machine-readable structures must be recovered, e.g. many GNPS-library annotations are compound names without SMILES.

## When NOT to use

- Annotations already carry a valid SMILES/InChI — the filter targets spectra that are only compound names without SMILES
- The compound name is absent, ambiguous, or the parent mass is unreliable; the PubChem lookup will not be able to confirm the correct structure via the mass-match check
- You need to verify fragments match the annotation — the article notes fragment-matching filters are only a possible future expansion, so this skill cannot detect mass-consistent but structurally wrong annotations

## Inputs

- Mass spectral library spectra (e.g. GNPS public library, downloaded 21-08-2023) with compound-name metadata but missing or removed SMILES annotation
- Spectrum metadata containing parent mass and compound name
- Stored compound-name-to-structure PubChem lookup cache (optional, to avoid repeated queries)

## Outputs

- Spectrum annotations enriched with derived canonical SMILES, InChI and InChIKey
- Processing report entries counting spectra for which structures could or could not be derived, and spectra with mismatching derived structures
- Spectra removed from the library when no mass-consistent PubChem structure is found or the annotation is incomplete

## How to apply

Within a matchms (v0.26.4) library-cleaning pipeline, apply the 'derive annotation from compound name' filter to each spectrum lacking a SMILES annotation: query PubChem for the compound name to obtain canonical SMILES, InChI and InChIKey. Accept the derived structure only if the PubChem-reported mass matches the parent mass in the spectrum metadata — this mass-match gate prevents name collisions pulling in the wrong structure. Load the recovered SMILES, InChI and InChIKey with RDKit and compare them to flag incomplete or inconsistent annotations. Reuse a stored compound-name→structure lookup file to minimize repeated PubChem queries, which matters because a full run over 500,569 GNPS spectra took 6 h 45 min. Validate the result by comparing derived structures to any original annotations (error-rate estimation showed 27.6% of spectra were not re-derivable and 1.62% of re-annotated spectra had a different 2D structure).

## Related tools

- **matchms** (Provides the 'derive annotation from compound name' filter executed within the library-cleaning pipeline (v0.26.4)) — https://github.com/matchms/matchms
- **PubChem** (Source of canonical SMILES, InChI and InChIKey for compound-name lookup, gated by a parent-mass match)
- **RDKit** (Loads the recovered SMILES, InChI and InChIKey and compares them to flag incorrect or incomplete 2D structures)

## Examples

```
from matchms import Pipeline; Pipeline.from_yaml('library_cleaning.yaml').run('gnps_library.msp')
```

## Evaluation signals

- Fraction of spectra for which SMILES could not be derived from the compound name reproduces the reported 27.6% on the GNPS error-estimation subset (413,314 spectra)
- Among successfully annotated spectra, the fraction with a different 2D structure than the original annotation is 1.62%
- Every accepted annotation has SMILES, InChI and InChIKey loaded by RDKit without error and mutually consistent, per the 'Require Valid Annotation' logic
- Each derived structure's PubChem mass matches the spectrum's parent mass in the metadata — mass mismatches are rejected or the spectrum removed
- The pipeline run emits a YAML file and processing report documenting which filter changed or removed how many spectra

## Limitations

- The SMILES is only derived from PubChem based on a compound name if it matches the given parent mass; wrong chemical annotations consistent with the measured mass go unnoticed because plausibility checks considering both metadata and measured fragments are lacking
- Fragment-matching filters (checking if fragments match the given annotation) are not implemented — only a possible future expansion
- Name-based lookup fails for a large fraction of entries: 27.6% of GNPS spectra in the error-estimation subset could not be re-annotated this way
- No plausibility checks that consider both metadata and measured fragments exist yet, so 1.62% of re-annotated spectra differed from the original annotation in 2D structure

## Evidence

- [methods] Many annotations in the GNPS library are only compound names without SMILES. This filter derives the canonical SMILES, InChI and InChIKey from PubChem.: "Many annotations in the GNPS library are only compound names without SMILES. This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [discussion] the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [results] Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [results] After removing the SMILES annotation from the selected spectra the SMILES were derived from the compound name.: "After removing the SMILES annotation from the selected spectra the SMILES were derived from the compound name."
