---
name: synonym-duplicate-detection-curation
description: Use when merging metabolite datasets by InChIKey and some entries share
  the same molecular skeleton but carry conflicting synonyms or duplicate IDs (HMDB,
  KEGG, ChEBI, CAS, Bin ID, PubChem CID), requiring reconciliation before pathway
  analysis.
when_to_use_negative:
- When only a single metabolite dataset is under study with no inter-study comparison
  — synonym merging adds little value and the article's motivation is harmonization
  across studies
- When entries differ in the first 14 InChIKey skeleton characters (different molecular
  skeletons) — those are distinct compounds and belong to Step 3a conjugate handling,
  not synonym curation
- Before Step 2 attribute retrieval — curation of conflicting identifiers cannot be
  assessed if PubChem/CTS attributes have not yet been populated
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: PubChem Identifier Exchange Service
  role: Translates metabolite names to InChIKeys and retrieves title names and CIDs
    used to detect synonym duplicates
- name: Chemical Translation Service (CTS)
  role: Provides HMDB, KEGG, LipidMaps, DrugBank, ChEBI, and CAS identifiers whose
    conflicts drive synonym curation
- name: RefMet
  role: Supplies the accepted metabolite name used to reconcile synonym variants across
    studies
- name: BinVestigate
  role: Source of Bin ID checked for non-unique or conflicting values during curation
- name: ClassyFire
  role: Provides the ontology (kingdom, superclass, class, subclass) carried alongside
    curated entries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:9008eeccbb7852f47c3ce14f59695f0f508027dd9506ca5b16316d8483126156
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# synonym-duplicate-detection-curation

## Summary

Curation step (MMS Step 3c) that merges duplicate metabolite entries created by synonyms and conflicting cross-database identifiers after InChIKey-based merging, preserving each source in a Reference column to enable urinary asthma metabolite inter-study comparability.

## When to use

Use when merging metabolite datasets by InChIKey and some entries share the same molecular skeleton but carry conflicting synonyms or duplicate IDs (HMDB, KEGG, ChEBI, CAS, Bin ID, PubChem CID), requiring reconciliation before pathway analysis.

## When NOT to use

- When only a single metabolite dataset is under study with no inter-study comparison — synonym merging adds little value and the article's motivation is harmonization across studies
- When entries differ in the first 14 InChIKey skeleton characters (different molecular skeletons) — those are distinct compounds and belong to Step 3a conjugate handling, not synonym curation
- Before Step 2 attribute retrieval — curation of conflicting identifiers cannot be assessed if PubChem/CTS attributes have not yet been populated

## Inputs

- InChIKey-keyed merged metabolite table with attributes per entry: PubChem title name and CID, RefMet accepted name, molecular weight, molecular formula, InChI, SMILES, ChEBI ID, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, CAS number, Bin ID, and ClassyFire ontology (kingdom, superclass, class, subclass)
- Source provenance per entry (which Metabolomics Workbench study, HMDB, or literature article contributed it) in a 'Reference' column

## Outputs

- Final curated merged metabolite dataset (merged_metabolites.csv) with duplicate/synonym entries collapsed to one row per unique compound, retaining combined source references
- Reconciled non-systematic identifiers (HMDB, KEGG, ChEBI, LipidMaps, DrugBank, CAS, Bin ID, PubChem CID) consistent across the merged table

## How to apply

After translating all metabolite names to full 27-character InChIKeys and retrieving attributes (PubChem title name and CID, RefMet accepted name, molecular weight and formula, ChEBI/HMDB/KEGG/LipidMaps/DrugBank IDs, CAS number, Bin ID from BinVestigate, and ClassyFire ontology), inspect each InChIKey's bidirectional attributes for non-unique or conflicting synonyms. Where multiple rows share an InChIKey or where a compound's RefMet name, HMDB ID, KEGG ID, LipidMaps ID, DrugBank ID, ChEBI ID, CAS number, or PubChem CID disagree, merge them into a single row and record every contributing source in the 'Reference' column rather than discarding provenance. This synonym/duplicate curation (Step 3c) follows the optional conjugated base/acid conversion (Step 3a, matching the first 14 skeleton-layer InChIKey characters) and the missing-attribute refill (Step 3b, re-querying PubChem and CTS). The rationale is that synonym and identifier duplication inflates metabolite counts and masks real overlaps between studies, so reconciling them is what exposes significant pathways hidden when no merging strategy is applied.

## Related tools

- **PubChem Identifier Exchange Service** (Translates metabolite names to InChIKeys and retrieves title names and CIDs used to detect synonym duplicates)
- **Chemical Translation Service (CTS)** (Provides HMDB, KEGG, LipidMaps, DrugBank, ChEBI, and CAS identifiers whose conflicts drive synonym curation)
- **RefMet** (Supplies the accepted metabolite name used to reconcile synonym variants across studies)
- **BinVestigate** (Source of Bin ID checked for non-unique or conflicting values during curation)
- **ClassyFire** (Provides the ontology (kingdom, superclass, class, subclass) carried alongside curated entries)

## Examples

```
python merge_mms.py --input step2_attributes.csv --reference-column Reference --curation step3c --output merged_metabolites.csv
```

## Evaluation signals

- The pre-curation merged table contains exactly 547 metabolite rows after Step 1 merging of the ten urinary asthma sources
- The final curated merged dataset contains exactly 391 metabolite rows, a reduction attributable to conjugate conversion plus synonym/duplicate curation
- Every retained row preserves the full set of contributing source names in the 'Reference' column — no provenance lost during merging
- No row has non-unique or conflicting values across the bidirectional attribute set (RefMet name, HMDB ID, Bin ID, DrugBank ID, KEGG ID, LipidMaps ID, ChEBI ID, CAS number, PubChem CID)
- Significant pathways are recovered that were not flagged when the datasets were analyzed without merging, as reported for the urinary asthma case

## Limitations

- Synonym curation depends on the completeness of Step 2 attribute retrieval; compounds whose PubChem or CTS queries fail cannot be reliably reconciled and may require the Step 3b refill pass
- Conjugated base/acid compounds require the optional Step 3a skeleton comparison (first 14 InChIKey characters) before synonym logic applies; skipping it can leave conjugate duplicates behind
- Merging is only as consistent as the external databases themselves — if RefMet, HMDB, or KEGG list conflicting synonyms for the same metabolite, manual judgement is still needed

## Evidence

- [abstract] Step 3c: check all bidirectional attributes for non-unique and conflicting synonyms, merging duplicate entries and keeping source provenance: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [abstract] Step 2 retrieves the identifiers that Step 3c must reconcile: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench), and chemical properties (molecular weight and"
- [other] Duplicate entries retain their sources in the Reference column rather than being dropped: [evidence span withheld — anchor status: not_found]
- [abstract] Curated datasets expose pathways hidden without merging: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
- [results] The asthma case study produced a concrete pre-merge count of 547 metabolites: "Following the MMS, a total of 547 metabolites have been retrieved and merged, leading to a final dataset of 391 metabolites."
