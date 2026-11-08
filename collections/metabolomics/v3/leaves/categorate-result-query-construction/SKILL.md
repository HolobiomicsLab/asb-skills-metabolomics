---
name: categorate-result-query-construction
description: 'Use when you have a categorate() result (eight standard tables: reactives,
  LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, ChemMatch) and need to derive
  a filtered chemical query list by database membership, FMCS properties, or chemical-library
  group.'
when_to_use_negative:
- Input is a raw GC-MS hit table (Component.RT, Component.Area, Base.Peak.MZ, File.Name,
  Compound.Name, Match.Factor columns) that has not yet been annotated with categorate()
  — use spreadOut()/mzExacto() instead.
- Selection should be driven by library match score rather than chemical annotation
  — derive the query directly from Match.Factor thresholds (e.g. Match.Factor > 80)
  on the raw Unknowns Analysis output.
- Data are MS2, MS3, or ion mobility + MS2 spectra, which uafR does not yet support
  because published spectra are unavailable in PubChem.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: uafR
  role: Provides exactoThese() for criteria-based subsetting of categorate() results
    and categorate() to generate the annotated input
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment in which the exactoThese() subsetting workflow is executed
- name: devtools
  role: Used to install uafR from GitHub before running the workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:1aeb790f1c2fdffae0bf0751e48c7e95128e5f10b53c6bc5ed34f1e69aff0a9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - LOTUS
  - R
  - RStudio
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# categorate-result-query-construction

## Summary

Constructs user-defined subsetting queries against a categorate() output using uafR's exactoThese(), selecting chemicals that meet database-, structure-, or library-group criteria. This is the bridge step between chemical annotation and re-extraction of targeted compounds from GC-MS peak tables.

## When to use

Use when you have a categorate() result (eight standard tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) and need to derive a filtered chemical query list by database membership, FMCS properties, or chemical-library group.

## When NOT to use

- Input is a raw GC-MS hit table (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor columns) that has not yet been annotated with categorate() — use spreadOut()/mzExacto() instead.
- Selection should be driven by library match score rather than chemical annotation — derive the query directly from Match.Factor thresholds (e.g. Match.Factor > 80) on the raw Unknowns Analysis output.
- Data are MS2, MS3, or ion mobility + MS2 spectra, which uafR does not yet support because published spectra are unavailable in PubChem.

## Inputs

- categorate() output (list of eight tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch; optionally enriched with detail = "research" PubChem/KEGG tables)
- User-specified subsetting criteria: subsetBy axis ("Database", "FMCS", or "Library"), subsetArgs (database names, FMCS property, or library group), optional subsetArgs2 comparison operator and subset_input value(s)

## Outputs

- Filtered data frame / chemical query list containing exactly the chemicals whose categorate fields satisfy the stated criteria
- Downstream input for mzExacto() to extract aggregated component areas across GC-MS samples for the selected compounds

## How to apply

Take the data frame or list produced by categorate() — either standard mode (eight tables) or detail = "research" mode with appended PubChem/KEGG tables — and call exactoThese() with a subsetBy axis: "Database" (subsetArgs from "All", "reactives", "LOTUS", "KEGG", "FEMA", "FDA_SPL", or a vector of several), "FMCS" (subsetArgs = "MW", "Rings", "Groups", "Atoms", or "NCharges" with subsetArgs2 = "Greater Than"/"Less Than"/"Between" and a numeric subset_input, e.g. MW between 125 and 200), or "Library" (subsetArgs = a group column name such as "GroupB"). The rationale is to let the user fluidly define a chemically meaningful subset — e.g. compounds with KEGG or LOTUS entries, or molecular weight ranges — after tentative library matching, rather than committing to a fixed compound list upstream. Feed the returned compound names back into mzExacto() with the spreadOut()-prepared GC-MS peak table to extract component areas for only the selected chemicals. Verify the returned data frame is non-empty and that every retained chemical's subset field matches the user-specified criteria (e.g. Database values restricted to LOTUS/KEGG).

## Related tools

- **uafR** (Provides exactoThese() for criteria-based subsetting of categorate() results and categorate() to generate the annotated input) — https://github.com/castratton/uafR
- **R** (Runtime environment in which the exactoThese() subsetting workflow is executed)
- **devtools** (Used to install uafR from GitHub before running the workflow)

## Examples

```
these_chems = exactoThese(query_categorated, subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG"))
```

## Evaluation signals

- The exactoThese() output is non-empty and contains only chemicals whose chosen information set (e.g. Database field) matches the supplied criteria — e.g. only LOTUS/KEGG-listed compounds when subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG").
- Applying a Between/FMCS MW query (subset_input = c(125, 200)) yields compounds whose molecular weights all fall within the stated range.
- The filtered list can be round-tripped through mzExacto(spread, these_chems) to produce a data frame with compound identities, optimal retention time, exact mass, best match factor, and aggregated component area.
- Exported result (e.g. exactoThese_filtered_chemicals.csv) row set equals the count of chemicals in the categorate result satisfying the criterion — no false positives from other databases.
- Validation of any research-enriched categorate result passes validateCategorateResult() before subsetting.

## Limitations

- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data, so the subsetting workflow is validated only for GC-MS peak tables.
- Algorithms do not apply to MS2, MS3, or ion mobility + MS2 data until those spectra are published in PubChem's database.
- Researchers should still validate that the compound areas identified downstream make chemical and/or biological sense in their study system.
- Subsetting by "Database" assumes the chemical was successfully found in public databases and assigned a canonical SMILES; chemicals failing decontaminate() are absent.

## Evidence

- [readme] exactoThese(query_categorated, subsetBy = "Database", subsetArgs = c("reactives", "FEMA")): [evidence span withheld — anchor status: external_doc]
- [readme] exactoThese FMCS MW Between query: [evidence span withheld — anchor status: external_doc]
- [other] exactoThese subsets categorate output per user criteria: [evidence span withheld — anchor status: not_found]
- [methods] subsetBy = Database with LOTUS and KEGG documented in methods: "subsetBy = "Database",
  subsetArgs = c("LOTUS", "KEGG")"
- [readme] Select compounds using database, structure, or library-group evidence: "Group and annotate compound names with `categorate()`, then select compounds
   for downstream extraction with `exactoThese()`."
