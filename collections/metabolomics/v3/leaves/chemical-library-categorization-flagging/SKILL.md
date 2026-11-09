---
name: chemical-library-categorization-flagging
description: Use when you have a list of tentative compound names (or PubChem CIDs)
  from GC-MS peak picking and need categorical/functional annotation plus structural
  matches against labeled chemical categories.
when_to_use_negative:
- Your data are MS2, MS3, or ion-mobility + MS2 spectra — uafR depends on published
  spectra and does not yet apply to these (deferred until published in PubChem)
- You need raw peak deconvolution or library matching from vendor output — use Agilent
  Unknowns Analysis/mzExacto() first; categorate() operates on already tentatively
  identified compound names
- Compounds lack published PubChem entries or canonical SMILES — decontamination removes
  such chemicals before enrichment, so categorate() would return unresolved identities
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_2258
tools:
- name: uafR
  role: Provides categorate(), exactoThese(), validateCategorateResult(), and exportCategorateWorkbook()
    for the enrichment and match-flagging workflow
  repo: github.com/castratton/uafR
- name: ChemmineR
  role: Parses resolved structures via read.SDFset() and computes substructure descriptors
    (rings, subgroups, atom counts, charges)
- name: fmcsR
  role: fmcsBatch() compares each query compound against every user-library entry
    to produce Tanimoto structural-match frames
- name: PubChem
  role: Primary source for categorical flags, normalized properties, and published
    structures used in enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:a6781d4e563d5d01b8f3cca2708b7db54d49a8b47788b4bf967c640ae2bdb27f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemmineR
  - LOTUS
  - PubChem
  - R
  - fmcsR
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Chemical-library categorization and structural-match flagging

## Summary

Enriches every tentative GC-MS compound with categorical and structural data from PubChem, LOTUS, KEGG, FEMA, MeSH, and FDA/SPL, then flags strong and moderate structural matches against a user-defined chemical library via Tanimoto similarity.

## When to use

Use when you have a list of tentative compound names (or PubChem CIDs) from GC-MS peak picking and need categorical/functional annotation plus structural matches against labeled chemical categories.

## When NOT to use

- Your data are MS2, MS3, or ion-mobility + MS2 spectra — uafR depends on published spectra and does not yet apply to these (deferred until published in PubChem)
- You need raw peak deconvolution or library matching from vendor output — use Agilent Unknowns Analysis/mzExacto() first; categorate() operates on already tentatively identified compound names
- Compounds lack published PubChem entries or canonical SMILES — decontamination removes such chemicals before enrichment, so categorate() would return unresolved identities

## Inputs

- Character vector of tentative compound names or PubChem CIDs from a GC-MS hit table
- User-labeled chemical library (data.frame, wide format, e.g. columns GroupA/GroupB with compound-name members)
- Optional spreadOut()-prepared GC-MS peak table with Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor columns for downstream extraction

## Outputs

- categorate result object with eight standard tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch
- Research-grade tables: PubChemProperties, ChemicalTraits, ChemicalTraitOntology, ChemicalTraitMatrix, ChemicalMeasurements, ChemicalHazards, KEGGPathways
- Strong-match frame (Tanimoto > 0.95) and moderate-match frame (Tanimoto > 0.85) with best-match compound names for strong matches
- SourceCoverage table showing which sources were queried and why (transport error vs. biological no-hit)
- Validation audit from validateCategorateResult() (Summary, TableQuality, SourceDiagnostics, Issues)

## How to apply

Load the query compound list and the user-labeled chemical library (e.g. a wide-format data.frame of GroupA/GroupB members), then call categorate() with detail = "research", cache = TRUE, a cache_dir such as "uafR_cache", and a throttle (e.g. 0.2–0.5 s) to space requests to live services conservatively. The function queries PubChem, LOTUS, KEGG, FEMA, MeSH, and FDA/SPL to assign categorical flags, source coverage, and normalized traits, returning eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) plus research-grade additions such as PubChemProperties, ChemicalTraits, and SourceCoverage. Resolved structures are parsed with ChemmineR::read.SDFset() and substructure descriptors (rings, subgroups, atom counts, charges) are computed. fmcsR::fmcsBatch() then compares each compound against every user-library entry, splitting results into a strong-match frame (Tanimoto > 0.95) and a moderate-match frame (Tanimoto > 0.85), with best-match names attached to strong matches. Run validateCategorateResult() on the result, inspect SourceCoverage and unresolved identities, and optionally export with exportCategorateWorkbook() before passing selected compounds (via exactoThese()) to mzExacto() for peak-area aggregation.

## Related tools

- **uafR** (Provides categorate(), exactoThese(), validateCategorateResult(), and exportCategorateWorkbook() for the enrichment and match-flagging workflow) — github.com/castratton/uafR
- **ChemmineR** (Parses resolved structures via read.SDFset() and computes substructure descriptors (rings, subgroups, atom counts, charges))
- **fmcsR** (fmcsBatch() compares each query compound against every user-library entry to produce Tanimoto structural-match frames)
- **PubChem** (Primary source for categorical flags, normalized properties, and published structures used in enrichment)

## Examples

```
research_result <- categorate(compounds = query_compounds, chemical_library = library_data, input_format = "wide", detail = "research", cache = TRUE, throttle = 0.2); audit <- validateCategorateResult(research_result)
```

## Evaluation signals

- validateCategorateResult() reports no validation warnings on the returned research result
- SourceCoverage confirms each enabled source (LOTUS, KEGG, FEMA, FDA/SPL, PubChem) was queried, with transport errors distinguished from biological no-hits
- Strong-match frame contains only pairs with Tanimoto similarity > 0.95; moderate-match frame only > 0.85
- Every strong match carries a best-match compound name in the BestChemMatch table
- Result contains all eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) plus research additions

## Limitations

- Untested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) and LC-MS data
- Algorithms depend on published data, so MS2/MS3 and ion-mobility + MS2 workflows are not supported until those spectra appear in PubChem
- Live-service requests require throttling (e.g. 0.2–0.5 s spacing); network transport errors must be distinguished from genuine biological no-hits when interpreting SourceCoverage
- Researchers must still manually validate that flagged compound areas and matches make chemical/biological sense in their study system

## Evidence

- [methods] categorate(detail = "research")` adds PubChem and KEGG profiles: "categorate(detail = "research")` adds PubChem and KEGG profiles"
- [intro] whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [readme] Use `detail = "research"` to append normalized PubChem and KEGG tables while preserving the original eight standard tables: [evidence span withheld — anchor status: external_doc]
- [readme] Validate the result before downstream analysis:: [evidence span withheld — anchor status: external_doc]
