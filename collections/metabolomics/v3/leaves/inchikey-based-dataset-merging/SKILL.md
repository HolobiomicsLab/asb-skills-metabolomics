---
name: inchikey-based-dataset-merging
description: Use when multiple metabolomics datasets with heterogeneous metabolite
  identifiers (names, HMDB, KEGG, PubChem CIDs, CAS) must be merged and intercompared,
  and a systematic InChIKey-based harmonization with curation is required.
when_to_use_negative:
- A single study's metabolite list with internally consistent identifiers — no cross-study
  merging or harmonization is needed
- Metabolites that cannot be resolved to standard InChIKeys (neither PubChem Identifier
  Exchange nor CTS translation succeeds), since the merge key is unavailable
- The analysis goal is pathway significance testing on an already-curated dataset
  rather than building a unified, reproducible merged metabolite dataset
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3395
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names to standard InChIKeys (Step 1) and retrieves title
    name, CID, InChI and SMILES per InChIKey (Step 2); re-queried in Step 3b to fill
    missing attributes
- name: Chemical Translation Service (CTS)
  role: Translates non-InChIKey identifiers in Step 1; supplies molecular weight,
    molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs and CAS number in Step
    2 and re-query in Step 3b
- name: Metabolomics Workbench / RefMet
  role: Source of the three urinary asthma studies (ST001039, ST001048, ST001317)
    and of the RefMet accepted name attribute per InChIKey
- name: ClassyFire
  role: Retrieves chemical ontology (kingdom, superclass, class, subclass) using InChI
    as input in Step 2
- name: BinVestigate
  role: Retrieves the Bin ID attribute for each InChIKey in Step 2
- name: Python
  role: Implements the batch translation, merging on InChIKey, attribute retrieval,
    curation steps and validation of row counts before saving merged_metabolites.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:e7ade56cc6bf792749f3801ce733e819298cbcc786a77c02389f41c4b9508d6c
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3395
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# inchikey-based-dataset-merging

## Summary

The Metabolites Merging Strategy (MMS) harmonizes multiple metabolite datasets by merging on standard InChIKeys, retrieving per-metabolite attributes (names, chemical properties, systematic and non-systematic identifiers, ontology), and applying a three-step curation process — enabling interstudy comparability, e.g. the 547→391 urinary asthma metabolite merge.

## When to use

Use when multiple metabolomics datasets with heterogeneous metabolite identifiers (names, HMDB, KEGG, PubChem CIDs, CAS) must be merged and intercompared, and a systematic InChIKey-based harmonization with curation is required.

## When NOT to use

- A single study's metabolite list with internally consistent identifiers — no cross-study merging or harmonization is needed
- Metabolites that cannot be resolved to standard InChIKeys (neither PubChem Identifier Exchange nor CTS translation succeeds), since the merge key is unavailable
- The analysis goal is pathway significance testing on an already-curated dataset rather than building a unified, reproducible merged metabolite dataset

## Inputs

- Metabolite lists from Metabolomics Workbench studies (e.g. ST001039, ST001048, ST001317) filtered by species=human, sample source=urine, disease=asthma
- HMDB entries filtered by urine biospecimen location and asthma-associated disorders
- Metabolite lists from six PubMed literature articles retrieved with keywords 'urine asthma metabolomics'
- Metabolite name or identifier strings requiring translation to standard InChIKeys

## Outputs

- Pre-curation merged dataset on InChIKey with a 'Reference' provenance column (547 rows in the urinary asthma case)
- Final curated merged dataset saved as merged_metabolites.csv (391 metabolite rows) with PubChem title/CID/InChI/SMILES, RefMet name, molecular weight, molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs, CAS number, Bin ID, and ClassyFire ontology (kingdom, superclass, class, subclass)

## How to apply

In Step 1, translate every metabolite name to a standard InChIKey via the PubChem Identifier Exchange Service (fall back to the Chemical Translation Service for non-InChIKey identifiers) and merge all datasets on InChIKey, deduplicating entries while recording each origin in a 'Reference' column — for the urinary asthma case this pre-curation merged table must contain 547 rows. In Step 2, retrieve attributes per InChIKey: PubChem title name, CID, InChI and SMILES; RefMet accepted name from Metabolomics Workbench; molecular weight, molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs and CAS number from CTS; Bin ID from BinVestigate; and kingdom/superclass/class/subclass ontology from ClassyFire using InChI as input. In Step 3, curate in three sub-steps: (3a, optional) collapse conjugated base/acid compounds by comparing the first 14 characters of the 27-character InChIKey (molecular skeleton layer), deleting conjugate entries while preserving their source in the retained compound's 'Reference' column; (3b) re-query PubChem and CTS to refill attributes that failed during batch conversion; (3c) check all bidirectional attributes (RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number, PubChem CID) for non-unique and conflicting synonyms, merging duplicates and keeping provenance. Validate row counts before saving the final dataset (547 pre-merge, 391 final for the asthma case) as merged_metabolites.csv.

## Related tools

- **PubChem Identifier Exchange Service** (Translates metabolite names to standard InChIKeys (Step 1) and retrieves title name, CID, InChI and SMILES per InChIKey (Step 2); re-queried in Step 3b to fill missing attributes)
- **Chemical Translation Service (CTS)** (Translates non-InChIKey identifiers in Step 1; supplies molecular weight, molecular formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs and CAS number in Step 2 and re-query in Step 3b)
- **Metabolomics Workbench / RefMet** (Source of the three urinary asthma studies (ST001039, ST001048, ST001317) and of the RefMet accepted name attribute per InChIKey)
- **ClassyFire** (Retrieves chemical ontology (kingdom, superclass, class, subclass) using InChI as input in Step 2)
- **BinVestigate** (Retrieves the Bin ID attribute for each InChIKey in Step 2)
- **Python** (Implements the batch translation, merging on InChIKey, attribute retrieval, curation steps and validation of row counts before saving merged_metabolites.csv)

## Evaluation signals

- Pre-curation merged table contains exactly 547 metabolite rows for the ten urinary asthma sources
- Final curated merged dataset contains exactly 391 metabolite rows in merged_metabolites.csv
- Every merged entry has a standard 27-character InChIKey; conjugate-form collapsing used the first 14 characters (molecular skeleton layer) consistently
- 'Reference' column retains source provenance for every merged and deleted-conjugate entry
- No duplicate InChIKey entries remain after the bidirectional synonym/conflict check across RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number and PubChem CID

## Limitations

- The conjugated base/acid conversion (Step 3a) is optional and must be decided per dataset; skipping or misapplying it changes final row counts
- Batch identifier/attribute conversion can fail for some entries, requiring a re-query pass (Step 3b) to PubChem and CTS
- Bidirectional identifiers can be non-unique or have conflicting synonyms, so curation decisions (which duplicate to retain) require provenance tracking in the 'Reference' column
- The pipeline depends on availability and stability of external services (PubChem, CTS, RefMet, ClassyFire, BinVestigate)

## Evidence

- [abstract] Step 1 translation and merging: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [abstract] three-step curation: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [abstract] MMS purpose: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [results] 547 to 391 counts: "Following the MMS, a total of 547 metabolites have been retrieved and merged, leading to a final dataset of 391 metabolites."
