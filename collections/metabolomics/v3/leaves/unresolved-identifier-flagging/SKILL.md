---
name: unresolved-identifier-flagging
description: Use when a source metabolomics dataset reports metabolite names (e.g.
  study ST001039 from Metabolomics Workbench) and some PubChem Identifier Exchange
  lookups return no InChIKey match that must be tracked.
when_to_use_negative:
- Identifiers in the source dataset are already systematic (e.g. InChIKeys), so no
  translation or flagging is needed
- The dataset uses RefMet names from Metabolomics Workbench, which are already standardized
  and are used directly in MMS harmonization
- The task is Step 2 attribute retrieval or Step 3 curation (conjugated base/acid
  rectification, missing attributes, synonym checking) rather than Step 1 translation
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3315
tools:
- name: PubChem Identifier Exchange Service
  role: External REST service that translates each metabolite name into its corresponding
    InChIKey; non-matches trigger the unresolved flagging
- name: Python (3.10.6)
  role: Scripting language used to call the service via API/REST connectivity and
    to write the mapping table and status column
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:497fc470e7d46e0d220989ec080c568fc19df29aa0b87eb8abdd11995c778dcf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem Identifier Exchange Service
  - Python
  - Python (3.10.6)
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# unresolved-identifier-flagging

## Summary

A Step-1 bookkeeping practice from the Metabolites Merging Strategy (MMS): after translating metabolite names to InChIKeys via the PubChem Identifier Exchange Service, names for which the service returns no match are left with an empty InChIKey cell and explicitly flagged in a dedicated status column instead of being silently dropped.

## When to use

Use when a source metabolomics dataset reports metabolite names (e.g. study ST001039 from Metabolomics Workbench) and some PubChem Identifier Exchange lookups return no InChIKey match that must be tracked.

## When NOT to use

- Identifiers in the source dataset are already systematic (e.g. InChIKeys), so no translation or flagging is needed
- The dataset uses RefMet names from Metabolomics Workbench, which are already standardized and are used directly in MMS harmonization
- The task is Step 2 attribute retrieval or Step 3 curation (conjugated base/acid rectification, missing attributes, synonym checking) rather than Step 1 translation

## Inputs

- Metabolite name list from a source dataset (e.g. Supplementary Table S3 of study I ST001039 from Metabolomics Workbench)
- PubChem Identifier Exchange Service responses (matched InChIKeys or empty results)

## Outputs

- Name-to-InChIKey mapping table with one row per input metabolite name
- Dedicated status column flagging names as resolved or unresolved
- Empty InChIKey cells for names the service could not resolve

## How to apply

Build a mapping table with one row per input metabolite name. For each name, call the PubChem Identifier Exchange Service (or PubChem PUG-REST) via a Python script (version 3.10.6) using API/REST connectivity and record the retrieved InChIKey next to the original name. When the service returns no match, leave the InChIKey cell empty and set a dedicated status column to 'unresolved' so the failure is visible. The rationale is MMS Step 1's goal of complete, auditable translation for merging: unresolved entries must remain traceable because Step 3's synonym-checking curation step may still reconcile duplicated or missing information later. Validate that the table is row-complete before passing resolved InChIKeys on to Step 2 attribute retrieval.

## Related tools

- **PubChem Identifier Exchange Service** (External REST service that translates each metabolite name into its corresponding InChIKey; non-matches trigger the unresolved flagging)
- **Python (3.10.6)** (Scripting language used to call the service via API/REST connectivity and to write the mapping table and status column)

## Examples

```
import requests; ik = requests.get(f'https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/{name}/property/InChIKey/TXT').text.strip(); status = 'resolved' if len(ik) == 27 else 'unresolved'
```

## Evaluation signals

- Mapping table contains exactly one row per input metabolite name from the source dataset
- Every resolved entry holds a 27-character InChIKey
- Every name the service returned no match for has an empty InChIKey cell AND an explicit 'unresolved' flag in the status column
- No input names are silently dropped from the table (row count equals input name count)
- Spot-check a sample of resolved InChIKeys against PubChem to confirm they correspond to the original metabolite names

## Limitations

- The PubChem Identifier Exchange Service may return no match for some names, so those metabolites remain unresolved (empty InChIKey) and cannot participate in InChIKey-based merging until reconciled by Step 3 curation
- Flagging records the failure but does not itself resolve it; unresolved names still require later synonym-checking or attribute-retrieval curation
- Translation depends on external REST service availability and Python API/REST connectivity

## Evidence

- [methods] where the service returns no match, leave the InChIKey cell empty and flag the name as unresolved in a dedicated status column: [evidence span withheld — anchor status: not_found]
- [abstract] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
