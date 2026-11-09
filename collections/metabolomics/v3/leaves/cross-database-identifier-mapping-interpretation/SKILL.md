---
name: cross-database-identifier-mapping-interpretation
description: Use when you have a merged/harmonized metabolite table (e.g. the 391-entry
  urinary asthma metabolite MMS output) and need to report or audit what percentage
  of entries carry each non-systematic identifier before interpreting or comparing
  annotations across databases.
when_to_use_negative:
- The table is a raw per-study metabolite list that has not yet gone through InChIKey-based
  merging and the three-step MMS curation process — coverage on uncurated data will
  not match the merged-dataset benchmarks
- You need to obtain/retrieve the identifiers themselves (MMS Step 2 attribute retrieval)
  rather than quantify the coverage of identifier columns that already exist
- The analysis concerns systematic identifiers (e.g. InChIKey) or chemical property
  descriptors rather than the eight non-systematic identifier attributes
edam_operation: http://edamontology.org/operation_0226
edam_topics: []
tools:
- name: Python
  role: pandas-based counting of non-empty identifier columns and computation of per-attribute
    coverage percentages for the merged dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:a8d6d0db79347bab83466c7f3090e8a56ac394df3efdb449b924185e25bb962b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-database-identifier-mapping-interpretation

## Summary

Quantifies how completely each non-systematic metabolite identifier (CAS, RefMet accepted name, ChEBI, HMDB, KEGG, BinVestigate, DrugBank, LipidMaps) annotates a merged metabolite dataset, following Step 2 (attribute retrieval from the InChIKey) of the Metabolites Merging Strategy (MMS). It turns identifier columns into interpretable coverage percentages that reveal which databases can realistically support downstream interstudy comparison.

## When to use

Use when you have a merged/harmonized metabolite table (e.g. the 391-entry urinary asthma metabolite MMS output) and need to report or audit what percentage of entries carry each non-systematic identifier before interpreting or comparing annotations across databases.

## When NOT to use

- The table is a raw per-study metabolite list that has not yet gone through InChIKey-based merging and the three-step MMS curation process — coverage on uncurated data will not match the merged-dataset benchmarks
- You need to obtain/retrieve the identifiers themselves (MMS Step 2 attribute retrieval) rather than quantify the coverage of identifier columns that already exist
- The analysis concerns systematic identifiers (e.g. InChIKey) or chemical property descriptors rather than the eight non-systematic identifier attributes

## Inputs

- Curated merged metabolite dataset (MMS case study output, 391 entries) with columns for CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, and LipidMaps ID

## Outputs

- coverage_table.csv with columns attribute, n_filled, percentage for the eight non-systematic identifier attributes
- Validation report flagging any attribute whose computed coverage deviates >1 percentage point from the reported values (CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21%)

## How to apply

Load the curated merged metabolite dataset (391 entries) from the MMS case study output file into a pandas DataFrame. For each of the eight non-systematic identifier attribute columns — CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, LipidMaps ID — count non-empty entries, divide by 391, and multiply by 100 to obtain coverage percentages. These attributes are exactly the Step 2 retrievals MMS obtains from the InChIKey, complementing systematic identifiers and name/property descriptors from PubChem and Metabolomics Workbench. Assemble the results into a table with columns attribute, n_filled, percentage and write it to coverage_table.csv. Validate against the reported benchmark values — CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21% — flagging any attribute whose computed percentage deviates by more than 1 percentage point, and interpret low-coverage attributes (e.g. LipidMaps at 21%) as databases that cannot support pathway or intercomparison analyses alone.

## Related tools

- **Python** (pandas-based counting of non-empty identifier columns and computation of per-attribute coverage percentages for the merged dataset)

## Examples

```
df = pd.read_csv('merged_dataset.csv'); attrs = ['CAS number','RefMet accepted name','ChEBI ID','HMDB ID','KEGG ID','BinVestigate ID','DrugBank ID','LipidMaps ID']; pct = df[attrs].notna().sum() / 391 * 100; pct.to_csv('coverage_table.csv')
```

## Evaluation signals

- Computed percentages match the reported values: CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21%
- No attribute deviates by more than 1 percentage point from its Discussion-reported value; any deviation is explicitly flagged
- n_filled counts are non-negative integers not exceeding 391 (the total merged entries), and percentages = 100 * n_filled / 391
- coverage_table.csv contains exactly eight rows, one per non-systematic identifier attribute, with columns attribute, n_filled, percentage
- Each identifier column is checked for non-empty (non-null, non-blank) entries consistently across all eight attributes

## Limitations

- Coverage is dataset-specific: percentages reflect the merged 391-metabolite urinary asthma dataset and will differ for other study sets
- BinVestigate, DrugBank, and LipidMaps coverage is low (42%, 42%, 21%), so these identifiers alone cannot support comprehensive downstream analysis of this dataset
- Coverage quantification depends on MMS's upstream three-step curation (conjugated base/acid rectification, missing-attribute retrieval, synonym checking); applying it to uncurated merges will undercount true coverage

## Evidence

- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [abstract] InChIKey attribute retrieval defines which identifier columns exist: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [discussion] Reported coverage values for the merged dataset: [evidence span withheld — anchor status: not_found]
- [abstract] Three-step curation precedes coverage assessment: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
