---
name: cross-database-identifier-harmonization
description: Reference for annotating an InChIKey-keyed metabolite table with cross-database
  identifiers and chemical properties when datasets from different studies must be
  merged and compared on shared keys.
when_to_use_negative:
- When the input table already contains populated CheBI/HMDB/KEGG/LipidMaps/DrugBank/CAS
  columns — re-querying CTS would duplicate Step 2 work instead of advancing to Step
  3 curation (synonym/missing-attribute checks)
- When compounds cannot be converted to InChIKeys (e.g. only metabolite names available
  with no resolvable identifier) — MMS Step 1 translation/merging by InChIKey must
  be completed first
- For primary identifier assignment in a database you control — this skill is an attribute-retrieval
  lookup over existing InChIKeys, not an ID-minting service
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Chemical Translator Service (CTS)
  role: External web service queried with each InChIKey to return MW, molecular formula,
    and the CheBI, HMDB, KEGG, LipidMaps, DrugBank, and CAS identifiers
- name: Python
  role: Driver language (v3.10.6) used to submit InChIKeys to CTS over REST/API and
    assemble attribute_table.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:95598f00d23462e0a823e73be9a0370d1021dd91582bdca89c5ab0a4bca23150
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translator Service (CTS)
  - Python
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-database-identifier-harmonization

## Summary

Retrieves, for each input InChIKey, the molecular weight, molecular formula, and non-systematic database identifiers (CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) via the Chemical Translator Service (CTS), as used in Step 2 of the Metabolites Merging Strategy (MMS).

## When to use

Reference for annotating an InChIKey-keyed metabolite table with cross-database identifiers and chemical properties when datasets from different studies must be merged and compared on shared keys.

## When NOT to use

- When the input table already contains populated CheBI/HMDB/KEGG/LipidMaps/DrugBank/CAS columns — re-querying CTS would duplicate Step 2 work instead of advancing to Step 3 curation (synonym/missing-attribute checks)
- When compounds cannot be converted to InChIKeys (e.g. only metabolite names available with no resolvable identifier) — MMS Step 1 translation/merging by InChIKey must be completed first
- For primary identifier assignment in a database you control — this skill is an attribute-retrieval lookup over existing InChIKeys, not an ID-minting service

## Inputs

- merged InChIKey-keyed metabolite table from MMS Step 1 (e.g. attribute source CSV with a unique InChIKey column)
- list of unique InChIKeys extracted as CTS query keys

## Outputs

- attribute_table.csv — one row per input InChIKey with eight CTS-sourced attribute columns: MW, molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number

## How to apply

Take the merged dataset produced by MMS Step 1 (a CSV keyed on InChIKeys) and extract the unique InChIKey column as the query keys. Query the Chemical Translator Service (CTS) from Python 3.10.6 with REST/API connectivity, requesting for each InChIKey: molecular weight (MW), molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number. Assemble the retrieved values into one row per InChIKey and write attribute_table.csv, keyed on InChIKey. Leave attribute cells blank rather than imputing when the CTS returns nothing, since batch conversion can fail intermittently. The rationale is that systematic InChIKeys are the only stable join across metabolite names from PubChem and RefMet, and that non-systematic identifiers are needed to link entries to ontology and pathway resources downstream in MMS Step 3 curation.

## Related tools

- **Chemical Translator Service (CTS)** (External web service queried with each InChIKey to return MW, molecular formula, and the CheBI, HMDB, KEGG, LipidMaps, DrugBank, and CAS identifiers)
- **Python** (Driver language (v3.10.6) used to submit InChIKeys to CTS over REST/API and assemble attribute_table.csv)

## Examples

```
python cts_lookup.py --input attribute_table.csv --key InChIKey --attributes MW,formula,CheBI,HMDB,KEGG,LipidMaps,DrugBank,CAS
```

## Evaluation signals

- attribute_table.csv exists and is non-empty
- attribute_table.csv contains exactly one row per input InChIKey (no duplicates, no dropped keys)
- All eight CTS-sourced attribute columns are present: MW, molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number
- Blank cells appear only where CTS returned nothing — no imputed or placeholder values
- Retrieved molecular weight and formula are consistent with the InChIKey-derived structure (spot-check a subset against known values)

## Limitations

- CTS batch conversion can fail intermittently, so some attribute cells will be blank; these must be left empty rather than imputed and may need follow-up curation in MMS Step 3
- Coverage depends on external database availability — some InChIKeys will not have entries in CheBI, HMDB, KEGG, LipidMaps, DrugBank, or CAS
- Requires Python 3.10.6 environment with REST/API connectivity to CTS; offline execution is not possible

## Evidence

- [methods] Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties (molecular weight and: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties (molecular weight and"
- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [methods] Load the merged dataset produced by MMS Step 1 and extract the unique InChIKey column as the query keys.: [evidence span withheld — anchor status: not_found]
- [methods] submit each InChIKey to the Chemical Translator Service (CTS) to retrieve molecular weight (MW), molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number: [evidence span withheld — anchor status: not_found]
- [methods] Assemble the retrieved values into one row per InChIKey and write the table to attribute_table.csv, keying each row on the InChIKey.: [evidence span withheld — anchor status: not_found]
- [methods] Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing.: [evidence span withheld — anchor status: not_found]
- [methods] Using Python 3.10.6 with REST/API connectivity: [evidence span withheld — anchor status: not_found]
- [methods] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration"
