---
name: annotation-consistency-checking
description: Use when a mass spectral library entry carries SMILES/InChI/InChIKey
  metadata whose mutual agreement must be verified before the spectrum is accepted,
  e.g. during library cleaning or validation-filter reporting.
when_to_use_negative:
- Spectra whose annotations were derived solely from compound names via PubChem and
  whose 2D structure consistency with the parent mass has already been confirmed —
  this filter would only re-check the same three fields.
- When the goal is to check whether measured fragments plausibly match the given annotation
  — the article notes such fragment-matching filters are a possible future expansion
  and not part of the matchms pipeline.
- When only the metadata (instrument type, collision energies) need standardization
  — the article states these fields are not yet cleaned by current matchms filters.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Runs the annotation-consistency filters (Require Valid Annotation, Repair
    Not Matching Annotation, Repair Smiles Of Salts, Repair Adduct And Parent Mass
    Based On Smiles, Require Parent Mass Match Smiles) within the library cleaning
    pipeline; v0.26.4 was used.
  repo: github.com/matchms/matchms
- name: RDKit
  role: Loads the SMILES, InChI and InChIKey annotations and compares them to detect
    incomplete or inconsistent fields.
- name: PubChem
  role: Source of canonical SMILES/InChI/InChIKey when annotations are derived from
    compound names; derived structure is only accepted if its PubChem mass matches
    the parent mass.
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:45195441357167be62f3a1f507ddbc298e7ddafe0f8ff1a06d6e218b1fb83222
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# annotation-consistency-checking

## Summary

This skill cross-validates SMILES, InChI and InChIKey annotations attached to MS/MS library spectra by loading all three with RDKit and comparing them, removing spectra whose annotation fields are incomplete or mutually inconsistent.

## When to use

Use when a mass spectral library entry carries SMILES/InChI/InChIKey metadata whose mutual agreement must be verified before the spectrum is accepted, e.g. during library cleaning or validation-filter reporting.

## When NOT to use

- Spectra whose annotations were derived solely from compound names via PubChem and whose 2D structure consistency with the parent mass has already been confirmed — this filter would only re-check the same three fields.
- When the goal is to check whether measured fragments plausibly match the given annotation — the article notes such fragment-matching filters are a possible future expansion and not part of the matchms pipeline.
- When only the metadata (instrument type, collision energies) need standardization — the article states these fields are not yet cleaned by current matchms filters.

## Inputs

- matchms Spectrum objects with SMILES, InChI, InChIKey, adduct, ionmode, precursor m/z and parent mass metadata
- GNPS public mass spectral library (e.g. downloaded 21-08-2023)
- stored compound-name lookup file for PubChem-derived annotations

## Outputs

- Per-spectrum pass/fail decisions for each annotation-consistency filter
- Aggregated keep/discard flag per spectrum (overall_keep)
- Processing report of spectra changed or removed by each filter
- Cleaned library with 448,485 curated spectra (from 500,569 input spectra on GNPS)

## How to apply

Apply the matchms 'Require Valid Annotation' filter to each Spectrum: SMILES, InChI and InChIKey are loaded by RDKit and compared to each other; if any of the three fields is incomplete or does not match the others the spectrum is discarded. Complementary repair filters run first: 'Repair Not Matching Annotation' attempts to fix metadata fields that disagree, 'Repair Smiles Of Salts' checks whether the parent mass matches separate or combined salt ions in the SMILES, and 'Repair Adduct And Parent Mass Based On Smiles' corrects the adduct from the SMILES and precursor m/z. Additional checks tie the annotation to measured quantities, e.g. 'Require Parent Mass Match Smiles' and 'Require Matching Adduct Precursor Mz And Parent Mass', which must all pass (alongside require_minimum_number_of_high_peaks and require_matching_adduct_and_ionmode) for a spectrum to be kept. Aggregate the per-filter pass/fail decisions into a per-spectrum overall keep/discard flag so removal counts can be compared against the processing report.

## Related tools

- **matchms** (Runs the annotation-consistency filters (Require Valid Annotation, Repair Not Matching Annotation, Repair Smiles Of Salts, Repair Adduct And Parent Mass Based On Smiles, Require Parent Mass Match Smiles) within the library cleaning pipeline; v0.26.4 was used.) — github.com/matchms/matchms
- **RDKit** (Loads the SMILES, InChI and InChIKey annotations and compares them to detect incomplete or inconsistent fields.)
- **PubChem** (Source of canonical SMILES/InChI/InChIKey when annotations are derived from compound names; derived structure is only accepted if its PubChem mass matches the parent mass.)

## Examples

```
from matchms.filtering import require_valid_annotation; spectrum = require_valid_annotation(spectrum)
```

## Evaluation signals

- Each spectrum yields a decision for all five validation filters (minimum high peaks, matching adduct/ionmode, matching adduct/precursor m/z/parent mass, parent mass match SMILES, valid annotation) with no missing entries.
- Overall keep flag is true only when every individual filter passes.
- Removal counts are consistent with the reported GNPS scale: 500,569 spectra before cleaning, 448,485 after the full pipeline.
- The processing report generated by matchms lists how many spectra were changed or removed by each filter.
- Among spectra whose SMILES were re-derived from compound names, 1.62% should differ in 2D structure and 27.6% should be non-derivable, matching the error-rate estimates.

## Limitations

- Wrong chemical annotations that are consistent with the measured mass (but not the fragments) go unnoticed — no plausibility checks consider both metadata and measured fragments.
- Filters that check whether measured fragments match the given annotation do not exist yet; they are only a possible future expansion.
- Metadata fields such as instrument type and collision energies are not yet standardized by current matchms filters.
- Deriving SMILES from compound names via PubChem fails for a substantial fraction (27.6% on the GNPS error-estimation subset).

## Evidence

- [methods] Require Valid Annotation loads SMILES/InChI/InChIKey with RDKit and compares them: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] Repair Smiles Of Salts checks salt-ion parent-mass matches: "We repair this, by checking if the parent mass matches any of the separate or combinations of salt ions given in the SMILES."
- [methods] Repair Adduct And Parent Mass Based On Smiles corrects the adduct from SMILES and precursor m/z: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [methods] Repair Not Matching Annotation fixes disagreement between metadata fields: "If the different metadata fields for an annotation do not match, this filter will try to correct it."
- [discussion] PubChem-derived SMILES must match the parent mass: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [discussion] Lack of fragment-based plausibility checks is a limitation: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [results] GNPS spectra counts before/after cleaning: "Before cleaning, the GNPS library contained 500,569 spectra."
- [readme] matchms Pipeline class and YAML workflow support: [evidence span withheld — anchor status: external_doc]
