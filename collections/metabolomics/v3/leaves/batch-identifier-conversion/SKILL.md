---
name: batch-identifier-conversion
description: Use when source metabolomics datasets report metabolites by names or
  non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin
  ID, CAS) and you need a unified InChIKey column before merging studies.
when_to_use_negative:
- When source entries already report InChIKeys — no conversion is needed; proceed
  directly to MMS Step 1 merging
- When the goal is retrieving chemical attributes (molecular weight, names, ontology)
  from InChIKeys — that is MMS Step 2 attribute retrieval, not identifier translation
- When curating conjugated base/acid disparities or synonym duplication — that belongs
  to MMS Step 3's three-step curation process
edam_operation: http://edamontology.org/operation_3345
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names from source datasets into InChIKeys via REST calls
- name: Chemical Translation Service (CTS)
  role: Converts non-InChIKey identifiers (CAS, HMDB, KEGG, etc.) into InChIKeys via
    its REST interface
- name: Python
  role: Drives both conversion services; version 3.10.6 reported in the case study
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:df25672f3928a4eeb93ae0966dd84e4106c1a128dfb2b5213dcd38ef07d1bc5c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translation Service (CTS)
  - PubChem Identifier Exchange Service
  - Python
  edam_operation: http://edamontology.org/operation_3345
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# batch-identifier-conversion

## Summary

In MMS Step 1, batch-convert heterogeneous metabolite identifiers (metabolite names or non-systematic IDs such as CAS, HMDB, KEGG, ChEBI, LipidMaps, DrugBank) into 27-character InChIKeys using the PubChem Identifier Exchange Service for names and the Chemical Translation Service (CTS) for non-InChIKey identifiers.

## When to use

Use when source metabolomics datasets report metabolites by names or non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS) and you need a unified InChIKey column before merging studies.

## When NOT to use

- When source entries already report InChIKeys — no conversion is needed; proceed directly to MMS Step 1 merging
- When the goal is retrieving chemical attributes (molecular weight, names, ontology) from InChIKeys — that is MMS Step 2 attribute retrieval, not identifier translation
- When curating conjugated base/acid disparities or synonym duplication — that belongs to MMS Step 3's three-step curation process

## Inputs

- Metabolite name lists from source datasets (e.g. study I ST001039 Supplementary Table S3 from Metabolomics Workbench)
- Metabolite entries carrying non-systematic identifiers (CAS, HMDB ID, KEGG ID, ChEBI, LipidMaps, DrugBank, Bin ID)
- Metabolomics Workbench studies ST001039, ST001048, ST001317; HMDB asthma-urine entries; literature articles LT01-06

## Outputs

- Identifier-to-InChIKey mapping table with one row per input identifier and a Reference column recording source provenance
- Status column flagging unresolved names (no PubChem match) rather than silently dropping them
- Merged InChIKey set used for data integration in MMS Step 1

## How to apply

First split the metabolite entries from each source (e.g. Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB asthma-urine entries, literature articles LT01-06) into two branches: entries already carrying InChIKeys and those with names or other identifiers. For names, call the PubChem Identifier Exchange Service via a Python 3.10.6 script over REST to retrieve the corresponding InChIKey; leave unresolved names with an empty InChIKey cell and an explicit flag in a status column rather than dropping them. For non-InChIKey identifiers, pass them through the Chemical Translator Service (CTS) REST interface with Python to convert them into InChIKeys. Merge both branches into a single identifier-to-InChIKey mapping table, retaining the originating source in a Reference column as the MMS prescribes. This table is the input to MMS Step 2 (attribute retrieval from the InChIKey) and Step 3 (curation of conjugated base/acid disparities, missing attributes, and synonym duplication).

## Related tools

- **PubChem Identifier Exchange Service** (Translates metabolite names from source datasets into InChIKeys via REST calls)
- **Chemical Translation Service (CTS)** (Converts non-InChIKey identifiers (CAS, HMDB, KEGG, etc.) into InChIKeys via its REST interface)
- **Python** (Drives both conversion services; version 3.10.6 reported in the case study)

## Examples

```
python 3.10.6 script calling PubChem Identifier Exchange Service API to convert metabolite names to InChIKeys; then CTS REST call for non-InChIKey IDs
```

## Evaluation signals

- Mapping table contains one row per submitted identifier/name
- Resolved entries hold a 27-character InChIKey
- Unresolved names are explicitly flagged in a status column, not silently dropped
- Source provenance retained in a Reference column for every translated entry
- Output table written to disk and populated before downstream MMS steps run

## Limitations

- PubChem Identifier Exchange and CTS may fail to resolve some names or identifiers; these must be flagged and routed to MMS Step 3's missing-attributes retrieval rather than discarded
- Translation is identifier-system dependent: names require PubChem, non-InChIKey systematic IDs require CTS; a single service will not cover both branches
- Conjugated base/acid disparities and synonym duplication can remain after conversion and require the optional Step 3 curation process

## Evidence

- [methods] When metabolite names are reported in a source dataset, MMS Step 1 uses the PubChem Identifier Exchange Service to retrieve the corresponding InChIKey.: [evidence span withheld — anchor status: not_found]
- [methods] The identifier-translation branch of MMS Step 1 handles entries lacking InChIKeys by passing their non-InChIKey identifiers through the Chemical Translator Service (CTS) to obtain InChIKeys.: [evidence span withheld — anchor status: not_found]
- [abstract] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed): "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
