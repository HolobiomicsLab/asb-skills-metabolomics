---
name: chemical-structure-substructure-descriptor-extraction
description: Use when tentative GC-MS compound names (e.g. from Unknowns Analysis
  output with a Match.Factor column) need substructure descriptors or Tanimoto structural
  matching against a user-defined chemical category library.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR depends on published spectra
  and its algorithms do not yet apply to these modes
- Compounds lack a PubChem entry or cannot have a canonical SMILES generated — the
  decontaminate step removes these, so no structure is available for descriptor extraction
- The goal is only GC-MS peak-area aggregation without structural interpretation —
  use spreadOut()/mzExacto()/standardifyIt() directly instead
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: uafR
  role: Provides categorate(), exactoThese(), and validateCategorateResult() that
    orchestrate descriptor extraction and FMCS-based library matching
  repo: https://github.com/castratton/uafR
- name: ChemmineR
  role: Parses resolved structures via read.SDFset() and computes rings, subgroups,
    atom counts, and charges; installed via BiocManager
- name: fmcsR
  role: Supplies fmcsBatch() for batch maximum-common-substructure Tanimoto comparison
    against the user-labeled library
- name: PubChem
  role: Source of published structures and Fingerprint2D used for Tanimoto summaries
    and canonical SMILES
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:ca2b4fa22107483bd3ed7655c3bc2bd8df940452dca64d4b41353a4611db1af0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# chemical-structure-substructure-descriptor-extraction

## Summary

Extracts ring, subgroup, atom-count, and charge substructure descriptors from resolved compound structures (via ChemmineR) and runs maximum common substructure (fmcsBatch) Tanimoto comparisons against a user-labeled chemical library, splitting matches into strong (>0.95) and moderate (>0.85) tiers. This is the structural-comparison core of uafR's categorate() cheminformatics workflow.

## When to use

Use when tentative GC-MS compound names (e.g. from Unknowns Analysis output with a Match.Factor column) need substructure descriptors or Tanimoto structural matching against a user-defined chemical category library.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR depends on published spectra and its algorithms do not yet apply to these modes
- Compounds lack a PubChem entry or cannot have a canonical SMILES generated — the decontaminate step removes these, so no structure is available for descriptor extraction
- The goal is only GC-MS peak-area aggregation without structural interpretation — use spreadOut()/mzExacto()/standardifyIt() directly instead

## Inputs

- Character vector of tentative compound names (e.g. Linalool, Methyl salicylate, Limonene, alpha-Pinene) or PubChem CIDs
- User-labeled chemical library as a wide-format data frame (e.g. chem_library with columns GroupA, GroupB)
- GC-MS hit table with required columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor, spread via spreadOut()

## Outputs

- categorate result object with eight standard tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch
- Research-mode additions: PubChemProperties, ChemicalTraits, ChemicalTraitOntology, ChemicalTraitMatrix, ChemicalTraitReport, ChemicalMeasurements, ChemicalMeasurementSummary, ChemicalHazards, ChemicalOccurrences, KEGGPathways, SourceCoverage
- Strong-match frame (Tanimoto > 0.95) and moderate-match frame (Tanimoto > 0.85) with best-match names for strong matches
- Validation audit from validateCategorateResult() (Summary, TableQuality, SourceDiagnostics, Issues) and export manifest from exportCategorateWorkbook()

## How to apply

Run categorate() on the query compound list with a user-labeled chem_library data frame (input_format = "wide") and detail = "research", cache = TRUE, cache_dir = "uafR_cache", and throttle = 0.5 to space requests to live services (PubChem, LOTUS, FEMA, KEGG, MeSH, FDA/SPL). Parse resolved structures with ChemmineR::read.SDFset() and compute substructure descriptors: rings, subgroups, atom counts, and charges. Then run fmcsR::fmcsBatch() to compare every compound against every entry in the user-labeled library, using the Tanimoto similarity metric the paper adopts from pharmaceutical ligand/drug similarity work. Split the batch results into a strong-match frame (Tanimoto > 0.95) and a moderate-match frame (Tanimoto > 0.85), retaining best-match names only for strong matches. Validate with validateCategorateResult() and inspect SourceCoverage to distinguish transport errors from biological no-hits before saving the result.

## Related tools

- **uafR** (Provides categorate(), exactoThese(), and validateCategorateResult() that orchestrate descriptor extraction and FMCS-based library matching) — https://github.com/castratton/uafR
- **ChemmineR** (Parses resolved structures via read.SDFset() and computes rings, subgroups, atom counts, and charges; installed via BiocManager)
- **fmcsR** (Supplies fmcsBatch() for batch maximum-common-substructure Tanimoto comparison against the user-labeled library)
- **PubChem** (Source of published structures and Fingerprint2D used for Tanimoto summaries and canonical SMILES)

## Examples

```
research_result <- categorate(query_compounds, library_data, input_format = "wide", detail = "research", cache = TRUE, throttle = 0.2); audit <- validateCategorateResult(research_result)
```

## Evaluation signals

- validateCategorateResult() reports no validation warnings on the categorate result
- SourceCoverage confirms each enabled source was queried, with transport errors distinguished from biological no-hits
- Strong-match frame contains only Tanimoto > 0.95 hits and moderate-match frame only Tanimoto > 0.85 hits
- BestChemMatch names are populated only for strong matches; moderate matches carry no best-match name
- Research-mode result contains all expected tables (ChemicalTraits, ChemicalTraitMatrix, FMCS, FunctionalGroups, SourceCoverage) with standard eight tables preserved

## Limitations

- Live provider calls need throttling (e.g. throttle = 0.5 conservative spacing) and caching (cache = TRUE, cache_dir = "uafR_cache") to be reliable
- Transport errors can be mistaken for biological no-hits unless SourceCoverage diagnostics are checked
- Researchers must still manually validate that compound areas and structural matches make chemical and/or biological sense in their study system
- Research-mode fields may gain provider-specific changes before the next stable release (functions labeled experimental)

## Evidence

- [intro] our package utilizes Tanimoto similarity, a commonly used and rigorously tested metric for physicochemical comparisons: "our package utilizes Tanimoto similarity, a commonly used and rigorously tested metric for physicochemical comparisons"
- [intro] Tanimoto structural-match thresholds: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [readme] FMCS substructure filtering in README: [evidence span withheld — anchor status: external_doc]
- [readme] Research-mode enrichment in README: [evidence span withheld — anchor status: external_doc]
