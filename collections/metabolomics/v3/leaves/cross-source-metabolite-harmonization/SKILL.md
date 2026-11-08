---
name: cross-source-metabolite-harmonization
description: Use when multiple metabolite datasets from different sources (e.g. Metabolomics
  Workbench studies, HMDB, literature tables) must be merged on a common key and annotated
  with identifiers and ontology for intercomparison.
when_to_use_negative:
- Input is already a single harmonized table with systematic identifiers — merging
  would only add redundant curation overhead.
- Only one dataset source is involved and no interstudy comparison is needed; the
  merge key and deduplication machinery add no value.
- Compounds cannot be translated to InChIKeys via PubChem or CTS — the merge key is
  undefined and rows cannot be joined reliably.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names to InChIKeys in Step 1 and retrieves title name,
    CID, InChI and SMILES per InChIKey in Step 2
- name: Chemical Translation Service (CTS)
  role: Translates non-InChIKey identifiers and retrieves molecular weight, molecular
    formula, ChEBI, HMDB, KEGG, LipidMaps, DrugBank IDs and CAS numbers
- name: Metabolomics Workbench / RefMet
  role: Source of urinary asthma study metabolite lists and provider of the RefMet
    accepted name attribute
- name: ClassyFire
  role: Assigns chemical ontology (kingdom, superclass, class, subclass) using InChI
    as input
- name: BinVestigate
  role: Supplies the Bin ID attribute per InChIKey in Step 2
- name: HMDB
  role: Source dataset filtered by urine biospecimen location and asthma-associated
    disorders; provides HMDB ID for bidirectional synonym checking
- name: Python
  role: Implements the merge, attribute retrieval, and three-step curation workflow
    programmatically
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:9c71304aa584c6bf273fc6575b5b5640ad6700253ab86901dc8e7006412c94b9
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
  - Metabolomics Workbench / RefMet
  - PubChem Identifier Exchange Service
  - Python
  - RefMet
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# cross-source-metabolite-harmonization

## Summary

Implements the three-step Metabolites Merging Strategy (MMS) to harmonize multiple metabolite datasets by translating names to InChIKeys, retrieving attributes per InChIKey, and applying a three-step curation process, enabling interstudy comparability of metabolomics datasets.

## When to use

Use when multiple metabolite datasets from different sources (e.g. Metabolomics Workbench studies, HMDB, literature tables) must be merged on a common key and annotated with identifiers and ontology for intercomparison.

## When NOT to use

- Input is already a single harmonized table with systematic identifiers — merging would only add redundant curation overhead.
- Only one dataset source is involved and no interstudy comparison is needed; the merge key and deduplication machinery add no value.
- Compounds cannot be translated to InChIKeys via PubChem or CTS — the merge key is undefined and rows cannot be joined reliably.

## Inputs

- Metabolite lists from multiple sources (e.g. Metabolomics Workbench studies ST001039, ST001048, ST001317 filtered by species human, sample source urine, disease asthma; HMDB entries filtered by urine biospecimen location and asthma-associated disorders; PubMed literature metabolite tables)
- Metabolite names or non-InChIKey identifiers requiring translation
- InChIKey used as the merge key and as ClassyFire input via InChI

## Outputs

- Pre-curation merged table keyed on InChIKey with a 'Reference' column recording every contributing source (e.g. 547 rows for the urinary asthma case)
- Attribute-enriched merged dataset: PubChem title name, CID, InChI, SMILES, RefMet name, molecular weight, molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs, CAS number, Bin ID, and ClassyFire ontology (kingdom, superclass, class, subclass)
- Final curated merged dataset saved as merged_metabolites.csv (e.g. exactly 391 metabolite rows)

## How to apply

Step 1: translate metabolite names to InChIKeys via the PubChem Identifier Exchange Service, and translate any non-InChIKey identifiers via the Chemical Translation Service (CTS); merge all datasets on InChIKey, combining duplicates and recording each source in a 'Reference' column (e.g. the urinary asthma case reached 547 pre-curation rows from ten sources). Step 2: retrieve attributes per InChIKey — PubChem title name, CID, InChI and SMILES from PubChem; RefMet accepted name from Metabolomics Workbench; molecular weight, molecular formula, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID and CAS number from CTS; Bin ID from BinVestigate; and ontology (kingdom, superclass, class, subclass) from ClassyFire using InChI as input. Step 3a (optional): convert conjugated base/acid compounds to non-conjugated forms by comparing the first 14 characters of the 27-character InChIKey (molecular skeleton layer), deleting conjugate entries while preserving provenance in the 'Reference' column. Step 3b: re-query PubChem and CTS to fill missing attributes that failed during batch conversion. Step 3c: check all bidirectional attributes (RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number, PubChem CID) for non-unique and conflicting synonyms, merging duplicates while keeping source provenance; save the curated table as merged_metabolites.csv.

## Related tools

- **PubChem Identifier Exchange Service** (Translates metabolite names to InChIKeys in Step 1 and retrieves title name, CID, InChI and SMILES per InChIKey in Step 2)
- **Chemical Translation Service (CTS)** (Translates non-InChIKey identifiers and retrieves molecular weight, molecular formula, ChEBI, HMDB, KEGG, LipidMaps, DrugBank IDs and CAS numbers)
- **Metabolomics Workbench / RefMet** (Source of urinary asthma study metabolite lists and provider of the RefMet accepted name attribute)
- **ClassyFire** (Assigns chemical ontology (kingdom, superclass, class, subclass) using InChI as input)
- **BinVestigate** (Supplies the Bin ID attribute per InChIKey in Step 2)
- **HMDB** (Source dataset filtered by urine biospecimen location and asthma-associated disorders; provides HMDB ID for bidirectional synonym checking)
- **Python** (Implements the merge, attribute retrieval, and three-step curation workflow programmatically)

## Examples

```
python merge_mms.py --sources ST001039,ST001048,ST001313,HMDB,literature --key InChIKey --output merged_metabolites.csv
```

## Evaluation signals

- Pre-curation merged table contains exactly the expected number of InChIKey rows (547 for the ten urinary asthma sources).
- Final curated merged dataset contains exactly the expected number of metabolite rows (391) before saving as merged_metabolites.csv.
- Every retained row carries a non-empty 'Reference' column entry tracing all contributing sources, including deleted conjugate entries whose provenance was transferred to the retained non-conjugated compound.
- All bidirectional attributes (RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number, PubChem CID) have been checked for non-unique and conflicting synonyms.
- Conjugated base/acid conversions agree on the first 14 characters of the 27-character InChIKey (molecular skeleton layer) between conjugate and non-conjugated forms.

## Limitations

- Step 3a conjugate conversion is optional and can delete conjugate entries; provenance must be preserved manually in the 'Reference' column of the retained compound.
- Batch attribute conversion can fail, requiring the Step 3b re-query loop against PubChem and CTS to fill missing attributes.
- Synonym checking is needed because identifiers can be non-unique or conflicting across sources; unresolved conflicts cause duplicated information.

## Evidence

- [abstract] MMS is a three-step framework to harmonize multiple metabolite datasets using InChIKeys, attribute retrieval, and a three-step curation process: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [methods] Step 1 translation and merging: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [methods] Step 2 attribute retrieval: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [methods] Step 3 curation process: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [results] 547 retrieved and 391 final metabolites: "Following the MMS, a total of 547 metabolites have been retrieved and merged, leading to a final dataset of 391 metabolites."
