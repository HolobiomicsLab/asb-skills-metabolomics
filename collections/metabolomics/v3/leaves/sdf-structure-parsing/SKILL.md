---
name: sdf-structure-parsing
description: Use when canonical SMILES/structures have already been resolved for tentative
  compounds (e.g. via categorate() against PubChem/LOTUS/KEGG) and structural descriptors
  or maximum common substructure comparisons are needed.
when_to_use_negative:
- Structures could not be resolved — the decontaminate step removes chemicals lacking
  a PubChem entry or canonical SMILES, leaving nothing to parse
- Data are MS2, MS3, or ion-mobility MS2 spectra, which uafR does not support because
  it depends on published spectra
- The input is a raw GC-MS peak table (Component.RT, Component.Area, Base.Peak.MZ,
  Match.Factor) that has not yet been spreadOut()/mzExacto()-processed or structure-resolved
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: uafR
  role: Wraps the categorate() workflow that loads structures, applies detail = 'research'
    / cache / throttle parameters, and returns FMCS, BestChemMatch, and SourceCoverage
    tables
  repo: https://github.com/castratton/uafR
- name: ChemmineR
  role: read.SDFset() parses resolved structures into an SDFset; substructure descriptor
    computation (rings, subgroups, atom counts, charges) is performed with ChemmineR
- name: fmcsR
  role: fmcsBatch() compares each compound against every entry in the user-defined
    chemical library to produce Tanimoto structural-match frames
- name: PubChem
  role: Source of published structures and chemical data from which resolved structures
    and canonical SMILES are obtained
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:f3c32b65079c80ef0356aa2238649e46b26f540bd1aca3e166ac392e1ac256f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - ChemmineR
  - LOTUS
  - PubChem
  - R
  - fmcsR
  - uafR
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# sdf-structure-parsing

## Summary

Parse resolved chemical structures into an SDFset with ChemmineR and compute substructure descriptors (rings, subgroups, atom counts, charges) so that GC-MS tentative identifications can be structurally compared to a user-labeled chemical library via fmcsR.

## When to use

Use when canonical SMILES/structures have already been resolved for tentative compounds (e.g. via categorate() against PubChem/LOTUS/KEGG) and structural descriptors or maximum common substructure comparisons are needed.

## When NOT to use

- Structures could not be resolved — the decontaminate step removes chemicals lacking a PubChem entry or canonical SMILES, leaving nothing to parse
- Data are MS2, MS3, or ion-mobility MS2 spectra, which uafR does not support because it depends on published spectra
- The input is a raw GC-MS peak table (Component.RT, Component.Area, Base.Peak.MZ, Match.Factor) that has not yet been spreadOut()/mzExacto()-processed or structure-resolved

## Inputs

- Resolved chemical structures (canonical SMILES or SDF content) for tentatively identified GC-MS compounds
- User-labeled chemical library (data frame, e.g. wide-format GroupA/GroupB columns)
- Query compound name or PubChem CID vector

## Outputs

- ChemmineR SDFset object with computed substructure descriptors (rings, subgroups, atom counts, charges)
- FMCS descriptor table consumed by exactoThese() subsetBy = 'FMCS' filters
- Strong-match frame (Tanimoto > 0.95) and moderate-match frame (Tanimoto > 0.85) plus BestChemMatch names
- Validated categorate result object with SourceCoverage and DataDictionary

## How to apply

After uafR's categorate() resolves structures and fetches categorical flags from LOTUS, FEMA, KEGG, MeSH, and FDA/SPL, parse the resolved structures with ChemmineR::read.SDFset() to build an SDFset object. Compute substructure descriptors on that SDFset — rings, subgroups, atom counts, and charges — which appear downstream in the FMCS table used by exactoThese() filters (e.g. subsetBy = 'FMCS' with MW, Rings, Groups, Atoms, or NCharges thresholds). Then run fmcsR::fmcsBatch() to compare each compound against every entry in the user-labeled chemical library, splitting results into strong matches (Tanimoto > 0.95) and moderate matches (Tanimoto > 0.85), retaining the best-match name for strong matches. When calling categorate() with detail = 'research', use cache = TRUE, cache_dir = 'uafR_cache', and conservative throttling (e.g. throttle = 0.5) to space live PubChem/KEGG requests. Correctness is judged by validateCategorateResult() reporting no warnings and by SourceCoverage confirming each enabled source was queried, with transport errors distinguished from biological no-hits.

## Related tools

- **uafR** (Wraps the categorate() workflow that loads structures, applies detail = 'research' / cache / throttle parameters, and returns FMCS, BestChemMatch, and SourceCoverage tables) — https://github.com/castratton/uafR
- **ChemmineR** (read.SDFset() parses resolved structures into an SDFset; substructure descriptor computation (rings, subgroups, atom counts, charges) is performed with ChemmineR)
- **fmcsR** (fmcsBatch() compares each compound against every entry in the user-defined chemical library to produce Tanimoto structural-match frames)
- **PubChem** (Source of published structures and chemical data from which resolved structures and canonical SMILES are obtained)

## Examples

```
research_result <- categorate(compounds = query_compounds, chemical_library = library_data, input_format = "wide", detail = "research", cache = TRUE, throttle = 0.2)
```

## Evaluation signals

- validateCategorateResult() reports no validation warnings on the categorate result
- SourceCoverage confirms each enabled source (PubChem, LOTUS, KEGG, FEMA, FDA/SPL) was queried, with transport errors distinguished from biological no-hits
- FMCS table populates MW, Rings, Groups, Atoms, and NCharges columns usable by exactoThese() range filters (e.g. MW between 125 and 200)
- Strong-match frame contains only pairs with Tanimoto similarity > 0.95; moderate frame entries exceed 0.85
- BestChemMatch is populated only for compounds exceeding the strong-match threshold

## Limitations

- uafR depends on published data; MS2, MS3, and ion-mobility MS2 spectra are not supported until published for most chemicals in PubChem's database
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- Live database calls require throttling (e.g. throttle = 0.5); unresolved identities and transport errors must be inspected via SourceDiagnostics before downstream use

## Evidence

- [methods] generates substructure data via ChemmineR (rings, subgroups, atom counts, charges): [evidence span withheld — anchor status: not_found]
- [methods] Parse resolved structures with ChemmineR::read.SDFset(): [evidence span withheld — anchor status: not_found]
- [intro] strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85): "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [methods] fmcsR::fmcsBatch() compares each compound against the user-labeled library: [evidence span withheld — anchor status: not_found]
- [methods] categorate(detail = "research")` adds PubChem and KEGG profiles: "categorate(detail = "research")` adds PubChem and KEGG profiles"
- [intro] removes any chemicals that did not have a strong match across samples, were unable to be found in public databases: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
