---
name: smiles-inchikey-structural-consistency
description: Use when cleaning annotated MS/MS library entries where SMILES, InChI
  and InChIKey metadata must be checked for completeness and mutual agreement before
  spectra are admitted to a cleaned library.
when_to_use_negative:
- The spectrum has no structural annotation at all and only a compound name — use
  'Derive Annotation From Compound Name' (PubChem-based SMILES/InChI/InChIKey derivation)
  first, otherwise every spectrum would be discarded
- You need to verify that measured fragments are consistent with the annotation —
  the article states current filters 'lack plausibility checks that consider both
  metadata and measured fragments', so fragment-matching validation is out of scope
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: matchms
  role: Provides the Require Valid Annotation filter and the library cleaning pipeline
    that invokes it on Spectrum objects
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey inside the filter; an
    incomplete or non-matching annotation fails the check
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:9d22b7a7036a7725940704930aa6cb08feeb48551338e1dfb84e2438f3ef586d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# smiles-inchikey-structural-consistency

## Summary

Validate that the SMILES, InChI, and InChIKey annotations of a mass-spectrum record describe the same complete chemical structure, using RDKit to load and cross-compare the three representations so incomplete or mutually inconsistent annotation fields cause the spectrum to be discarded.

## When to use

Use when cleaning annotated MS/MS library entries where SMILES, InChI and InChIKey metadata must be checked for completeness and mutual agreement before spectra are admitted to a cleaned library.

## When NOT to use

- The spectrum has no structural annotation at all and only a compound name — use 'Derive Annotation From Compound Name' (PubChem-based SMILES/InChI/InChIKey derivation) first, otherwise every spectrum would be discarded
- You need to verify that measured fragments are consistent with the annotation — the article states current filters 'lack plausibility checks that consider both metadata and measured fragments', so fragment-matching validation is out of scope

## Inputs

- MS/MS Spectrum objects with SMILES, InChI and InChIKey metadata fields (e.g. from an .msp library file such as the GNPS public mass spectral library downloaded 21-08-2023)

## Outputs

- Keep/discard decision per spectrum; cleaned library with only fully consistent structural annotations; processing report counting spectra changed/removed by the Require Valid Annotation filter

## How to apply

Within the matchms library cleaning pipeline, apply the 'Require Valid Annotation' filter to each Spectrum: RDKit loads the SMILES, InChI and InChIKey fields and cross-compares them to each other, so the three representations are checked for structural agreement rather than treated independently. If any of the three fields is missing, incomplete, or inconsistent with the others, the spectrum is removed from the library. Run this filter after metadata-repair filters (e.g. Repair Not Matching Annotation, Repair Adduct And Parent Mass Based On Smiles) so annotations have had a chance to be corrected first; the library cleaning pipeline runs all default filters plus annotation repair before requiring complete annotations. Consistency with related structural checks matters here: require_parent_mass_match_smiles validates that the parent mass matches the SMILES-derived mass, and require_matching_adduct_precursor_mz_and_parent_mass ties adduct, precursor m/z and parent mass together, so an annotation passing the RDKit cross-comparison should also be checked against these mass-consistency filters. Record per-spectrum keep/discard decisions in a processing report so the number of spectra removed by this filter can be audited against the overall cleaning run (the GNPS run removed spectra from 500,569 down to 448,485 across all filters).

## Related tools

- **matchms** (Provides the Require Valid Annotation filter and the library cleaning pipeline that invokes it on Spectrum objects) — https://github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey inside the filter; an incomplete or non-matching annotation fails the check)

## Examples

```
from matchms.filtering import require_valid_annotation; spectrum = require_valid_annotation(spectrum)
```

## Evaluation signals

- Every kept spectrum has a SMILES, an InChI and an InChIKey that RDKit can load and that cross-compare as the same structure
- Processing report lists a non-zero count of spectra removed specifically by the Require Valid Annotation filter
- Removal counts across the pipeline stay consistent with the GNPS cleaning run scale (500,569 spectra before cleaning to 448,485 after)
- No kept spectrum has an annotation field that is empty, unparsable by RDKit, or structurally mismatched with the other two representations

## Limitations

- The filter only checks consistency among SMILES, InChI and InChIKey; a chemically wrong but internally consistent annotation that matches the measured parent mass will go unnoticed, as the article notes wrong annotations consistent with the measured mass escape current checks
- SMILES derived from compound names rely on PubChem matching the given parent mass; incomplete coverage means some entries cannot be rescued before this filter
- Filtering out metadata fields such as instrument type or collision energies is not yet standardized, so annotation validity says nothing about acquisition metadata quality
- Fragment-matching filters (fragments matching the given annotation) are only a proposed future expansion, not implemented

## Evidence

- [methods] RDKit cross-comparison semantics: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] Pipeline composition of filters: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [discussion] Known failure mode of annotation checks: "Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [results] Scale of the cleaning run: "Before cleaning, the GNPS library contained 500,569 spectra."
- [readme] Tool scope from README: [evidence span withheld — anchor status: external_doc]
