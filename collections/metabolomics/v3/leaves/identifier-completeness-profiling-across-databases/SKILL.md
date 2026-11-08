---
name: identifier-completeness-profiling-across-databases
description: 'Use when a merged metabolomics dataset needs its identifier coverage
  audited: you must report, per attribute, the proportion of entries with a non-empty
  non-systematic identifier and check it against published coverage values.'
when_to_use_negative:
- The dataset is a raw, unmerged study table rather than the InChIKey-merged MMS output
  — merging via InChIKeys (Step 1) must precede coverage profiling
- The goal is to retrieve or fill missing identifiers (MMS Step 2 attribute retrieval
  / Step 3 curation) rather than to measure how complete those attributes already
  are
- The analysis targets systematic identifiers (e.g. InChIKeys, PubChem CIDs used as
  merge keys) instead of the eight non-systematic identifier attributes
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Python
  role: Used with pandas to count non-empty identifier entries per attribute column
    and compute coverage percentages over the 391 merged entries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:8f1a4f6c6bf10c6658c95a00bb2559f9801c5878174df2c62695c6e442491627
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# identifier-completeness-profiling-across-databases

## Summary

A method to quantify, per non-systematic identifier attribute (CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, LipidMaps ID), what percentage of a merged metabolite dataset carries a filled value — here the 391-metabolite urinary asthma dataset produced by the Metabolites Merging Strategy (MMS).

## When to use

Use when a merged metabolomics dataset needs its identifier coverage audited: you must report, per attribute, the proportion of entries with a non-empty non-systematic identifier and check it against published coverage values.

## When NOT to use

- The dataset is a raw, unmerged study table rather than the InChIKey-merged MMS output — merging via InChIKeys (Step 1) must precede coverage profiling
- The goal is to retrieve or fill missing identifiers (MMS Step 2 attribute retrieval / Step 3 curation) rather than to measure how complete those attributes already are
- The analysis targets systematic identifiers (e.g. InChIKeys, PubChem CIDs used as merge keys) instead of the eight non-systematic identifier attributes

## Inputs

- Curated merged metabolite dataset from the MMS case study (391 entries, e.g. CSV) with columns for CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, LipidMaps ID
- Optional: reported coverage percentages from the Discussion for validation

## Outputs

- coverage_table.csv with columns attribute, n_filled, percentage (percentage of 391 merged entries filled per identifier attribute)

## How to apply

Load the curated merged metabolite dataset (391 entries) from the MMS case study output file with pandas. For each of the eight non-systematic identifier columns (CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, LipidMaps ID), count non-empty entries, divide by 391, and multiply by 100. Assemble the results into a table with columns attribute, n_filled, percentage and write coverage_table.csv. Validate by comparing to the reported values — CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21% — flagging any attribute whose computed percentage deviates by more than 1 percentage point. Rationale: these attributes are retrieved in MMS Step 2 from the InChIKey, so coverage percentages are a direct audit of how completely Step 2 attribute retrieval populated the merged table before the Step 3 curation process.

## Related tools

- **Python** (Used with pandas to count non-empty identifier entries per attribute column and compute coverage percentages over the 391 merged entries)

## Examples

```
import pandas as pd; df = pd.read_csv('merged_dataset.csv'); attrs = ['CAS number','RefMet accepted name','ChEBI ID','HMDB ID','KEGG ID','BinVestigate ID','DrugBank ID','LipidMaps ID']; cov = df[attrs].notna().mean().mul(100).round(0)
```

## Evaluation signals

- Computed percentages match the reported values exactly: CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21%
- No attribute deviates by more than 1 percentage point from the reported value (deviations are flagged)
- coverage_table.csv contains exactly eight attribute rows and has the schema attribute, n_filled, percentage
- Sum consistency check: each n_filled equals count of non-empty cells in the corresponding column of the 391-row merged dataset

## Limitations

- Coverage reflects only the eight non-systematic identifier attributes; PubChem title names and InChIKeys used for merging are assessed elsewhere
- Percentages depend on the Step 2 attribute retrieval from the InChIKey and the Step 3 curation process; gaps upstream (e.g. missing attributes not rectified) propagate into low coverage values such as LipidMaps at 21%

## Evidence

- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [abstract] MMS framework: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [abstract] attribute retrieval: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
