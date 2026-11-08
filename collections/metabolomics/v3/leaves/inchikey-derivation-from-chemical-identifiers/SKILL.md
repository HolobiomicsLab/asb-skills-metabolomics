---
name: inchikey-derivation-from-chemical-identifiers
description: Use when MS/MS library spectra (e.g. GNPS ALL_GNPS.json, 154,820 spectra)
  lack InChIKey annotations but carry compound name, molecular formula, or parent
  mass that can be resolved against PubChem.
when_to_use_negative:
- Spectra already carry InChIKey annotations after matchms cleaning — no PubChem lookup
  is needed
- Spectra lack usable compound name, molecular formula, or parent mass metadata, so
  there is no basis for a PubChem query
- The goal is spectral similarity scoring (e.g. Spec2Vec, cosine) rather than structural
  annotation of library entries
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3335
tools:
- name: matchms
  role: Cleans and corrects spectrum metadata (ionmode, charge, compound name, molecular
    formula, parent mass) before PubChem lookup
  repo: https://github.com/matchms/matchms
- name: pubchempy
  role: Programmatic interface for the automated lookup search against PubChem to
    resolve compound identifiers to InChIKeys
- name: PubChem
  role: External compound database queried by name, formula, and parent mass to retrieve
    structures and InChIKeys
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:6617aa110c64a9e73c89e98ef511d47c1b1ce8df95e92cbbf1187bf24b9a1b1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - matchms
  - pubchempy
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inchikey-derivation-from-chemical-identifiers

## Summary

Links mass spectra in the GNPS ALL_GNPS library to InChIKeys by combining matchms metadata cleaning with an extensive automated lookup search against PubChem via pubchempy, so structure-based analyses (e.g. Tanimoto similarity) become possible.

## When to use

Use when MS/MS library spectra (e.g. GNPS ALL_GNPS.json, 154,820 spectra) lack InChIKey annotations but carry compound name, molecular formula, or parent mass that can be resolved against PubChem.

## When NOT to use

- Spectra already carry InChIKey annotations after matchms cleaning — no PubChem lookup is needed
- Spectra lack usable compound name, molecular formula, or parent mass metadata, so there is no basis for a PubChem query
- The goal is spectral similarity scoring (e.g. Spec2Vec, cosine) rather than structural annotation of library entries

## Inputs

- GNPS ALL_GNPS.json library (154,820 spectra, https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json)
- Spectrum metadata fields: compound name, molecular formula, parent mass, ionmode, charge

## Outputs

- Annotated library gnps_allannotated_cleaned.json with full and planar InChIKeys attached to spectrum metadata
- Per-spectrum InChIKey annotations (~128,042 spectra linked; 14,978 unique planar InChIKeys)

## How to apply

Load the GNPS ALL_GNPS.json library (https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json) into matchms spectrum objects, then clean and correct the provided metadata with matchms, harmonizing fields such as ionmode, charge, compound name, molecular formula, and parent mass. For spectra still lacking InChIKey annotations after cleaning, run an extensive automated lookup search against PubChem via pubchempy, querying by compound name, molecular formula, and parent mass. For matched compounds derive both the full and planar InChIKeys and attach them to the spectrum metadata, then save the annotated library as gnps_allannotated_cleaned.json. Validate against the published preprocessing outcome: 128,042 of 154,820 spectra (14,978 unique planar InChIKeys) should be linked, matching Zenodo 10.5281/zenodo.3979010.

## Related tools

- **matchms** (Cleans and corrects spectrum metadata (ionmode, charge, compound name, molecular formula, parent mass) before PubChem lookup) — https://github.com/matchms/matchms
- **pubchempy** (Programmatic interface for the automated lookup search against PubChem to resolve compound identifiers to InChIKeys)
- **PubChem** (External compound database queried by name, formula, and parent mass to retrieve structures and InChIKeys)

## Examples

```
import pubchempy as pcp; comps = pcp.get_compounds(compound_name, 'name')  # then match formula/parent mass and derive InChIKey
```

## Evaluation signals

- 128,042 out of 154,820 spectra are linked to an InChIKey after the pipeline
- 14,978 unique planar InChIKeys are derived across the annotated library
- Output matches the published preprocessing outcome on Zenodo (10.5281/zenodo.3979010)
- All annotated spectra carry both a full and a planar InChIKey in their matchms metadata
- Metadata fields (ionmode, charge, compound name, molecular formula, parent mass) are harmonized before querying

## Limitations

- PubChem lookup depends on the quality of compound name, molecular formula, and parent mass metadata; ambiguous or erroneous names can yield wrong structures
- A substantial fraction of spectra (26,778 of 154,820) remain unannotated after cleaning and lookup
- In the published workflow only 94,121 spectra had InChIKey annotations from metadata cleaning alone; the remainder required the PubChem search, so skipping the lookup sharply reduces coverage

## Evidence

- [discussion] Metadata cleaning rationale: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [discussion] PubChem lookup outcome: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [readme] matchms capabilities: [evidence span withheld — anchor status: external_doc]
