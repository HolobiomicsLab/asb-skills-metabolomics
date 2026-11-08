---
name: metabolite-identifier-mapping
description: Use when you have a table of InChIKeys from merged metabolomics datasets
  and need to attach MW, formula, and cross-database IDs (ChEBI, HMDB, KEGG, LipidMaps,
  DrugBank, CAS) for interstudy comparability.
when_to_use_negative:
- Input metabolites are given only as common names/synonyms with no InChIKeys — use
  MMS Step 1 (translation and merging by InChIKeys) first
- The identifiers you need are only the title name (PubChem) or RefMet name (Metabolomics
  Workbench) descriptors — those belong to other Step 2 attribute categories
- A dataset already contains all eight attribute columns populated with no gaps —
  re-querying CTS adds no value
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3314
tools:
- name: Chemical Translator Service (CTS)
  role: REST service queried with each InChIKey to return MW, molecular formula, and
    ChEBI/HMDB/KEGG/LipidMaps/DrugBank/CAS identifiers
- name: Python
  role: Python 3.10.6 with REST/API connectivity used to submit InChIKeys to CTS and
    assemble attribute_table.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:0539c88242b7fc300984418fa738d50d6656f3cbbd5233479b3b7597924c74a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translator Service (CTS)
  - Google Colaboratory
  - Jupyter Notebook
  - MetaboAnalyst
  - Python
  - pip3
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-identifier-mapping

## Summary

Retrieves metabolite attributes (molecular weight, molecular formula, and non-systematic identifiers: ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS number) for each InChIKey via the Chemical Translator Service (CTS), as Step 2 of the Metabolites Merging Strategy (MMS).

## When to use

Use when you have a table of InChIKeys from merged metabolomics datasets and need to attach MW, formula, and cross-database IDs (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) for interstudy comparability.

## When NOT to use

- Input metabolites are given only as common names/synonyms with no InChIKeys — use MMS Step 1 (translation and merging by InChIKeys) first
- The identifiers you need are only the title name (PubChem) or RefMet name (Metabolomics Workbench) descriptors — those belong to other Step 2 attribute categories
- A dataset already contains all eight attribute columns populated with no gaps — re-querying CTS adds no value

## Inputs

- merged metabolite dataset from MMS Step 1 containing a column of unique InChIKeys
- list of InChIKey strings used as query keys

## Outputs

- attribute_table.csv with one row per input InChIKey and eight CTS-sourced attribute columns (MW, formula, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS), blank where the service returned nothing

## How to apply

Load the merged dataset produced by MMS Step 1 (InChIKey-based merging of the metabolite datasets) and extract the unique InChIKey column as query keys. Using Python 3.10.6 with REST/API connectivity, submit each InChIKey to the Chemical Translator Service (CTS) to retrieve molecular weight, molecular formula, and the non-systematic identifiers CheBI, HMDB, KEGG, LipidMaps, DrugBank, and CAS. Assemble the retrieved values into one row per InChIKey and write attribute_table.csv, keying each row on the InChIKey. Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing; missing attributes can later be addressed in MMS Step 3's curation process.

## Related tools

- **Chemical Translator Service (CTS)** (REST service queried with each InChIKey to return MW, molecular formula, and ChEBI/HMDB/KEGG/LipidMaps/DrugBank/CAS identifiers)
- **Python** (Python 3.10.6 with REST/API connectivity used to submit InChIKeys to CTS and assemble attribute_table.csv)

## Examples

```
import requests; recs=[requests.get(f'https://cts.fiehnlab.ucdavis.edu/rest/convert/InChIKey/to/all/{k}').json() for k in inchikeys]
```

## Evaluation signals

- attribute_table.csv exists and contains one row per input InChIKey
- The table includes all eight CTS-sourced attribute columns: MW, formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS
- Blank cells occur only where the CTS service returned nothing, not from imputation
- MW and molecular formula values are internally consistent with the compound encoded by each InChIKey

## Limitations

- Batch conversion via CTS can fail intermittently, so individual attributes may be missing and must be left blank rather than imputed
- Non-systematic identifiers are database-specific (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS); CTS coverage is not guaranteed for every InChIKey

## Evidence

- [abstract] Step 2 attribute retrieval: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [methods] CTS query finding: [evidence span withheld — anchor status: not_found]
- [full_text] blank-cell handling: [evidence span withheld — anchor status: not_found]
