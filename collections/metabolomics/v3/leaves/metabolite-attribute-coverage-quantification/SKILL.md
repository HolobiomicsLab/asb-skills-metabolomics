---
name: metabolite-attribute-coverage-quantification
description: Use when you have a curated merged metabolite table (e.g. 391 urinary
  asthma metabolites from the MMS pipeline) and need to report how completely each
  non-systematic identifier column is populated before or after attribute retrieval
  and curation.
when_to_use_negative:
- The input is a raw, unmerged per-study metabolite list that has not yet passed through
  InChIKey-based merging (MMS Step 1) and attribute retrieval (Step 2) — coverage
  quantification on pre-merged data will not reflect identifier harmonization
- You need systematic identifier coverage (e.g. InChIKey completeness) rather than
  non-systematic identifier coverage, which is a different attribute class in the
  MMS framework
- The goal is pathway-level interpretation of merged urinary asthma metabolites, not
  annotation completeness reporting
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python
  role: Used with pandas to load the merged dataset, count non-empty identifier entries
    per attribute column, and compute coverage percentages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:70646b78c32e633f12df3a32f491bc88a14fc446e7d8140c13ef500e47739bfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# metabolite-attribute-coverage-quantification

## Summary

Quantifies the proportion of a merged metabolite dataset annotated with each non-systematic identifier (CAS, RefMet, ChEBI, HMDB, KEGG, BinVestigate, DrugBank, LipidMaps), reporting per-attribute fill rates as percentages. It serves as a completeness audit of the attribute-retrieval step of the Metabolites Merging Strategy (MMS).

## When to use

Use when you have a curated merged metabolite table (e.g. 391 urinary asthma metabolites from the MMS pipeline) and need to report how completely each non-systematic identifier column is populated before or after attribute retrieval and curation.

## When NOT to use

- The input is a raw, unmerged per-study metabolite list that has not yet passed through InChIKey-based merging (MMS Step 1) and attribute retrieval (Step 2) — coverage quantification on pre-merged data will not reflect identifier harmonization
- You need systematic identifier coverage (e.g. InChIKey completeness) rather than non-systematic identifier coverage, which is a different attribute class in the MMS framework
- The goal is pathway-level interpretation of merged urinary asthma metabolites, not annotation completeness reporting

## Inputs

- Curated merged metabolite dataset from the MMS case study output file (391 entries, with columns for CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, LipidMaps ID)

## Outputs

- coverage_table.csv with columns attribute, n_filled, percentage
- Validation flags for any attribute whose computed coverage deviates by more than 1 percentage point from the reported values

## How to apply

Load the curated merged dataset (391 entries from the MMS case study output) into a pandas DataFrame. For each of the eight non-systematic identifier attribute columns — CAS number, RefMet accepted name, ChEBI ID, HMDB ID, KEGG ID, BinVestigate ID, DrugBank ID, and LipidMaps ID — count non-empty entries and divide by the total merged entries (391), multiplying by 100 to obtain coverage percentages. Assemble the results into a table with columns attribute, n_filled, percentage and write it to coverage_table.csv. Validate each computed percentage against reported reference values (CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21%), flagging any attribute deviating by more than 1 percentage point. This quantification exposes which identifier sources the InChIKey-driven attribute retrieval (Step 2 of MMS) filled and where the three-step curation process (Step 3) closed gaps.

## Related tools

- **Python** (Used with pandas to load the merged dataset, count non-empty identifier entries per attribute column, and compute coverage percentages)

## Examples

```
import pandas as pd; df = pd.read_csv('mms_merged_391.csv'); attrs = ['CAS number','RefMet accepted name','ChEBI ID','HMDB ID','KEGG ID','BinVestigate ID','DrugBank ID','LipidMaps ID']; pct = {a: round(100*df[a].notna().sum()/len(df),0) for a in attrs}
```

## Evaluation signals

- Computed coverage percentages match the reported values exactly: CAS 99%, RefMet 95%, ChEBI 88%, HMDB 87%, KEGG 83%, BinVestigate 42%, DrugBank 42%, LipidMaps 21%
- No attribute deviates by more than 1 percentage point from the reported value; deviations are flagged
- coverage_table.csv contains exactly eight rows (one per non-systematic identifier attribute) with the schema attribute, n_filled, percentage
- n_filled for each attribute is an integer between 0 and 391, and percentage equals 100 * n_filled / 391

## Limitations

- Coverage depends on the quality of the upstream InChIKey-based merging and attribute retrieval; sparse identifier columns (e.g. LipidMaps at 21%) reflect limits of attribute retrieval, not necessarily of the underlying metabolites
- The validation benchmark assumes the specific 391-entry merged urinary asthma dataset; applying it to other merged datasets requires recalculating expected percentages rather than reusing the reported values

## Evidence

- [abstract] non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number): "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
- [full_text] Step 2: Attributes' retrieval from the InChIkey, including descriptors of name and chemical properties, systematic and non-systematic identifiers, and ontology: [evidence span withheld — anchor status: not_found]
- [full_text] a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking"
