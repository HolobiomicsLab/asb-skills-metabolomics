---
name: compound-database-annotation-enrichment
description: Use when you have a list of tentative compound names (or PubChem CIDs)
  from GC-MS peak annotation and need database-backed categorization, substructure
  descriptors, and strong/moderate structural matches to a user-defined chemical library.
when_to_use_negative:
- Your data are MS2, MS3, or ion mobility + MS2 spectra — uafR depends on published
  spectra in PubChem and does not yet apply to those data types
- You need LC-MS or non-Agilent/Q-TOF/Orbitrap-style processing — uafR is untested
  in those contexts
- The input is already a normalized feature/abundance table needing only extraction
  or standardization (use mzExacto()/standardifyIt() instead of categorate())
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3336
tools:
- name: uafR
  role: Provides categorate(), validateCategorateResult(), and exportCategorateWorkbook()
    for the enrichment workflow
  repo: github.com/castratton/uafR
- name: PubChem
  role: Source of categorical flags, chemical profiles, properties, and published
    structures for every tentative compound
- name: KEGG
  role: Database subset used to assign categorical flags and KEGGPathways in research
    detail mode
- name: LOTUS
  role: Natural-product database subset for categorical annotation
- name: ChemmineR
  role: read.SDFset() parsing of resolved structures and computation of ring, subgroup,
    atom-count, and charge descriptors
- name: fmcsR
  role: fmcsBatch() maximum common substructure comparison producing Tanimoto similarity
    to the user library
- name: R
  role: Runtime environment; uafR requires R 4.2 or newer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:485beadcc535d282a649ac674d90366aff65000fc1931dc7e74966a25283e31b
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
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# compound-database-annotation-enrichment

## Summary

Enrich a list of tentative GC-MS compound identifications with categorical, structural, and chemical-identity information from public databases (PubChem, KEGG, LOTUS, FEMA, MeSH, FDA/SPL) and flag structural matches against a user-labeled chemical library using fmcsR Tanimoto comparisons.

## When to use

Use when you have a list of tentative compound names (or PubChem CIDs) from GC-MS peak annotation and need database-backed categorization, substructure descriptors, and strong/moderate structural matches to a user-defined chemical library.

## When NOT to use

- Your data are MS2, MS3, or ion mobility + MS2 spectra — uafR depends on published spectra in PubChem and does not yet apply to those data types
- You need LC-MS or non-Agilent/Q-TOF/Orbitrap-style processing — uafR is untested in those contexts
- The input is already a normalized feature/abundance table needing only extraction or standardization (use mzExacto()/standardifyIt() instead of categorate())

## Inputs

- Character vector of tentative compound names or PubChem CIDs (e.g. c("Linalool", "Methyl Salicylate", "Limonene"))
- User-labeled chemical library as a wide data.frame mapping compounds to category columns (GroupA, GroupB, ...)
- Optional: categorate result from the prior run when using exactoThese() for downstream selection

## Outputs

- categorate result object with eight standard tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch
- detail = "research" additions: PubChemProperties, ChemicalTraits, ChemicalTraitOntology, ChemicalTraitMatrix, ChemicalMeasurements, ChemicalHazards, KEGGPathways, SourceCoverage
- Strong-match frame (Tanimoto > 0.95) and moderate-match frame (Tanimoto > 0.85) with best-match names for strong matches
- Validation audit from validateCategorateResult() (Summary, TableQuality, SourceDiagnostics, Issues)
- Exportable CSV/Excel bundle via exportCategorateWorkbook()

## How to apply

Run uafR::categorate() on the query compound list and a user-labeled chemical library (data.frame of category columns, input_format = "wide"), with detail = "research" to append normalized PubChem and KEGG tables to the eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch). Use cache = TRUE with a cache_dir and a conservative throttle (0.5 as in the source workflow; 0.2 shown in the README) to space live PubChem/KEGG requests. Parse resolved structures with ChemmineR::read.SDFset() and compute substructure descriptors (rings, subgroups, atom counts, charges), then run fmcsR::fmcsBatch() against every library entry. Split batch results into a strong-match frame (Tanimoto > 0.95) and a moderate-match frame (Tanimoto > 0.85), keeping the best-match name only for strong matches. Validate with validateCategorateResult(), inspect SourceCoverage (distinguishing transport errors from biological no-hits), unresolved identities, and the DataDictionary before saving.

## Related tools

- **uafR** (Provides categorate(), validateCategorateResult(), and exportCategorateWorkbook() for the enrichment workflow) — github.com/castratton/uafR
- **PubChem** (Source of categorical flags, chemical profiles, properties, and published structures for every tentative compound)
- **KEGG** (Database subset used to assign categorical flags and KEGGPathways in research detail mode)
- **LOTUS** (Natural-product database subset for categorical annotation)
- **ChemmineR** (read.SDFset() parsing of resolved structures and computation of ring, subgroup, atom-count, and charge descriptors)
- **fmcsR** (fmcsBatch() maximum common substructure comparison producing Tanimoto similarity to the user library)
- **R** (Runtime environment; uafR requires R 4.2 or newer)

## Examples

```
research_result <- categorate(compounds = query_compounds, chemical_library = library_data, input_format = "wide", detail = "research", cache = TRUE, throttle = 0.2); audit <- validateCategorateResult(research_result)
```

## Evaluation signals

- validateCategorateResult() reports no validation warnings
- SourceCoverage confirms every enabled source (PubChem, LOTUS, FEMA, KEGG, MeSH, FDA/SPL) was queried, with transport errors distinguished from biological no-hits
- Strong-match frame contains only Tanimoto > 0.95 hits; moderate frame only > 0.85
- BestChemMatch provides a best-match name for strong matches and none for weaker ones
- All eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) are present in the result; exportCategorateWorkbook() manifest lists the expected files

## Limitations

- Live web services require throttling (0.5 in the source workflow, 0.2 in the README) or queries may fail; unresolved identities must be inspected in the validated result
- Manually validate that identified compound areas make chemical and/or biological sense in the study system
- Depends on published data; does not apply to MS2, MS3, or ion mobility + MS2 data
- Excel export requires openxlsx or writexl installed

## Evidence

- [methods] categorate(detail = "research")` adds PubChem and KEGG profiles: "categorate(detail = "research")` adds PubChem and KEGG profiles"
- [intro] strong/moderate Tanimoto thresholds: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [methods] decontaminate filter: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [readme] validate and export research result: [evidence span withheld — anchor status: external_doc]
- [readme] standard-mode eight tables: "Standard mode returns eight tables:

reactives
LOTUS
KEGG
FEMA
FDA_SPL
FMCS
FunctionalGroups
BestChemMatch"
- [intro] Tanimoto against user categories rationale: "the pharmaceutical industry uses the Tanimoto similarity metric to discover compounds that will bind known ligands or share biological activity with known drugs"
