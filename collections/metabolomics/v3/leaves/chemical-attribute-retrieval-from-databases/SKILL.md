---
name: chemical-attribute-retrieval-from-databases
description: Use when a merged metabolite table keyed on InChIKeys lacks attributes
  such as CID, SMILES, RefMet name, HMDB/KEGG/ChEBI IDs, or chemical ontology and
  must be enriched before interstudy comparison.
when_to_use_negative:
- The metabolite list is not yet merged on InChIKeys — attribute retrieval in MMS
  is keyed on the InChIKey, so run Step 1 (translation and merging via PubChem Identifier
  Exchange Service and CTS) first
- Only one identifier type is needed for a single metabolite — the full multi-database
  sweep (PubChem, RefMet, CTS, BinVestigate, ClassyFire) is designed for whole-dataset
  harmonization, not one-off lookups
- The dataset uses non-standard identifiers without translatable names and no access
  to CTS or PubChem Identifier Exchange Service, since translation to InChIKey is
  a prerequisite
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names to InChIKeys and retrieves title name, CID, InChI,
    and SMILES per InChIKey
- name: Chemical Translation Service (CTS)
  role: Translates non-InChIKey identifiers and retrieves molecular weight, molecular
    formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs, and CAS number
- name: RefMet (Metabolomics Workbench)
  role: Supplies the RefMet accepted name for each InChIKey
- name: ClassyFire
  role: Returns chemical ontology (kingdom, superclass, class, subclass) using the
    InChI as input
- name: BinVestigate
  role: Retrieves the Bin ID for each compound
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:930e1fb9bd8cf535a77a8d00f8d04f799f1de7511a0ff28bd994426efa7a18c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinVestigate
  - ChEBI
  - Chemical Translation Service (CTS)
  - ClassyFire
  - DrugBank
  - HMDB
  - KEGG
  - LipidMaps
  - Metabolomics Workbench
  - PubChem Identifier Exchange Service
  - Python
  - RefMet
  - RefMet (Metabolomics Workbench)
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# chemical-attribute-retrieval-from-databases

## Summary

Retrieves standardized chemical descriptors and identifiers for every metabolite in a merged dataset by querying PubChem, RefMet, CTS, ClassyFire, and BinVestigate using the InChIKey as the lookup key, as defined in Step 2 of the Metabolites Merging Strategy (MMS).

## When to use

Use when a merged metabolite table keyed on InChIKeys lacks attributes such as CID, SMILES, RefMet name, HMDB/KEGG/ChEBI IDs, or chemical ontology and must be enriched before interstudy comparison.

## When NOT to use

- The metabolite list is not yet merged on InChIKeys — attribute retrieval in MMS is keyed on the InChIKey, so run Step 1 (translation and merging via PubChem Identifier Exchange Service and CTS) first
- Only one identifier type is needed for a single metabolite — the full multi-database sweep (PubChem, RefMet, CTS, BinVestigate, ClassyFire) is designed for whole-dataset harmonization, not one-off lookups
- The dataset uses non-standard identifiers without translatable names and no access to CTS or PubChem Identifier Exchange Service, since translation to InChIKey is a prerequisite

## Inputs

- Merged metabolite table (e.g. pre-curation merged CSV with 547 rows) keyed on 27-character InChIKeys with a 'Reference' column recording each source
- InChI strings from PubChem (used as ClassyFire input)
- Dataset identifiers from Metabolomics Workbench studies (e.g. ST001039, ST001048, ST001317) for RefMet name lookup

## Outputs

- Attribute-enriched merged metabolites CSV with PubChem title name, CID, InChI, SMILES, RefMet accepted name, molecular weight, molecular formula, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number, Bin ID, and ClassyFire ontology (kingdom, superclass, class, subclass)
- Updated 'Reference' provenance column after duplicate/synonym merging

## How to apply

First ensure the metabolite table is merged on full 27-character InChIKeys (MMS Step 1), since the InChIKey is the input for all attribute lookups. For each InChIKey, retrieve the PubChem title name, CID, InChI, and SMILES from the PubChem Identifier Exchange Service, and the RefMet accepted name from Metabolomics Workbench. From the Chemical Translation Service (CTS), pull molecular weight, molecular formula, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, and CAS number; additionally retrieve the Bin ID from BinVestigate and the ClassyFire ontology (kingdom, superclass, class, subclass) using the InChI as input. If batch conversion leaves attributes missing, re-query PubChem and CTS for those specific entries (MMS Step 3b). Finally, check bidirectional attributes (RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number, PubChem CID) for non-unique and conflicting synonyms, merging duplicates while preserving source provenance in the Reference column.

## Related tools

- **PubChem Identifier Exchange Service** (Translates metabolite names to InChIKeys and retrieves title name, CID, InChI, and SMILES per InChIKey)
- **Chemical Translation Service (CTS)** (Translates non-InChIKey identifiers and retrieves molecular weight, molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs, and CAS number)
- **RefMet (Metabolomics Workbench)** (Supplies the RefMet accepted name for each InChIKey)
- **ClassyFire** (Returns chemical ontology (kingdom, superclass, class, subclass) using the InChI as input)
- **BinVestigate** (Retrieves the Bin ID for each compound)

## Examples

```
python retrieve_attributes.py --inchikeys merged_metabolites.csv --out merged_metabolites.csv --sources pubchem refmet cts classyfire binvestigate
```

## Evaluation signals

- Every row of the merged table has a non-empty 27-character InChIKey before attribute retrieval begins
- The pre-curation merged table contains 547 rows and the final curated dataset contains exactly 391 metabolite rows for the urinary asthma use case
- Attribute columns (PubChem CID, RefMet name, HMDB ID, KEGG ID, ChEBI ID, CAS number, etc.) are populated for the majority of rows; residual gaps are handled by the Step 3b re-query of PubChem and CTS
- Bidirectional attributes show no non-unique or conflicting synonyms after curation; duplicates are merged with sources preserved in the 'Reference' column
- Each retained entry's Reference column preserves the source datasets it came from after conjugate-form collapsing (Step 3a, first-14-character skeleton comparison)

## Limitations

- Attribute retrieval can fail in batch mode; missing attributes must be recovered by re-querying PubChem and CTS (Step 3b)
- Bidirectional identifiers (HMDB ID, KEGG ID, DrugBank ID, CAS number, etc.) can be non-unique or conflicting, requiring synonym checking and manual duplicate merging
- Conjugated base/acid compounds can produce distinct InChIKeys for the same skeleton; collapsing them (Step 3a) is optional and relies on comparing the first 14 characters of the InChIKey
- The strategy depends on external services (PubChem, CTS, ClassyFire, BinVestigate) being available and up to date

## Evidence

- [abstract] MMS framework overview: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [abstract] Step 2 attribute retrieval: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [abstract] Non-systematic identifiers: "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [full_text] Step 2 detailed attribute list: [evidence span withheld — anchor status: not_found]
- [abstract] Step 3 curation: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
