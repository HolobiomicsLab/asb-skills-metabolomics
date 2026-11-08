---
name: database-label-filtering
description: Use when a categorate() result for GC-MS compounds must be narrowed to
  chemicals meeting user-defined database-membership (LOTUS, KEGG, FEMA, FDA_SPL,
  reactives), structural-property (MW, Rings, Groups, Atoms, NCharges), or library-group
  criteria before downstream extraction.
when_to_use_negative:
- Input is already a filtered compound list or a raw GC-MS hit table rather than a
  categorate() result — use spreadOut()/mzExacto() directly instead.
- The goal is to keep every tentative compound without database/structural/library
  criteria; exactoThese() subsetting is unnecessary and would drop compounds.
- Working with MS2, MS3, or ion mobility + MS2 data, where uafR's published-data-dependent
  filtering does not yet apply.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Provides categorate() and exactoThese(); exactoThese() subsets the categorate()
    output by database, FMCS property, or library-group criteria
  repo: https://github.com/castratton/uafR
- name: LOTUS
  role: Database label used as a subsetArgs value under subsetBy = 'Database' to keep
    chemicals with a LOTUS entry
- name: KEGG
  role: Database label used as a subsetArgs value under subsetBy = 'Database' to keep
    chemicals with a KEGG entry
- name: PubChem
  role: Enrichment source whose profiles (PubChemProperties, PubChem Fingerprint2D)
    populate categorate(detail = "research") output used for subsetting
- name: mzExacto()
  role: Consumes the exactoThese()-filtered compound list to extract those chemicals'
    areas across GC-MS samples
  repo: https://github.com/castratton/uafR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:8a40c57061bb30d5e8761f6ec09a54b3f0c9c3378063a957efa3699202397153
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - KEGG
  - LOTUS
  - PubChem
  - R
  - RStudio
  - devtools
  - mzExacto()
  - uafR
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# database-label-filtering

## Summary

Subsets chemicals of interest from the categorate() output using exactoThese() according to a user-specified information set (e.g. Database, FMCS, or Library) and matching criteria, enabling targeted downstream GC-MS extraction of the filtered compound set.

## When to use

Use when a categorate() result for GC-MS compounds must be narrowed to chemicals meeting user-defined database-membership (LOTUS, KEGG, FEMA, FDA_SPL, reactives), structural-property (MW, Rings, Groups, Atoms, NCharges), or library-group criteria before downstream extraction.

## When NOT to use

- Input is already a filtered compound list or a raw GC-MS hit table rather than a categorate() result — use spreadOut()/mzExacto() directly instead.
- The goal is to keep every tentative compound without database/structural/library criteria; exactoThese() subsetting is unnecessary and would drop compounds.
- Working with MS2, MS3, or ion mobility + MS2 data, where uafR's published-data-dependent filtering does not yet apply.

## Inputs

- categorate() output (a named list/data frame of annotation tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch; optionally enriched PubChem/KEGG tables when detail = "research")
- User-specified subsetting parameters: subsetBy ('Database', 'FMCS', or 'Library'), subsetArgs, optional subsetArgs2, and subset_input

## Outputs

- Filtered vector/data frame of chemicals satisfying the user-specified criteria, suitable as the 'chemicals' argument to mzExacto()
- Downstream mzExacto() extraction of those selected compounds across GC-MS samples (compound identities, optimal retention time, exact mass, best match factor, aggregated component area)

## How to apply

Take the data frame of named tables returned by categorate() (standard mode returns reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups and BestChemMatch tables) and call exactoThese() with the appropriate subsetBy argument: 'Database' with subsetArgs naming one or more databases (e.g. c("LOTUS","KEGG") or "reactives"), 'FMCS' with subsetArgs naming a property (MW, Rings, Groups, Atoms, NCharges) plus subsetArgs2 ("Greater Than", "Less Than", or "Between") and subset_input giving the numeric threshold or range, or 'Library' with subsetArgs naming a chemical-library group. The function selects chemicals whose values in the chosen information set satisfy the stated criteria and returns the filtered compound list, which is then fed directly to mzExacto() for aggregation of component areas across GC-MS samples. For research-grade runs built with categorate(detail = "research"), the same subsetting applies after enrichment with PubChem and KEGG profiles; validate the categorate result with validateCategorateResult() before subsetting if that detail level was used.

## Related tools

- **uafR** (Provides categorate() and exactoThese(); exactoThese() subsets the categorate() output by database, FMCS property, or library-group criteria) — https://github.com/castratton/uafR
- **LOTUS** (Database label used as a subsetArgs value under subsetBy = 'Database' to keep chemicals with a LOTUS entry)
- **KEGG** (Database label used as a subsetArgs value under subsetBy = 'Database' to keep chemicals with a KEGG entry)
- **PubChem** (Enrichment source whose profiles (PubChemProperties, PubChem Fingerprint2D) populate categorate(detail = "research") output used for subsetting)
- **mzExacto()** (Consumes the exactoThese()-filtered compound list to extract those chemicals' areas across GC-MS samples) — https://github.com/castratton/uafR

## Examples

```
these_chems <- exactoThese(query_categorated, subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG")); input_exacto <- mzExacto(input_spread, these_chems)
```

## Evaluation signals

- Output is non-empty and contains only chemicals whose chosen information-set field matches the user-specified criteria (e.g. Database values LOTUS or KEGG).
- For subsetBy = "FMCS" with subsetArgs2 = "Between" and subset_input = c(125, 200), every returned compound's MW falls within that range.
- For subsetBy = "Database" with multiple subsetArgs (e.g. c("reactives","FEMA")), the output equals the union of chemicals in each named database table.
- Feed the exactoThese() result into mzExacto(input_spread, these_chems) and confirm the extracted data frame covers exactly the selected compounds.
- When using categorate(detail = "research"), run validateCategorateResult() and check audit$Summary/audit$Issues before subsetting.

## Limitations

- Subsetting depends on chemical names matching entries in the referenced public databases; chemicals absent from PubChem or without canonical SMILES are removed by the upstream decontamination step and cannot be selected.
- categorate(detail = "research") requires network access to PubChem and KEGG; offline runs are limited to the standard eight tables.
- uafR has only been validated on GC-MS data from single-quadrupole instruments; it has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS data.

## Evidence

- [methods] exactoThese() subsets according to which set of information the user specifies and criteria they indicate: [evidence span withheld — anchor status: not_found]
- [methods] subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG"): "subsetBy = "Database",
  subsetArgs = c("LOTUS", "KEGG")"
- [methods] categorate(detail = "research") adds PubChem and KEGG profiles: "categorate(detail = "research")` adds PubChem and KEGG profiles"
- [readme] categorate() then select compounds for downstream extraction with exactoThese(): "Group and annotate compound names with `categorate()`, then select compounds
   for downstream extraction with `exactoThese()`."
- [readme] Standard mode returns eight tables including reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS: "Standard mode returns eight tables:

```text
reactives
LOTUS
KEGG
FEMA
FDA_SPL
FMCS
FunctionalGroups
BestChemMatch
```"
- [readme] mzExacto() returns compound identities, optimal retention time, exact mass, best match factor, and aggregated component area across samples: "`mzExacto()` returns compound identities, optimal retention time, exact mass,
best match factor, and aggregated component area across samples."
