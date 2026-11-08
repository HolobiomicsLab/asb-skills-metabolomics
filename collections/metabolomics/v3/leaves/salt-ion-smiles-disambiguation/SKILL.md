---
name: salt-ion-smiles-disambiguation
description: Use when a library spectrum's SMILES is a salt (e.g. 'C1=NC2=NC=NC(=C2N1)N.Cl'),
  the parent mass is ambiguous, and you want to keep the spectrum rather than discard
  it during annotation repair.
when_to_use_negative:
- The spectrum already has a single-component (non-salt) SMILES and a matching parent
  mass — no repair is needed and the filter is a no-op.
- The SMILES, InChI, and InChIKey fields describe different compounds; that case belongs
  to Repair Not Matching Annotation, not salt disambiguation.
- The parent mass was computed from the molar mass instead of the monoisotopic mass;
  use Repair Parent Mass Is Molar Mass instead.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Provides the Spectrum object model and the Repair Smiles Of Salts filter within
    the library cleaning pipeline; version 0.26.4 was used to run the pipelines.
  repo: github.com/matchms/matchms
- name: RDKit
  role: Parses the salt SMILES into separate ion components whose monoisotopic masses
    are matched against the stored parent mass.
- name: Python
  role: Hosts the matchms filter functions, each receiving a spectrum and returning
    a spectrum.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:e63ea74bb42483b7b2eb15d48392c784e72d8d7571b4b89e99b7fa1561778159
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# salt-ion-smiles-disambiguation

## Summary

A matchms library-cleaning filter that repairs spectra annotated with multi-component salt SMILES by testing whether the stored parent mass matches any separate salt-ion component or a combination of components, then replacing the SMILES with the matching ion portion.

## When to use

Use when a library spectrum's SMILES is a salt (e.g. 'C1=NC2=NC=NC(=C2N1)N.Cl'), the parent mass is ambiguous, and you want to keep the spectrum rather than discard it during annotation repair.

## When NOT to use

- The spectrum already has a single-component (non-salt) SMILES and a matching parent mass — no repair is needed and the filter is a no-op.
- The SMILES, InChI, and InChIKey fields describe different compounds; that case belongs to Repair Not Matching Annotation, not salt disambiguation.
- The parent mass was computed from the molar mass instead of the monoisotopic mass; use Repair Parent Mass Is Molar Mass instead.

## Inputs

- Annotated MS/MS spectra as matchms Spectrum objects with SMILES and parent mass metadata
- Salt-form SMILES strings such as 'C1=NC2=NC=NC(=C2N1)N.Cl' from libraries like the GNPS public mass spectral library (downloaded 21-08-2023)

## Outputs

- Repaired spectrum with a salt-free SMILES restricted to the measured ion portion
- Processing report entry counting spectra whose SMILES metadata was changed by Repair Smiles Of Salts

## How to apply

Load spectra as matchms (v0.26.4) Spectrum objects so parent mass, precursor m/z, adduct, and SMILES metadata are exposed. Parse the salt SMILES into its separate ion components with RDKit (e.g. 'C1=NC2=NC=NC(=C2N1)N.Cl' yields the base and 'Cl'). Compare the stored parent mass against each single component's monoisotopic mass and against combinations of components; when a match is found, replace the SMILES with the matching ion portion, leaving unmeasured salt ions out. Run the filter as a function receiving and returning a spectrum, respecting the pipeline's predefined global filter order (salt repair runs after the adduct-derivation filter that retrieves the adduct from the compound name). Log, via the processing report, the number of spectra whose SMILES metadata was changed by this filter.

## Related tools

- **matchms** (Provides the Spectrum object model and the Repair Smiles Of Salts filter within the library cleaning pipeline; version 0.26.4 was used to run the pipelines.) — github.com/matchms/matchms
- **RDKit** (Parses the salt SMILES into separate ion components whose monoisotopic masses are matched against the stored parent mass.)
- **Python** (Hosts the matchms filter functions, each receiving a spectrum and returning a spectrum.)

## Examples

```
from matchms.filtering import repair_smiles_of_salts; spectrum = repair_smiles_of_salts(spectrum)
```

## Evaluation signals

- Each repaired spectrum's SMILES no longer contains unmeasured salt ions; only the ion portion consistent with the parent mass remains.
- The processing report lists a per-filter count of spectra changed by Repair Smiles Of Salts.
- Repaired SMILES load cleanly in RDKit and the kept component's monoisotopic mass agrees with the stored parent mass.
- The YAML workflow file generated by the pipeline records this filter in its predefined position in the global filter order.
- Non-salt spectra pass through unchanged (no false modifications).

## Limitations

- The filter only checks the parent mass against salt-ion components; wrong chemical annotations that are consistent with the measured mass go unnoticed because current filters lack plausibility checks that consider both metadata and measured fragments.
- Filters checking if fragments match the given annotation are only a possible future expansion and are not implemented.
- The filter corrects the SMILES but does not itself fix a parent mass computed from the molar mass — that is a separate Repair Parent Mass Is Molar Mass filter.

## Evidence

- [methods] Salt SMILES repair by matching parent mass to salt-ion components: "We repair this, by checking if the parent mass matches any of the separate or combinations of salt ions given in the SMILES."
- [methods] Predefined global filter order places salt repair after adduct derivation: [evidence span withheld — anchor status: not_found]
- [methods] Processing report counts spectra changed by each filter: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] Library cleaning pipeline repairs errors in the annotations: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [discussion] Wrong annotations consistent with measured mass go unnoticed: "Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [discussion] Fragments-matching filters are future expansion, not implemented: "Future expansions might include filters that check if the fragments match the given annotation."
- [readme] matchms supports metadata cleaning and validation as key features: [evidence span withheld — anchor status: external_doc]
