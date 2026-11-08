---
name: mass-spectrometry-error-rate-estimation
description: Use when quantifying how many spectra in a mass spectral library have
  missing or incorrect structure annotations, by re-deriving SMILES from compound
  names and comparing against original annotations.
when_to_use_negative:
- When the goal is to actually clean/repair a library rather than quantify error rates
  — use the full library cleaning pipeline instead of the error-estimation subset
  protocol
- When spectra lack compound names in their metadata, so re-derivation from names
  is impossible by definition
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms 0.26.4
  role: Runs the 'derive annotation from compound name' filter that looks up canonical
    SMILES, InChI and InChIKey in PubChem and only accepts a structure whose PubChem
    mass matches the parent mass
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads the derived SMILES, InChI and InChIKey and compares them to flag incorrect
    2D structures
- name: PubChem
  role: Source of canonical SMILES, InChI and InChIKey looked up from compound names
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:e4ac80e70c82052842d7ee664cc589606ecc867b391e4258e7fc3067df3b3af7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms 0.26.4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-spectrometry-error-rate-estimation

## Summary

Estimate annotation error rates in a mass spectral library by removing SMILES annotations and re-deriving structures from compound names, quantifying how often structures cannot be re-derived or differ in 2D structure.

## When to use

Use when quantifying how many spectra in a mass spectral library have missing or incorrect structure annotations, by re-deriving SMILES from compound names and comparing against original annotations.

## When NOT to use

- When the goal is to actually clean/repair a library rather than quantify error rates — use the full library cleaning pipeline instead of the error-estimation subset protocol
- When spectra lack compound names in their metadata, so re-derivation from names is impossible by definition

## Inputs

- Mass spectral library spectra with annotations and compound names (e.g. GNPS public mass spectral library, downloaded 21-08-2023)
- Parent mass and precursor m/z metadata per spectrum
- Stored compound-name lookup file to minimize repeated PubChem queries

## Outputs

- Fraction of spectra for which SMILES were successfully derived from the compound name (e.g. 72.4%)
- Fraction of spectra not derivable from compound name (e.g. 27.6%)
- Fraction of annotated spectra with an incorrect 2D structure (e.g. 1.62%)

## How to apply

Select the subset of spectra carrying valid annotations plus a compound name from the library being assessed (e.g. the GNPS public mass spectral library downloaded 21-08-2023, with 500,569 spectra). Remove the existing SMILES annotation from each selected spectrum to create an error-estimation test set. Run the matchms (v0.26.4) 'derive annotation from compound name' filter, which looks up canonical SMILES, InChI and InChIKey in PubChem and only accepts a structure whose PubChem mass matches the given parent mass in the metadata. Load the recovered SMILES, InChI and InChIKey with RDKit and compare them to flag incorrect 2D structures. Compute the aggregate rates: fraction of spectra for which SMILES were successfully derived from the compound name (72.4% on the GNPS error-estimation subset of 413,314 spectra), fraction not derivable (27.6%), and fraction of annotated spectra with an incorrect 2D structure (1.62%).

## Related tools

- **matchms 0.26.4** (Runs the 'derive annotation from compound name' filter that looks up canonical SMILES, InChI and InChIKey in PubChem and only accepts a structure whose PubChem mass matches the parent mass) — https://github.com/matchms/matchms
- **RDKit** (Loads the derived SMILES, InChI and InChIKey and compares them to flag incorrect 2D structures)
- **PubChem** (Source of canonical SMILES, InChI and InChIKey looked up from compound names)

## Examples

```
from matchms.filtering import derive_annotation_from_compound_name; derive_annotation_from_compound_name(spectrum)
```

## Evaluation signals

- Computed rates reproduce the reported values of 72.4% derived, 27.6% not derivable, and 1.62% incorrect 2D structures among annotated spectra on the GNPS error-estimation subset
- The number of spectra in the error-estimation subset (413,314) matches the reported count
- RDKit comparison confirms that derived SMILES, InChI and InChIKey are internally consistent for successfully annotated spectra
- PubChem lookups only accept a structure when the PubChem mass matches the given parent mass

## Limitations

- The SMILES is only derived from PubChem based on a compound name if it matches the given parent mass, so correct annotations may be rejected when parent mass metadata is wrong
- Plausibility checks that consider both metadata and measured fragments are still lacking, so wrong chemical annotations consistent with the measured mass go unnoticed

## Evidence

- [results] 72.4% derived, 27.6% not derivable, 1.62% incorrect 2D structures: [evidence span withheld — anchor status: not_found]
- [discussion] parent mass matching constraint: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [methods] filter method: "This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [methods] matchms version used: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] RDKit comparison: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
