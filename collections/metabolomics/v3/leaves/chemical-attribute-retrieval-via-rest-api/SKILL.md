---
name: chemical-attribute-retrieval-via-rest-api
description: Use when a merged metabolite dataset already carries InChIKeys (output
  of MMS Step 1) and you need per-metabolite chemical properties and cross-database
  identifiers to enable interstudy comparability.
when_to_use_negative:
- Input metabolites have no InChIKeys yet — run MMS Step 1 (translation and merging
  of datasets by employing InChIKeys) first
- You need standardized name descriptors (title name from PubChem, RefMet name from
  Metabolomics Workbench) or synonym/duplicate curation — those are separate Step
  2 descriptors and Step 3 curation, not CTS attribute retrieval
- Only a single database identifier (e.g., HMDB alone) is needed and direct database
  lookup is available without a translation service
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3172
tools:
- name: Chemical Translator Service (CTS)
  role: REST service queried with each InChIKey to return MW, formula, and non-systematic
    identifiers (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS)
- name: Python
  role: Scripting language (Python 3.10.6) used to submit InChIKeys to CTS via REST/API
    and assemble attribute_table.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:da3aef38483dfd84ed70ce13509369f60f5383757b9d38bb750550aa2eb8e4f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translator Service (CTS)
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-attribute-retrieval-via-rest-api

## Summary

Step 2 of the Metabolites Merging Strategy (MMS): queries the Chemical Translator Service (CTS) via REST with each InChIKey to retrieve molecular weight, molecular formula, and non-systematic identifiers (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) into a single attribute table.

## When to use

Use when a merged metabolite dataset already carries InChIKeys (output of MMS Step 1) and you need per-metabolite chemical properties and cross-database identifiers to enable interstudy comparability.

## When NOT to use

- Input metabolites have no InChIKeys yet — run MMS Step 1 (translation and merging of datasets by employing InChIKeys) first
- You need standardized name descriptors (title name from PubChem, RefMet name from Metabolomics Workbench) or synonym/duplicate curation — those are separate Step 2 descriptors and Step 3 curation, not CTS attribute retrieval
- Only a single database identifier (e.g., HMDB alone) is needed and direct database lookup is available without a translation service

## Inputs

- Merged dataset from MMS Step 1 with a unique InChIKey column
- Chemical Translator Service (CTS) REST endpoint (Python 3.10.6)

## Outputs

- attribute_table.csv with one row per InChIKey and eight CTS-sourced columns: MW, molecular formula, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number (blank where the service returned nothing)

## How to apply

Load the merged dataset from MMS Step 1 and extract the unique InChIKey column as query keys. Using Python 3.10.6 with REST/API connectivity, submit each InChIKey to the Chemical Translator Service (CTS) to retrieve molecular weight (MW), molecular formula, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number. Assemble the retrieved values into one row per InChIKey, keyed on the InChIKey, and write attribute_table.csv. Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing — missing attributes are handled later in MMS Step 3's three-step curation process. Note that CTS does not supply the name descriptors (title name from PubChem, RefMet name from Metabolomics Workbench), which are retrieved separately in Step 2.

## Related tools

- **Chemical Translator Service (CTS)** (REST service queried with each InChIKey to return MW, formula, and non-systematic identifiers (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS))
- **Python** (Scripting language (Python 3.10.6) used to submit InChIKeys to CTS via REST/API and assemble attribute_table.csv)

## Examples

```
python step2_cts_attributes.py --input merged_step1.csv --inchikey-col InChIKey --output attribute_table.csv
```

## Evaluation signals

- attribute_table.csv exists after the run
- The table contains one row per input InChIKey (no dropped or duplicated keys)
- All eight CTS-sourced attribute columns (MW, formula, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) are present
- Blank cells occur only where the CTS returned nothing — no imputed or fabricated values
- Each row is keyed on its InChIKey, allowing lossless join back to the Step 1 merged dataset

## Limitations

- CTS batch conversion can fail intermittently, so some attributes come back empty and are left blank rather than imputed
- Retrieval is limited to non-systematic identifiers and chemical properties; name/ontology descriptors and missing-attribute rectification require other Step 2 descriptors and the Step 3 curation process
- Retrieval quality depends on the InChIKeys produced in Step 1; metabolites that could not be translated to InChIKeys are out of scope

## Evidence

- [abstract] Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench): "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [methods] Using Python 3.10.6 with REST/API connectivity, submit each InChIKey to the Chemical Translator Service (CTS) to retrieve molecular weight (MW), molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number.: [evidence span withheld — anchor status: not_found]
- [methods] Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing.: [evidence span withheld — anchor status: not_found]
