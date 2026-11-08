---
name: chemical-identity-subsetting
description: Use when you have a categorate() output from uafR and need to reduce
  a compound list to those matching explicit database, structural, or library-group
  criteria before downstream GC-MS extraction.
when_to_use_negative:
- Input is a raw GC-MS hit table that has not yet been annotated — run spreadOut()
  and categorate() first; exactoThese() operates on categorate output, not vendor
  peak lists
- You need tentative peak extraction by retention time and match factor from the raw
  table — use mzExacto() directly with a query chemical vector instead
- Data are MS2, MS3, or ion mobility + MS2, which uafR's published-spectrum-dependent
  algorithms do not yet support
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: uafR
  role: Provides exactoThese() and categorate(); the package whose Database/FMCS/Library
    annotation tables are subset by user-defined criteria
  repo: github.com/castratton/uafR
- name: R
  role: Runtime environment in which exactoThese() subsetting of the categorate result
    is executed
- name: devtools
  role: Used to install uafR from GitHub before running the subsetting workflow
  repo: castratton/uafR
- name: RStudio
  role: Environment from which uafR can be installed and the subsetting workflow run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:f459dc5864a888514f30f0a88d2122a1e60ff8f2ee73b6c4e30b06658a157986
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - R
  - RStudio
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chemical-identity-subsetting

## Summary

Subsets chemicals of interest from a uafR categorate() annotation result using exactoThese(), driven by user-specified criteria such as database membership (LOTUS, KEGG, FEMA, FDA_SPL, reactives), FMCS properties, or chemical-library group.

## When to use

Use when you have a categorate() output from uafR and need to reduce a compound list to those matching explicit database, structural, or library-group criteria before downstream GC-MS extraction.

## When NOT to use

- Input is a raw GC-MS hit table that has not yet been annotated — run spreadOut() and categorate() first; exactoThese() operates on categorate output, not vendor peak lists
- You need tentative peak extraction by retention time and match factor from the raw table — use mzExacto() directly with a query chemical vector instead
- Data are MS2, MS3, or ion mobility + MS2, which uafR's published-spectrum-dependent algorithms do not yet support

## Inputs

- categorate() result (data frame/list of eight annotation tables: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch)
- User-specified subsetting criteria: subsetBy ('Database', 'FMCS', or 'Library'), subsetArgs (database names or FMCS property), optional subsetArgs2 comparator and subset_input numeric threshold(s)

## Outputs

- Filtered data frame/list containing exactly the chemicals whose categorate fields satisfy the stated criteria
- Compound name vector suitable as the 'chemicals' argument to mzExacto() for component-area aggregation across GC-MS samples

## How to apply

Run categorate() first on query compound names with a chemical library (input_format = 'wide'), which returns eight standard tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch). Then call exactoThese() on that result: use subsetBy = 'Database' with subsetArgs naming one or more databases (e.g. c('LOTUS','KEGG') or 'All'), subsetBy = 'FMCS' with a property (MW, Rings, Groups, Atoms, NCharges) plus a comparator ('Greater Than', 'Less Than', 'Between') and subset_input threshold (e.g. MW between 125 and 200), or subsetBy = 'Library' with a group name like 'GroupB'. The rationale is to let the user fluidly interact with the experiment and focus interpretation on chemicals that are meaningful, rather than manually looking up each molecule. The selected compound names feed directly into mzExacto() on the spreadOut() peak table to aggregate component areas across GC-MS samples.

## Related tools

- **uafR** (Provides exactoThese() and categorate(); the package whose Database/FMCS/Library annotation tables are subset by user-defined criteria) — github.com/castratton/uafR
- **R** (Runtime environment in which exactoThese() subsetting of the categorate result is executed)
- **devtools** (Used to install uafR from GitHub before running the subsetting workflow) — castratton/uafR
- **RStudio** (Environment from which uafR can be installed and the subsetting workflow run)

## Examples

```
selected <- exactoThese(annotations, subsetBy = "Database", subsetArgs = c("LOTUS", "KEGG"))
```

## Evaluation signals

- Returned data frame is non-empty and contains only chemicals whose subset field matches the user-specified criteria (e.g. Database values exactly LOTUS or KEGG when subsetBy = 'Database', subsetArgs = c('LOTUS','KEGG'))
- For FMCS criteria, every retained compound satisfies the stated comparator and threshold (e.g. molecular weight between 125 and 200, Rings greater than 1)
- For subsetBy = 'Library', retained compounds belong to the named library group (e.g. GroupB) in the supplied chem_library
- Downstream check: feeding the selected names into mzExacto() on the spreadOut() table returns rows only for the selected chemicals
- Standard mode output retains the eight categorate tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch) as the subsetting source

## Limitations

- Only chemicals with strong cross-sample matches, PubChem entries, and canonical SMILES survive the upstream decontaminate step, so exactoThese() cannot recover compounds absent from the categorate output
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data, limiting the subsetting workflow's scope to supported GC-MS contexts
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [other] The exactoThese() function takes the output generated by categorate() and subsets it according to which set of information the user specifies and criteria they indicate the chemicals should meet.: [evidence span withheld — anchor status: not_found]
- [methods] exactoThese database subsetting examples: "subsetBy = "Database",
  subsetArgs = c("LOTUS", "KEGG")"
- [readme] Standard categorate tables listed in README: "reactives
LOTUS
KEGG
FEMA
FDA_SPL
FMCS
FunctionalGroups
BestChemMatch"
- [readme] FMCS property subsetting example from README: [evidence span withheld — anchor status: external_doc]
- [readme] exactoThese selected names feed mzExacto extraction: "Use the selected names in the GC–MS extraction workflow:

selected_data <- mzExacto(
  data_in = spread,
  chemicals = selected
)"
- [intro] Focus interpretation on meaningful chemicals rationale: "focusing the interpretation on specific chemicals or chemistries that are meaningful would require looking up each molecule for published information"
