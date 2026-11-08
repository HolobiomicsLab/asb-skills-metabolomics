---
name: cross-dataset-provenance-merging
description: Use when metabolite entries from datasets being merged carry non-systematic
  identifiers (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID) that
  must be converted to InChIKeys before datasets can be integrated.
when_to_use_negative:
- The metabolite entries already carry InChIKeys, so no identifier translation branch
  is needed
- The task is MMS Step 2 attribute retrieval (PubChem title name, RefMet name, molecular
  weight) or Step 3 curation rather than identifier harmonization
- Only a single dataset is being processed with no cross-study merging goal, so no
  interstudy harmonization is required
tools:
- name: Chemical Translation Service (CTS)
  role: REST service called from Python to convert each non-systematic identifier
    (e.g. CAS, HMDB ID, KEGG ID) into an InChIKey
- name: Python
  role: Language used to drive the CTS REST calls, merge translated InChIKeys with
    original identifiers, and write the mapping table to disk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:3f2124f45c24be55ded4cbb84c82b4c7ffeca80c75742ba899a6c9ea4ab469cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translation Service (CTS)
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-dataset-provenance-merging

## Summary

Translates non-InChIKey metabolite identifiers (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank IDs) into InChIKeys via the Chemical Translation Service (CTS) and merges the results with name-derived InChIKeys, as Step 1 of the Metabolites Merging Strategy (MMS).

## When to use

Use when metabolite entries from datasets being merged carry non-systematic identifiers (CAS, HMDB, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID) that must be converted to InChIKeys before datasets can be integrated.

## When NOT to use

- The metabolite entries already carry InChIKeys, so no identifier translation branch is needed
- The task is MMS Step 2 attribute retrieval (PubChem title name, RefMet name, molecular weight) or Step 3 curation rather than identifier harmonization
- Only a single dataset is being processed with no cross-study merging goal, so no interstudy harmonization is required

## Inputs

- Metabolite entries with non-InChIKey identifiers from Metabolomics Workbench studies (ST001039, ST001048, ST001317)
- HMDB asthma-urine metabolite entries carrying HMDB IDs
- Metabolite tables from the six literature articles LT01-06 with non-systematic identifiers (CAS, KEGG, PubChem, ChEBI, LipidMaps, DrugBank, Bin ID)

## Outputs

- Merged identifier-to-InChIKey mapping table written to disk, with one row per submitted identifier
- Reference column recording the originating source for each identifier-to-InChIKey mapping
- List of flags for identifiers CTS failed to convert, for use in the later missing-attributes retrieval step

## How to apply

Load the metabolite entries from each case-study source (e.g. Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB asthma-urine entries, and the six literature articles LT01-06) and select those whose reported identifier is not an InChIKey. For each such entry, call the Chemical Translator Service (CTS) via its REST interface using Python, requesting conversion of the non-systematic identifier into an InChIKey. Merge the returned InChIKeys with the original identifier values, recording the originating source in a Reference column as the MMS prescribes, and write the merged identifier-to-InChIKey mapping table to disk for use in subsequent MMS steps. Identifiers that CTS fails to convert are flagged rather than dropped, so they can be handled by the later missing-attributes retrieval step of the curation process. The rationale is that MMS harmonizes heterogeneous studies onto the systematic InChIKey representation so that datasets from different sources become intercomparable.

## Related tools

- **Chemical Translation Service (CTS)** (REST service called from Python to convert each non-systematic identifier (e.g. CAS, HMDB ID, KEGG ID) into an InChIKey)
- **Python** (Language used to drive the CTS REST calls, merge translated InChIKeys with original identifiers, and write the mapping table to disk)

## Examples

```
import requests; r = requests.get(CTS_REST_URL, params={"from": "HMDB", "to": "InChIKey", "value": "HMDB0000042"}); inchikey = r.json()[0]["result"]
```

## Evaluation signals

- The output mapping table exists on disk and contains one row per submitted identifier
- Every successfully translated row has an InChIKey populated in the InChIKey field
- Source provenance is retained: the Reference column records the originating source for each mapping
- Any identifiers CTS failed to convert are flagged rather than silently dropped, and are available for the missing-attributes retrieval step

## Limitations

- The CTS cannot convert every non-systematic identifier; failures must be flagged and deferred to the later missing-attributes retrieval step
- Translation quality depends on CTS coverage of the identifier vocabularies handled in MMS (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS number)

## Evidence

- [abstract] Step 1: translation and merging of datasets by employing InChIKeys for data integration: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [abstract] non-systematic identifiers handled by MMS: "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [abstract] MMS purpose: harmonization for interstudy comparability: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
