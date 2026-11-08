---
name: inchikey-based-database-querying
description: Use when you hold a merged metabolite table keyed on unique InChIKeys
  and need to populate per-metabolite attributes (name descriptors, MW, formula, database
  IDs) for downstream interstudy comparability checks.
when_to_use_negative:
- The metabolite names have not yet been translated to InChIKeys — run MMS Step 1
  (translation and merging by InChIKeys) first, since this skill queries CTS per InChIKey.
- The dataset already carries the eight attribute columns from a prior MMS run and
  only needs disparity rectification (conjugated base/acid compounds, synonym checking)
  — that belongs to Step 3 curation, not Step 2 retrieval.
- You need systematic identifier translation between arbitrary databases with no InChIKey
  keys — CTS batch conversion is queried here strictly per unique InChIKey from the
  merged table.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: Chemical Translator Service (CTS)
  role: External REST service queried with each InChIKey to return MW, molecular formula,
    CheBI, HMDB, KEGG, LipidMaps, DrugBank, and CAS identifiers
- name: Python 3.10.6
  role: Scripting environment used to submit InChIKeys to CTS over REST/API and assemble
    attribute_table.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:15ad79027d906a1b1eda4568a57424dbc894020986ed47edc29cff032b126f51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chemical Translator Service (CTS)
  - Python
  - Python 3.10.6
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# inchikey-based-database-querying

## Summary

Step 2 of the Metabolites Merging Strategy (MMS): retrieve chemical descriptors and non-systematic identifiers (MW, molecular formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) for each merged InChIKey by querying the Chemical Translator Service (CTS), writing one row per InChIKey to attribute_table.csv.

## When to use

Use when you hold a merged metabolite table keyed on unique InChIKeys and need to populate per-metabolite attributes (name descriptors, MW, formula, database IDs) for downstream interstudy comparability checks.

## When NOT to use

- The metabolite names have not yet been translated to InChIKeys — run MMS Step 1 (translation and merging by InChIKeys) first, since this skill queries CTS per InChIKey.
- The dataset already carries the eight attribute columns from a prior MMS run and only needs disparity rectification (conjugated base/acid compounds, synonym checking) — that belongs to Step 3 curation, not Step 2 retrieval.
- You need systematic identifier translation between arbitrary databases with no InChIKey keys — CTS batch conversion is queried here strictly per unique InChIKey from the merged table.

## Inputs

- Merged dataset from MMS Step 1 with a unique InChIKey column per metabolite
- Chemical Translator Service (CTS) REST/API endpoint

## Outputs

- attribute_table.csv — one row per input InChIKey with eight CTS-sourced attribute columns: MW, molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number

## How to apply

Load the merged dataset produced by MMS Step 1 and extract the unique InChIKey column as the query keys. Using Python 3.10.6 with REST/API connectivity, submit each InChIKey to the Chemical Translator Service (CTS) to retrieve molecular weight (MW), molecular formula, CheBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number. Assemble the retrieved values into one row per InChIKey, keying each row on the InChIKey, and write the table to attribute_table.csv. Where CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing — missing attributes are handled later in Step 3 of the MMS curation process. The rationale is that InChIKeys act as the systematic anchor, so all eight non-systematic identifiers can be resolved from a single systematic key.

## Related tools

- **Chemical Translator Service (CTS)** (External REST service queried with each InChIKey to return MW, molecular formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, and CAS identifiers)
- **Python 3.10.6** (Scripting environment used to submit InChIKeys to CTS over REST/API and assemble attribute_table.csv)

## Evaluation signals

- attribute_table.csv exists and contains exactly one row per unique input InChIKey from the Step 1 merged dataset.
- The table includes all eight CTS-sourced attribute columns: MW, formula, CheBI, HMDB, KEGG, LipidMaps, DrugBank, CAS.
- Blanks appear only where CTS returned nothing for an attribute (batch conversion failures), with no imputed or fabricated values.
- Each row is keyed on its InChIKey, so joined records trace back one-to-one to the Step 1 merged dataset.
- Retrieved MW and molecular formula values are chemically consistent with the corresponding InChIKeys.

## Limitations

- CTS batch conversion can fail intermittently, so some attribute cells will legitimately come back blank; these are left empty and reconciled in the later three-step curation process rather than imputed at this step.
- Coverage depends on CTS: metabolites absent from the mapped databases (ChEBI, HMDB, KEGG, LipidMaps, DrugBank, CAS) will yield partial rows.
- Only the eight listed attributes are retrieved here; other descriptors such as title name from PubChem and RefMet name from Metabolomics Workbench belong to the broader Step 2 attribute retrieval in the article.

## Evidence

- [other] The attribute-retrieval step of MMS queries the Chemical Translator Service with each InChIKey: [evidence span withheld — anchor status: not_found]
- [abstract] Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties (molecular weight and: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties (molecular weight and"
- [other] Where the CTS returns no value for an attribute (batch conversion can fail intermittently), leave the cell blank rather than imputing.: [evidence span withheld — anchor status: not_found]
