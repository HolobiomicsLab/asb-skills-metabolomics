---
name: rdkit-structure-parsing
description: Use when library spectra carry structure annotations in more than one
  format (SMILES, InChI, InChIKey) that must be parsed, checked for mutual consistency,
  or repaired against a stored parent mass before downstream matching.
when_to_use_negative:
- Spectra have no structure annotation at all and the goal is to derive one from a
  compound name via PubChem rather than validate existing fields
- You need to check whether measured fragment peaks match the annotation — the paper
  notes such fragment-matching plausibility checks are only a future expansion, not
  implemented
- Input is peak-level data (mzML, mzXML) that first needs peak picking; RDKit parsing
  here operates on metadata strings, not spectra peaks
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: RDKit
  role: Parses SMILES/InChI/InChIKey annotations, decomposes salt SMILES into ion
    components, and compares structure fields against the stored parent mass to drive
    repair filters
- name: matchms
  role: Provides the Spectrum objects and the pipeline/filters framework (e.g. Repair
    Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Not Matching Annotation)
    that invoke RDKit for annotation repair
  repo: github.com/matchms/matchms
- name: Python
  role: Language in which matchms and its RDKit-based filters are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:2d150a803d24950daf068df642f522db1a9700ae41f8584ba8837ae43947e411
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rdkit-structure-parsing

## Summary

Load and cross-validate chemical structure annotations (SMILES, InChI, InChIKey) with RDKit so that spectra in MS/MS libraries can be repaired, compared, and consistently validated against each other.

## When to use

Use when library spectra carry structure annotations in more than one format (SMILES, InChI, InChIKey) that must be parsed, checked for mutual consistency, or repaired against a stored parent mass before downstream matching.

## When NOT to use

- Spectra have no structure annotation at all and the goal is to derive one from a compound name via PubChem rather than validate existing fields
- You need to check whether measured fragment peaks match the annotation — the paper notes such fragment-matching plausibility checks are only a future expansion, not implemented
- Input is peak-level data (mzML, mzXML) that first needs peak picking; RDKit parsing here operates on metadata strings, not spectra peaks

## Inputs

- matchms Spectrum objects with metadata fields parent mass, precursor m/z, adduct, ionmode, SMILES, InChI, InChIKey
- Library spectra annotations such as GNPS public mass spectral library entries (downloaded 21-08-2023)
- Compound-name-only annotations needing derived structure fields

## Outputs

- Repaired spectrum metadata with consistent SMILES, InChI, InChIKey and parent mass
- Salt-free SMILES where the parent mass matched a single salt ion or a combination of salt ions
- A processing report counting spectra changed by each repair filter

## How to apply

Load each spectrum's SMILES, InChI, and InChIKey metadata fields into RDKit and compare the three descriptions of the compound to each other, as matchms does in its annotation-validation filters. Use RDKit to parse salt SMILES such as 'C1=NC2=NC=NC(=C2N1)N.Cl' into separate salt-ion components, then check whether the stored parent mass matches any single component or a combination of components; if a component matches, replace the SMILES with that ion portion. When SMILES, InChI, or InChIKey fields describe different compounds, keep only the field whose compound matches the stored parent mass and delete the non-matching annotations. Rationale: public libraries like GNPS contain many annotation errors (molar-mass parents, salt SMILES, mismatched fields) that must be corrected before spectra can serve as reliable training or matching data; RDKit provides the parsing and canonicalization needed to detect these inconsistencies automatically. Version context: matchms 0.26.4 uses RDKit for these repairs, and a full GNPS cleaning run over 500,569 spectra took 6 h and 45 min.

## Related tools

- **RDKit** (Parses SMILES/InChI/InChIKey annotations, decomposes salt SMILES into ion components, and compares structure fields against the stored parent mass to drive repair filters)
- **matchms** (Provides the Spectrum objects and the pipeline/filters framework (e.g. Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Not Matching Annotation) that invoke RDKit for annotation repair) — github.com/matchms/matchms
- **Python** (Language in which matchms and its RDKit-based filters are implemented)

## Examples

```
from matchms.importing import load_from_msp; spectra = load_from_msp('gnps_library.msp'); from matchms.filtering import repair_smiles_of_salts; spectra = [repair_smiles_of_salts(s) for s in spectra]
```

## Evaluation signals

- After running Repair Not Matching Annotation, each surviving spectrum's SMILES, InChI, and InChIKey all describe the same compound as loaded by RDKit
- After Repair Smiles Of Salts, repaired spectra have a SMILES free of unmeasured salt ions and a parent mass equal to the monoisotopic mass of the kept annotation
- After Repair Parent Mass Is Molar Mass, the parent mass equals the monoisotopic mass computed from the most common isotopes, no longer the molar mass
- The processing report lists, per filter, the number of spectra whose metadata was changed, e.g. counts comparable to the GNPS run that processed 500,569 spectra
- Require Valid Annotation removes any spectrum where one of SMILES/InChI/InChIKey is incomplete or does not match the others

## Limitations

- Plausibility checks that consider both metadata and measured fragments are lacking; wrong chemical annotations consistent with the measured mass go unnoticed
- Metadata fields such as instrument type and collision energies are not yet standardized by current matchms filters
- The SMILES is only derived from PubChem based on a compound name if it matches the given parent mass
- A validator tool for automatic curation during upload to public libraries is only proposed as future work

## Evidence

- [methods] Repair Smiles Of Salts parses salt SMILES into components: "We repair this, by checking if the parent mass matches any of the separate or combinations of salt ions given in the SMILES."
- [methods] Repair Not Matching Annotation corrects conflicting annotation fields: "If the different metadata fields for an annotation do not match, this filter will try to correct it."
- [methods] RDKit loads and compares SMILES, InChI, InChIKey: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
- [methods] Require Valid Annotation removes spectra with incomplete or non-matching annotations: "If one of these is not complete or does not match the spectrum will be removed."
- [methods] Matchms version used for the pipeline: "Matchms version 0.26.4 was used to run these pipelines."
- [discussion] Wrong annotations consistent with measured mass go unnoticed: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [discussion] PubChem-derived SMILES must match parent mass: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
- [readme] Matchms README describes metadata cleaning and validation capabilities: [evidence span withheld — anchor status: external_doc]
