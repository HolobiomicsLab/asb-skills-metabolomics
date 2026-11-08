---
name: rest-api-querying-pubchem
description: Use when metabolite names (or non-systematic identifiers such as CAS
  numbers, HMDB, KEGG or LIPID MAPS IDs) reported in a source metabolomics dataset
  must be converted to systematic InChIKeys for cross-dataset merging, as in MMS Step
  1.
when_to_use_negative:
- Input metabolites already carry InChIKeys, so no name-to-InChIKey translation is
  needed
- The task is MMS Step 2 (attribute retrieval from the InChIKey, e.g. PubChem title
  name, RefMet name, molecular weight) rather than Step 1 identifier translation
- Only a single metabolite needs lookup and a manual PubChem web query would suffice
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PubChem Identifier Exchange Service
  role: REST/web service called for each metabolite name to retrieve the corresponding
    InChIKey
- name: Python
  role: Scripting language (version 3.10.6) used to loop over the metabolite name
    list and perform the API/REST calls to PubChem
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:c42c3810732a16edbddcc70442367aeb2abaa373f0124198695eb8f94613a6b4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem Identifier Exchange Service
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rest-api-querying-pubchem

## Summary

Queries the PubChem Identifier Exchange Service over REST from a Python script (3.10.6) to translate metabolite names reported in a source dataset into the 27-character InChIKey identifier used by the Metabolites Merging Strategy (MMS) Step 1 for merging datasets from different studies.

## When to use

Use when metabolite names (or non-systematic identifiers such as CAS numbers, HMDB, KEGG or LIPID MAPS IDs) reported in a source metabolomics dataset must be converted to systematic InChIKeys for cross-dataset merging, as in MMS Step 1.

## When NOT to use

- Input metabolites already carry InChIKeys, so no name-to-InChIKey translation is needed
- The task is MMS Step 2 (attribute retrieval from the InChIKey, e.g. PubChem title name, RefMet name, molecular weight) rather than Step 1 identifier translation
- Only a single metabolite needs lookup and a manual PubChem web query would suffice

## Inputs

- List of metabolite names from a source dataset (e.g. Supplementary Table S3 of study I ST001039 from Metabolomics Workbench)
- Source datasets reporting non-systematic identifiers (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS number)

## Outputs

- Mapping table with one row per input metabolite name, holding the original name, the retrieved 27-character InChIKey, and a dedicated status column flagging unresolved names

## How to apply

Load the metabolite name list from the source dataset (e.g. study I ST001039 from Metabolomics Workbench, Supplementary Table S3). For each metabolite name, call the PubChem Identifier Exchange Service via a Python script (version 3.10.6) using API/REST connectivity to retrieve the corresponding InChIKey. Record the retrieved InChIKey next to the original name in the mapping table; where the service returns no match, leave the InChIKey cell empty and flag the name as unresolved in a dedicated status column. The rationale is that Step 1 of MMS requires a single systematic identifier (InChIKey) to merge datasets that otherwise report heterogeneous non-systematic identifiers (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number).

## Related tools

- **PubChem Identifier Exchange Service** (REST/web service called for each metabolite name to retrieve the corresponding InChIKey)
- **Python** (Scripting language (version 3.10.6) used to loop over the metabolite name list and perform the API/REST calls to PubChem)

## Examples

```
requests.get('https://pubchem.ncbi.nlm.nih.gov/idexchange/idexchange.cgi', params={'inputid': metabolite_name, 'idType': 'SerializedID', 'outputType': 'InChIKey', 'get': 'Get'})
```

## Evaluation signals

- Mapping table contains exactly one row per input metabolite name (row count matches input name count)
- Every resolved entry holds a valid 27-character InChIKey
- Unresolved names are explicitly flagged in the status column rather than silently dropped
- No resolved row has an empty InChIKey cell

## Limitations

- The PubChem Identifier Exchange Service may return no match for a given metabolite name, leaving the InChIKey empty and the name flagged as unresolved
- Unresolved names remain unmerged because MMS Step 1 integration relies on the InChIKey as the join key

## Evidence

- [methods] When metabolite names are reported in a source dataset, MMS Step 1 uses the PubChem Identifier Exchange Service to retrieve the corresponding InChIKey.: [evidence span withheld — anchor status: not_found]
- [abstract] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed): "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
