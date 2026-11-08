---
name: missing-value-handling-in-harmonized-metabolite-tables
description: Use when assembling an InChIKey-keyed attribute table from CTS queries
  and some attributes come back empty — decide whether to record blanks or impute,
  and validate the table schema.
when_to_use_negative:
- When the table is keyed on metabolite names or a systematic identifier other than
  InChIKey — the CTS query flow described here is built specifically on InChIKey keys
- When an attribute can be filled deterministically from an authoritative local reference
  (e.g. a known CAS number) — this skill only records what CTS returns and leaves
  the rest blank, deferring fills to the Step 3 curation process
- When imputation of identifiers is explicitly desired (e.g. for a downstream tool
  requiring no nulls) — this skill's policy is explicitly to leave cells blank rather
  than impute
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Python
  role: Python 3.10.6 runtime used for REST/API calls to CTS and for assembling attribute_table.csv
    keyed on InChIKey
- name: Chemical Translator Service (CTS)
  role: External REST service queried with each InChIKey to obtain MW, formula, and
    the non-systematic identifiers (CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS);
    its empty responses are the missing values this skill handles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:b6a3efae8d183ad2aa59aa1f94ae7e6b1620e1a47064710b0f79a1ba5cf6b119
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translator Service (CTS)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# missing-value-handling-in-harmonized-metabolite-tables

## Summary

Handles blanks in the MMS Step 2 attribute table by leaving cells empty (never imputing) when the Chemical Translator Service (CTS) returns no value for an InChIKey's MW, formula, or non-systematic identifiers (CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS).

## When to use

Use when assembling an InChIKey-keyed attribute table from CTS queries and some attributes come back empty — decide whether to record blanks or impute, and validate the table schema.

## When NOT to use

- When the table is keyed on metabolite names or a systematic identifier other than InChIKey — the CTS query flow described here is built specifically on InChIKey keys
- When an attribute can be filled deterministically from an authoritative local reference (e.g. a known CAS number) — this skill only records what CTS returns and leaves the rest blank, deferring fills to the Step 3 curation process
- When imputation of identifiers is explicitly desired (e.g. for a downstream tool requiring no nulls) — this skill's policy is explicitly to leave cells blank rather than impute

## Inputs

- Merged dataset from MMS Step 1 with an InChIKey column
- Unique InChIKey list used as CTS query keys

## Outputs

- attribute_table.csv — one row per InChIKey with eight CTS-sourced attribute columns (MW, formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS), blanks where CTS returned nothing

## How to apply

After MMS Step 1 produces the merged dataset, extract the unique InChIKey column as query keys and, using Python 3.10.6 with REST/API connectivity, submit each key to the Chemical Translator Service (CTS) to retrieve molecular weight (MW), molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number. Assemble the retrieved values into one row per InChIKey in attribute_table.csv, keying each row on the InChIKey. Where CTS returns no value for an attribute — batch conversion can fail intermittently — leave the cell blank rather than imputing, because fabricated identifiers would corrupt downstream merging and curation. Blanks are not terminal: MMS Step 3's three-step curation process is the designated stage for rectifying missing attributes. Validation is schema-level: confirm the file exists, contains one row per input InChIKey, and includes all eight CTS-sourced attribute columns with blanks only where the service returned nothing.

## Related tools

- **Python** (Python 3.10.6 runtime used for REST/API calls to CTS and for assembling attribute_table.csv keyed on InChIKey)
- **Chemical Translator Service (CTS)** (External REST service queried with each InChIKey to obtain MW, formula, and the non-systematic identifiers (CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS); its empty responses are the missing values this skill handles)

## Evaluation signals

- attribute_table.csv exists after the run
- Row count equals the number of unique input InChIKeys (one row per key)
- All eight CTS-sourced attribute columns are present: MW, formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS
- Blanks occur only where the service returned nothing — no fabricated or imputed identifier values anywhere in the table
- Each row's identifiers plausibly resolve to its InChIKey (spot-check a sample against the CTS responses)

## Limitations

- CTS batch conversion can fail intermittently, producing blanks that are not attributable to absent attributes
- Blank cells are only remediated later, in the optional Step 3 curation process (which also handles conjugated base/acid disparities and synonym checking), so a freshly built attribute table may contain unresolvable nulls

## Evidence

- [other] Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing.: [evidence span withheld — anchor status: not_found]
- [abstract] Step 2: Attributes' retrieval from the InChIKey, including descriptors of name and chemical properties.: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties"
- [abstract] a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information): "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
