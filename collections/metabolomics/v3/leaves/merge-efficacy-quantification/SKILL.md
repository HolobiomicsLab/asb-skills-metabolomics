---
name: merge-efficacy-quantification
description: Use when you have just applied a metabolite-merging strategy (e.g. MMS)
  to a combined multi-study dataset and need to report the merge's efficacy — counts
  of name-consistency and of compounds that would not have merged by name alone.
when_to_use_negative:
- You have not yet run the merge step (e.g. MMS step 1) — quantifying merge efficacy
  before the merged dataset exists is meaningless.
- Your dataset has no independent canonical identifier (e.g. PubChem 'title' name
  or InChIKey) to compare reported names against — exact string matching alone cannot
  distinguish synonyms from genuinely different metabolites.
- You need pathway-level or biological-signature results, not merge-quality statistics
  — MMS efficacy quantification does not test pathway enrichment outcomes.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python
  role: Used to load the merged and pre-merge datasets with pandas, perform the exact
    string comparisons between reported metabolite names and PubChem title names,
    aggregate the counts, and write the resulting naming_statistics.csv.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:bac0239992574e0a2eb4f8c542eedb88352fd070586195c5f5282c3269abef90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Merge efficacy quantification

## Summary

Quantifies how much value a metabolite-merging strategy adds by measuring how many metabolites share their reported name versus the canonical (PubChem title) name after merging, and how many compounds would have been missed without the merge (e.g. 179 vs 212 of 391 metabolites; 213 of 372 unmerged without MMS).

## When to use

Use when you have just applied a metabolite-merging strategy (e.g. MMS) to a combined multi-study dataset and need to report the merge's efficacy — counts of name-consistency and of compounds that would not have merged by name alone.

## When NOT to use

- You have not yet run the merge step (e.g. MMS step 1) — quantifying merge efficacy before the merged dataset exists is meaningless.
- Your dataset has no independent canonical identifier (e.g. PubChem 'title' name or InChIKey) to compare reported names against — exact string matching alone cannot distinguish synonyms from genuinely different metabolites.
- You need pathway-level or biological-signature results, not merge-quality statistics — MMS efficacy quantification does not test pathway enrichment outcomes.

## Inputs

- Reproduced merged metabolite dataset (e.g. Supplementary Table S2, 391 metabolites) with reported metabolite name and PubChem 'title' columns
- Pre-merge combined dataset of all source studies (547 entries) for the unmerged-by-name-alone comparison

## Outputs

- naming_statistics.csv — counts and percentages of same-name vs different-name metabolites and of the five subgroup categories over the merged set
- Count of compounds that would not have been merged by name alone (e.g. 213 of 372)

## How to apply

Load the reproduced merged dataset (391 metabolites, e.g. Supplementary Table S2) and the pre-merge combined dataset (547 entries) with pandas. For each merged entry, compare the reported metabolite name against the PubChem 'title' name by exact string matching so synonyms (3,4-dihydroxycinnamic acid vs caffeic acid) and acronyms (CAR 12:0 vs lauroylcarnitine) fall into the 'different name' group. Categorize compounds found in more than one source into five subgroups — 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', 'different name' — then aggregate counts and percentages (target: 179 (46%) same vs 212 (54%) different over 391 entries). Sum the 'same name (other studies differ)' and 'different name (other studies correct)' subgroups to derive the count of compounds that would not have been merged by name alone (target: 213 of 372). Write the resulting table as naming_statistics.csv.

## Related tools

- **Python** (Used to load the merged and pre-merge datasets with pandas, perform the exact string comparisons between reported metabolite names and PubChem title names, aggregate the counts, and write the resulting naming_statistics.csv.)

## Examples

```
python quantify_merge_efficacy.py --merged SupplementaryTableS2.csv --pre_merge combined_547.csv --out naming_statistics.csv
```

## Evaluation signals

- Recomputed counts reproduce the reported 179 (46%) same-name vs 212 (54%) different-name split over the 391-entry merged dataset.
- Sum of the 'same name (other studies differ)' and 'different name (other studies correct)' subgroups equals the reported 213-of-372 compounds that would not have been merged without MMS.
- Output file naming_statistics.csv is present and contains both percentage and absolute count columns for each category.
- Synonyms and acronyms (e.g. 3,4-dihydroxycinnamic acid vs caffeic acid, CAR 12:0 vs lauroylcarnitine) are classified into the different-name group, confirming exact string matching is being used.
- Subgroup labels exactly match the five reported categories: 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', and 'different name'.

## Limitations

- Exact string matching is used, so metabolites reported under synonyms or acronyms are always counted as different-name even when they are chemically identical; the curation steps of MMS (synonym checking) must already have resolved duplicates upstream.
- The efficacy figures depend on having followed the full MMS pipeline (including curation and attribute retrieval); skipping curation can alter the 46%/54% split and the unmerged count.
- The reported split (46% vs 54%) is tied to the specific urinary asthma metabolite dataset; other datasets may yield different proportions, so treat the numbers as dataset-specific rather than universal merge-efficacy benchmarks.

## Evidence

- [abstract] we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [abstract] descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench): "descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [abstract] Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed): "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
