---
name: adduct-ionmode-cross-validation
description: Use when cleaning an MS/MS spectral library where spectra carry adduct,
  ionmode, precursor m/z and parent mass metadata (e.g. .msp/.MGF GNPS entries) and
  you must detect inconsistent annotations before matching.
when_to_use_negative:
- When the library spectra lack structured adduct or ionmode metadata (e.g. unannotated
  spectra) — the cross-validation cannot be performed and the filters would discard
  valid entries
- When only peak-intensity cleaning is needed (peak filtering / intensity normalization)
  with no annotation metadata to cross-check
- When you intend to repair, rather than discard, an inconsistent adduct — the article's
  repair filters (Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching
  Annotation) are the appropriate choice instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the filtering functions require_matching_adduct_and_ionmode and require_matching_adduct_precursor_mz_and_parent_mass
    applied within the library-cleaning pipeline (version 0.26.4 in the article)
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads the SMILES structure to compute the parent mass used for the precursor-m/z
    cross-check
- name: Python
  role: Execution environment for the matchms filtering workflow and the wrapper that
    records per-spectrum filter decisions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:c54a0bb29cdc68f2e4a684e8631f3a7dc94189e82fef4e4c5c96e34c845b6cbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# adduct-ionmode-cross-validation

## Summary

Cross-validates a spectrum's adduct annotation against the recorded ion mode and, additionally, against the precursor m/z and parent mass derived from the structure annotation, discarding spectra whose metadata fields disagree.

## When to use

Use when cleaning an MS/MS spectral library where spectra carry adduct, ionmode, precursor m/z and parent mass metadata (e.g. .msp/.MGF GNPS entries) and you must detect inconsistent annotations before matching.

## When NOT to use

- When the library spectra lack structured adduct or ionmode metadata (e.g. unannotated spectra) — the cross-validation cannot be performed and the filters would discard valid entries
- When only peak-intensity cleaning is needed (peak filtering / intensity normalization) with no annotation metadata to cross-check
- When you intend to repair, rather than discard, an inconsistent adduct — the article's repair filters (Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation) are the appropriate choice instead

## Inputs

- matchms Spectrum objects loaded from a public MS/MS spectral library (e.g. GNPS public mass spectral library, .msp/.MGF format)
- Per-spectrum metadata fields: adduct, ionmode, precursor m/z, parent mass, SMILES/InChI/InChIKey annotation

## Outputs

- Per-spectrum filter decisions (keep/discard) for require_matching_adduct_and_ionmode and require_matching_adduct_precursor_mz_and_parent_mass
- Overall keep/discard flag per spectrum when combined with other require_* filters
- Validation report (e.g. validation_filter_results.csv with columns spectrum_id, filter_matching_adduct_ionmode, filter_matching_adduct_precursor_parent, overall_keep)
- matchms processing report listing how many spectra were removed by each filter

## How to apply

Load spectra as matchms Spectrum objects from the GNPS public mass spectral library (or a similar .msp/MGF library) and apply the matchms.filtering functions require_matching_adduct_and_ionmode and require_matching_adduct_precursor_mz_and_parent_mass within the library-cleaning pipeline. The first filter checks whether the adduct string (e.g. '[M+H]+') implies the recorded ion mode; the second derives the expected precursor m/z from the adduct and the parent mass computed from the SMILES structure (via RDKit) and compares it with the recorded precursor m/z, flagging the spectrum when the derived value falls outside tolerance. Each filter's keep/discard decision is recorded per spectrum (as in validation_filter_results.csv with columns filter_matching_adduct_ionmode and filter_matching_adduct_precursor_parent) so decisions can be aggregated into an overall keep flag. The rationale is that the article reports these as required 'library cleaning' filters — spectra whose adduct contradicts its ion mode, or whose precursor m/z, parent mass and structure are mutually inconsistent, are removed so downstream library searching or machine-learning training is not corrupted.

## Related tools

- **matchms** (Provides the filtering functions require_matching_adduct_and_ionmode and require_matching_adduct_precursor_mz_and_parent_mass applied within the library-cleaning pipeline (version 0.26.4 in the article)) — https://github.com/matchms/matchms
- **RDKit** (Loads the SMILES structure to compute the parent mass used for the precursor-m/z cross-check)
- **Python** (Execution environment for the matchms filtering workflow and the wrapper that records per-spectrum filter decisions)

## Examples

```
python scripts/clean_library.py --input gnps_library.msp --output validation_filter_results.csv
```

## Evaluation signals

- Each spectrum in the input library must have a populated value in the filter_matching_adduct_ionmode and filter_matching_adduct_precursor_parent columns of the output CSV (no missing decisions)
- The overall_keep flag must equal True only when all five require_* filters pass; any spectrum failing the adduct/ionmode or precursor/parent cross-check must be flagged False
- Removal counts reported for these filters in the matchms processing report must be consistent with the article's scale of spectrum removals (GNPS library: 500,569 spectra before cleaning down to 448,485 after the full pipeline)
- The script must run without error across the full loaded library and produce one output row per input spectrum
- Spectra that pass should show derived precursor m/z consistent with the recorded value within the filter's tolerance; disagreements indicate a failed cross-validation

## Limitations

- Wrong chemical annotations that are consistent with the measured mass go unnoticed — the filter only checks metadata/structure consistency, not whether fragments match the annotation
- The filters discard inconsistent spectra rather than repairing them; repair must be done beforehand with filters such as Repair Adduct And Parent Mass Based On Smiles or Repair Not Matching Annotation
- Fragment-matching filters are only a possible future expansion and are not implemented in matchms

## Evidence

- [methods] require_matching_adduct_and_ionmode: [evidence span withheld — anchor status: not_found]
- [methods] Repair Adduct And Parent Mass Based On Smiles: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [methods] Require Valid Annotation: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [intro] annotation validation scope: "structure annotation validation through adduct, precursor m/z, and annotation comparison and harmonization"
- [methods] library cleaning pipeline definition: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [results] removal counts scale: "Before cleaning, the GNPS library contained 500,569 spectra."
- [readme] matchms extensibility: [evidence span withheld — anchor status: external_doc]
- [discussion] plausibility-check limitation: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
