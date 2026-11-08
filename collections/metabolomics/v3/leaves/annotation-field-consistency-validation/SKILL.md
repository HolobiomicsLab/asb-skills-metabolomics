---
name: annotation-field-consistency-validation
description: Use when cleaning annotated MS/MS library spectra whose SMILES, InChI,
  and InChIKey metadata fields are mutually inconsistent (e.g. after salt repair or
  adduct correction) and must be reconciled against the parent mass.
when_to_use_negative:
- The spectrum has no structure annotations at all (no SMILES/InChI/InChIKey) — there
  is nothing to reconcile; downstream Require Valid Annotation would remove it instead.
- All three annotation fields already describe the same compound and match the parent
  mass — the filter would be a no-op and other cleaning steps (peak filtering, intensity
  normalization) should be prioritized.
- You need to validate that measured fragments match the annotation — the article
  notes such plausibility checks are not yet implemented.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: matchms 0.26.4
  role: Framework providing the Repair Not Matching Annotation filter within its library
    cleaning pipeline
  repo: github.com/matchms/matchms
- name: RDKit
  role: Used to load and compare SMILES, InChI, and InChIKey annotations and to identify
    which field matches the parent mass
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:1fb26c87cad3713b8d53ee7767dd3b8f75a934927e1d77d083a0b4268524994b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - matchms 0.26.4
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# annotation-field-consistency-validation

## Summary

A matchms library-cleaning repair step (Repair Not Matching Annotation) that detects and corrects spectra whose SMILES, InChI, and InChIKey annotation fields describe different compounds, keeping only the field consistent with the stored parent mass.

## When to use

Use when cleaning annotated MS/MS library spectra whose SMILES, InChI, and InChIKey metadata fields are mutually inconsistent (e.g. after salt repair or adduct correction) and must be reconciled against the parent mass.

## When NOT to use

- The spectrum has no structure annotations at all (no SMILES/InChI/InChIKey) — there is nothing to reconcile; downstream Require Valid Annotation would remove it instead.
- All three annotation fields already describe the same compound and match the parent mass — the filter would be a no-op and other cleaning steps (peak filtering, intensity normalization) should be prioritized.
- You need to validate that measured fragments match the annotation — the article notes such plausibility checks are not yet implemented.

## Inputs

- matchms Spectrum objects with SMILES, InChI, InChIKey and parent mass metadata (e.g. from an .msp file such as the GNPS public mass spectral library)

## Outputs

- Spectrum objects whose inconsistent annotation fields have been deleted so only the annotation matching the parent mass remains

## How to apply

Within the matchms (v0.26.4) library cleaning pipeline, apply the Repair Not Matching Annotation filter after metadata harmonization and other repair filters (e.g. Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass). For each Spectrum, load the SMILES, InChI, and InChIKey with RDKit and compare them to each other; if the three fields do not describe the same compound, determine which annotation matches the stored parent mass and keep that field, deleting the non-matching annotation fields. This corrects annotation errors without discarding the spectrum, unlike downstream filters such as Require Valid Annotation, which removes spectra with incomplete or non-matching annotations. Since matchms 0.18.0 the filter can be assembled into a Pipeline whose settings are automatically written to a YAML file for reproducibility, and a processing report logs how many spectra each filter changed.

## Related tools

- **matchms 0.26.4** (Framework providing the Repair Not Matching Annotation filter within its library cleaning pipeline) — github.com/matchms/matchms
- **RDKit** (Used to load and compare SMILES, InChI, and InChIKey annotations and to identify which field matches the parent mass)

## Examples

```
from matchms.importing import load_from_msp; spectra = list(load_from_msp('gnps_library.msp')); from matchms.filtering import repair_not_matching_annotation; cleaned = [repair_not_matching_annotation(s) for s in spectra]
```

## Evaluation signals

- The processing report shows a non-zero count of spectra changed by Repair Not Matching Annotation when run on the GNPS library.
- After filtering, each modified spectrum retains only annotation fields that RDKit parses to the same compound as the kept one.
- No spectrum in the output carries both a valid parent mass and a contradictory (non-matching) annotation pair.
- The pipeline runs reproducibly: the auto-generated YAML file records the filter among the library-cleaning settings.
- Downstream Require Valid Annotation removes fewer spectra than it would without this repair.

## Limitations

- Cannot detect wrong chemical annotations that are consistent with the measured mass — the article notes such plausibility checks are still lacking.
- Only reconciles SMILES/InChI/InChIKey against the parent mass; metadata like instrument type and collision energies are not yet standardized by current matchms filters.
- Filter order matters: it should run after salt and molar-mass repair filters in the predefined global order for correct results.

## Evidence

- [methods] Repair Not Matching Annotation corrects annotations whose metadata fields disagree: [evidence span withheld — anchor status: not_found]
- [methods] RDKit comparison of the three identifier fields: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] matchms version used to run the pipeline: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] Processing report records per-filter spectrum changes: "This report gives an overview of the number of spectra that were changed in the library by each filter."
- [readme] Pipeline settings persisted to YAML for reproducibility: [evidence span withheld — anchor status: external_doc]
- [discussion] Wrong-but-mass-consistent annotations go unnoticed: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
