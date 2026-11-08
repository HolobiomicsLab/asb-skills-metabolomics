---
name: pubchem-compound-name-annotation-derivation
description: Use when a mass spectral library entry (e.g. from the GNPS public library)
  carries only a compound name in its annotation and lacks SMILES/InChI/InChIKey,
  and you want to repair rather than remove that spectrum.
when_to_use_negative:
- The spectrum already has a complete SMILES/InChI/InChIKey annotation — use the validation
  filters (Require Valid Annotation via RDKit) instead of re-deriving from name.
- No compound name is present in the metadata, so PubChem lookup has no seed field.
- You need fragment-level plausibility checking — the article notes current filters
  do not check whether measured fragments match the annotation.
edam_operation: http://edamontology.org/operation_3762
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: matchms 0.26.4
  role: Framework whose library-cleaning pipeline runs the Derive Annotation From
    Compound Name filter in the predefined global filter order
  repo: https://github.com/matchms/matchms
- name: PubChem
  role: Lookup service from which the canonical SMILES, InChI and InChIKey are derived
    from the compound name
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey in the downstream Require Valid
    Annotation filter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:0f95d991c16130e37a1821f1668e94ffb33c2bb6a972d3515711439621cca65e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms 0.26.4
  edam_operation: http://edamontology.org/operation_3762
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pubchem-compound-name-annotation-derivation

## Summary

A matchms library-cleaning filter that fills in missing structure annotations (SMILES, InChI, InChIKey) by looking up the compound name in PubChem, so that name-only GNPS entries become machine-interpretable instead of being discarded.

## When to use

Use when a mass spectral library entry (e.g. from the GNPS public library) carries only a compound name in its annotation and lacks SMILES/InChI/InChIKey, and you want to repair rather than remove that spectrum.

## When NOT to use

- The spectrum already has a complete SMILES/InChI/InChIKey annotation — use the validation filters (Require Valid Annotation via RDKit) instead of re-deriving from name.
- No compound name is present in the metadata, so PubChem lookup has no seed field.
- You need fragment-level plausibility checking — the article notes current filters do not check whether measured fragments match the annotation.

## Inputs

- Mass spectral library in GNPS public-library form (e.g. no-propagated version, 21-08-2023 snapshot; 500,569 input spectra) loaded in matchms
- Spectra whose annotation metadata contain a compound name but no SMILES/InChI/InChIKey

## Outputs

- Repaired spectra with canonical SMILES, InChI and InChIKey derived from PubChem
- Per-spectrum library (e.g. 448,485 curated GNPS spectra) plus an auto-generated YAML workflow file and a processing report counting metadata changes per filter

## How to apply

Within the matchms library-cleaning pipeline (matchms 0.26.4), apply the 'Derive Annotation From Compound Name' filter to spectra whose annotations are only compound names: the filter derives the canonical SMILES, InChI and InChIKey from PubChem based on the compound name. Guard the derivation with the parent mass: the SMILES is only accepted if it matches the given parent mass, preventing wrong lookups from homonymous names. Order this repair filter before the requirement filters (e.g. Require Valid Annotation via RDKit) so repaired spectra can pass the later validation instead of being removed. Judge success via the auto-generated processing report, which counts per filter how many spectra had metadata changed; on the 500,569-spectrum GNPS library this repair strategy reduced removals from 83,843 to 31,758 spectra (52,084 repaired).

## Related tools

- **matchms 0.26.4** (Framework whose library-cleaning pipeline runs the Derive Annotation From Compound Name filter in the predefined global filter order) — https://github.com/matchms/matchms
- **PubChem** (Lookup service from which the canonical SMILES, InChI and InChIKey are derived from the compound name)
- **RDKit** (Loads and compares SMILES, InChI and InChIKey in the downstream Require Valid Annotation filter)

## Examples

```
from matchms.importing.load_spectra import load_spectra; pipeline.run(load_spectra('GNPS-library.mgf'))  # create_workflow(...) pipeline including Derive Annotation From Compound Name
```

## Evaluation signals

- Processing report shows a nonzero count of spectra with metadata changed by the Derive Annotation From Compound Name filter (52,084 repaired on the GNPS run).
- Total removed spectra with repair functions enabled is 31,758 versus 83,843 without repair for the 500,569-spectrum GNPS library.
- Every derived SMILES matches the spectrum's given parent mass before acceptance.
- Downstream Require Valid Annotation (RDKit) confirms derived SMILES/InChI/InChIKey are mutually consistent.
- YAML workflow file records the filter settings so the run can be rerun reproducibly.

## Limitations

- The SMILES is only derived from PubChem based on a compound name if it matches the given parent mass, so name-only entries with missing or wrong parent masses may remain unrepaired.
- Wrong chemical annotations that are consistent with the measured mass will go unnoticed; no plausibility checks consider both metadata and measured fragments.
- Instrument type and collision energy metadata fields are not standardized by these filters.
- The GNPS run took 6 h 45 min for 500,569 spectra, so runtime is substantial for large libraries.

## Evidence

- [methods] Derive Annotation From Compound Name: "Many annotations in the GNPS library are only compound names without SMILES. This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [discussion] mass-gated PubChem lookup: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [results] repair impact on removals: "Before cleaning, the GNPS library contained 500,569 spectra."
- [methods] processing report: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] validation via RDKit: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
