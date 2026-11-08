---
name: metabolite-name-to-inchikey-translation
description: Use when a metabolomics dataset (e.g. Metabolomics Workbench study ST001039,
  Supplementary Table S3) reports metabolites by non-systematic names (PubChem, ChEBI,
  HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS) and InChIKeys are needed for merging.
when_to_use_negative:
- The dataset already reports systematic InChIKeys — MMS Step 1 translates names only
  'if needed' and re-querying would be redundant.
- The goal is retrieving attributes (title name, RefMet name, molecular weight, ontology)
  from an existing InChIKey — that is MMS Step 2, not name translation.
- The task is resolving conjugated base/acid disparities, missing attributes, or synonym
  duplication — those belong to the MMS Step 3 three-step curation process.
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: PubChem Identifier Exchange Service
  role: REST/API service queried for each metabolite name to retrieve the corresponding
    InChIKey used as the systematic merge identifier
- name: Python
  role: Scripting language (version 3.10.6) used to call the PubChem Identifier Exchange
    Service via API/REST connectivity and build the name-to-InChIKey mapping table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:b31e61da92639f8eb6021b581859b1f8a06bf6db231bb15355c5dca99b3ed76d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem Identifier Exchange Service
  - Python
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-name-to-inchikey-translation

## Summary

Translates metabolite names reported in a source dataset into systematic 27-character InChIKeys using the PubChem Identifier Exchange Service, as Step 1 of the Metabolites Merging Strategy (MMS) to enable dataset merging and interstudy comparability.

## When to use

Use when a metabolomics dataset (e.g. Metabolomics Workbench study ST001039, Supplementary Table S3) reports metabolites by non-systematic names (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS) and InChIKeys are needed for merging.

## When NOT to use

- The dataset already reports systematic InChIKeys — MMS Step 1 translates names only 'if needed' and re-querying would be redundant.
- The goal is retrieving attributes (title name, RefMet name, molecular weight, ontology) from an existing InChIKey — that is MMS Step 2, not name translation.
- The task is resolving conjugated base/acid disparities, missing attributes, or synonym duplication — those belong to the MMS Step 3 three-step curation process.

## Inputs

- List of metabolite names from a source dataset (e.g. Supplementary Table S3 of Metabolomics Workbench study ST001039)
- Source datasets using non-systematic identifiers (PubChem, ChEBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID, CAS number)

## Outputs

- Mapping table with one row per input metabolite name: original name, retrieved InChIKey (or empty), and a status column flagging unresolved names
- InChIKey-annotated dataset ready for MMS Step 1 merging

## How to apply

Load the metabolite name list from the source dataset (e.g. study I ST001039 from Metabolomics Workbench) and, for each name, call the PubChem Identifier Exchange Service via a Python script (version 3.10.6) using API/REST connectivity to retrieve the corresponding InChIKey. Record the retrieved InChIKey next to the original name in a mapping table. Where the service returns no match, leave the InChIKey cell empty and flag the name as unresolved in a dedicated status column rather than silently dropping it. This translation is performed only 'if needed' within MMS Step 1, since some datasets already supply InChIKeys; the resulting keys are what MMS uses to merge datasets before Step 2 attribute retrieval (title name from PubChem, RefMet name from Metabolomics Workbench, chemical properties).

## Related tools

- **PubChem Identifier Exchange Service** (REST/API service queried for each metabolite name to retrieve the corresponding InChIKey used as the systematic merge identifier)
- **Python** (Scripting language (version 3.10.6) used to call the PubChem Identifier Exchange Service via API/REST connectivity and build the name-to-InChIKey mapping table)

## Examples

```
python name_to_inchikey.py --input ST001039_metabolite_names.txt --pubchem-identifier-exchange --output name_inchikey_map.csv --unresolved-flagged
```

## Evaluation signals

- Mapping table contains exactly one row per input metabolite name (no rows added or lost).
- Every resolved entry holds a 27-character InChIKey.
- Unresolved names are explicitly flagged in a dedicated status column, never silently dropped.
- Spot-check a subset of known metabolites (e.g. from ST001039) to confirm the returned InChIKey corresponds to the reported name.
- Count of empty InChIKey cells equals count of flagged unresolved names.

## Limitations

- Names for which the PubChem Identifier Exchange Service returns no match remain unresolved and must be flagged rather than merged.
- Translation coverage depends on external API/REST connectivity and PubChem's name coverage for the identifiers used in the source dataset.
- The InChIKey obtained in this step does not itself fix synonym duplication or conjugate base/acid disparities; those require the downstream MMS curation steps.

## Evidence

- [abstract] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed): "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [methods] For each metabolite name, call the PubChem Identifier Exchange Service via a Python script (version 3.10.6) using API/REST connectivity to retrieve the corresponding InChIKey.: [evidence span withheld — anchor status: not_found]
- [abstract] we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
