---
name: pubchem-kegg-profile-enrichment
description: Use when a GC-MS hit table (Component.RT, Component.Area, Base.Peak.MZ,
  File.Name, Compound.Name, Match.Factor) has yielded compound names and you need
  PubChem/KEGG traits, pathways, and source-coverage diagnostics for every published
  chemical.
when_to_use_negative:
- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's published-data-dependent
  algorithms do not yet apply to those modes
- The chemicals of interest lack PubChem entries or canonical SMILES, so public-database
  enrichment cannot resolve them
- Input is already an enriched/annotated feature table and you only need peak-area
  extraction (use spreadOut()/mzExacto()/standardifyIt() instead)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2258
tools:
- name: uafR
  role: Provides categorate(detail = "research"), validateCategorateResult(), and
    exportCategorateWorkbook() for PubChem/KEGG enrichment
  repo: github.com/castratton/uafR
- name: PubChem
  role: Open-source chemical database queried for normalized properties, traits, hazards,
    and occurrences per compound
  repo: pubchem.gov
- name: KEGG
  role: Source of KEGGPathways profiles appended in research mode and of exactoThese()
    Database subsets
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:350d2d9d4d2ceecea72b22bdaaa4517a94959a67f271113c90b717eb118f2ae4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemmineR
  - KEGG
  - LOTUS
  - PubChem
  - R
  - fmcsR
  - uafR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pubchem-kegg-profile-enrichment

## Summary

Enrich tentative GC-MS compound identifications with normalized PubChem and KEGG profiles via uafR's categorate(detail = "research") workflow, then validate and export the result before downstream use. The enrichment attaches chemical traits, pathways, hazards, and source coverage while preserving the eight standard categorate tables.

## When to use

Use when a GC-MS hit table (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) has yielded compound names and you need PubChem/KEGG traits, pathways, and source-coverage diagnostics for every published chemical.

## When NOT to use

- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's published-data-dependent algorithms do not yet apply to those modes
- The chemicals of interest lack PubChem entries or canonical SMILES, so public-database enrichment cannot resolve them
- Input is already an enriched/annotated feature table and you only need peak-area extraction (use spreadOut()/mzExacto()/standardifyIt() instead)

## Inputs

- Vector of compound names (or PubChem CIDs) from a GC-MS hit table
- User-labeled chemical library data.frame (e.g., uafR example data 'library_data', input_format = 'wide')
- R >= 4.2 session with uafR installed via remotes::install_github('castrattonDSU/uafR') and BiocManager-installed ChemmineR/fmcsR

## Outputs

- Categorate result object with eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) plus research-mode tables (PubChemProperties, KEGGPathways, ChemicalTraits, ChemicalTraitMatrix, ChemicalHazards, SourceCoverage)
- Validation audit from validateCategorateResult() (Summary, TableQuality, SourceDiagnostics, Issues)
- Exported CSV/Excel workbook bundle with manifest from exportCategorateWorkbook()

## How to apply

Run categorate() with detail = "research", input_format = "wide", a user-labeled chem_library, and conservative live-service settings such as cache = TRUE and throttle = 0.2–0.5 to space requests to PubChem/KEGG. Research mode appends normalized PubChem and KEGG tables (PubChemProperties, ChemicalTraits, ChemicalTraitOntology, ChemicalTraitMatrix, ChemicalMeasurements, ChemicalHazards, ChemicalOccurrences, KEGGPathways, SourceCoverage) while preserving the eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch). Validate the returned object with validateCategorateResult(), inspecting SourceCoverage to confirm each enabled source was queried and distinguishing transport errors from biological no-hits. Select the compounds to carry forward with exactoThese() (e.g., subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG")) and export a curated bundle with exportCategorateWorkbook(). The rationale is to ground each tentative molecule in 'the most advanced open-source chemical dataset' so downstream identification uses published molecular understanding rather than library match alone.

## Related tools

- **uafR** (Provides categorate(detail = "research"), validateCategorateResult(), and exportCategorateWorkbook() for PubChem/KEGG enrichment) — github.com/castratton/uafR
- **PubChem** (Open-source chemical database queried for normalized properties, traits, hazards, and occurrences per compound) — pubchem.gov
- **KEGG** (Source of KEGGPathways profiles appended in research mode and of exactoThese() Database subsets)

## Examples

```
research_result <- categorate(compounds = query_compounds, chemical_library = library_data, input_format = "wide", detail = "research", cache = TRUE, throttle = 0.2); audit <- validateCategorateResult(research_result)
```

## Evaluation signals

- validateCategorateResult() returns no validation warnings and audit$Issues is empty
- SourceCoverage table confirms every enabled source was queried, with transport errors distinguished from biological no-hits
- The eight standard categorate tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) are present unchanged alongside the appended research tables
- exportCategorateWorkbook() returns a manifest listing every written CSV file in the output path
- BestChemMatch contains strong (Tanimoto > 0.95) and moderate (> 0.85) structural-match frames against the user-labeled library

## Limitations

- Researchers should still validate that the compound areas and enriched annotations make chemical and/or biological sense in their study system
- Live-service calls require throttling (conservative request spacing, e.g. 0.2–0.5 s) to avoid provider transport errors; unresolved identities should be inspected
- uafR has not yet been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS data

## Evidence

- [methods] Research-mode enrichment appends normalized database tables: "categorate(detail = "research")` adds PubChem and KEGG profiles"
- [intro] Package communicates with public chemistry utilities for enrichment: "communicates with public chemistry utilities (including PubChem and the National Cancer Institute) to sort and process the aggregated set of all tentatively identified molecules"
- [readme] README documents research-mode table appending and preservation of standard tables: [evidence span withheld — anchor status: external_doc]
- [readme] Validation step before downstream analysis: [evidence span withheld — anchor status: external_doc]
- [results] PubChem grounding rationale: "This package harnesses the most advanced open-source chemical dataset and makes it accessible to anyone with basic experience working in R."
