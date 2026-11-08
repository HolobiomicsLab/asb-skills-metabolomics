---
name: metabolite-nomenclature-consistency-analysis
description: Use when a merged metabolomics dataset has been assembled (e.g. via InChIKey-based
  MMS) and you need to audit naming consistency, count synonym-hidden merges, and
  report same/different-name statistics.
when_to_use_negative:
- Input is a raw, unmerged study table where the question is which identifiers to
  attach (MMS Step 2 attribute retrieval), not whether already-merged rows agree on
  naming
- Names have already been curated to RefMet/PubChem title names, so a same/different-name
  audit would be trivially 100%/0% and uninformative
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: Python
  role: pandas-based scripting to load the merged and pre-merge datasets, perform
    exact string comparison of reported names against PubChem title names, aggregate
    subgroup statistics, and write naming_statistics.csv
- name: PubChem
  role: source of the canonical 'title' name used as the reference for the same-name
    vs different-name classification
- name: Metabolomics Workbench / RefMet
  role: provides the RefMet name retrieved in MMS Step 2 as an additional naming attribute
    for cross-checking consistency
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:5297d340340781ab17d0ba191805e5f3b0fbc527edb262dc3dbe326f8a8cf6a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Metabolomics Workbench / RefMet
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-nomenclature-consistency-analysis

## Summary

Quantifies how often merged metabolomics entries are reported under their canonical database title name versus a synonym, using exact string comparison of reported names against the PubChem title (and RefMet) names, and derives how many compounds would have gone unmerged without name-merging support.

## When to use

Use when a merged metabolomics dataset has been assembled (e.g. via InChIKey-based MMS) and you need to audit naming consistency, count synonym-hidden merges, and report same/different-name statistics.

## When NOT to use

- Input is a raw, unmerged study table where the question is which identifiers to attach (MMS Step 2 attribute retrieval), not whether already-merged rows agree on naming
- Names have already been curated to RefMet/PubChem title names, so a same/different-name audit would be trivially 100%/0% and uninformative

## Inputs

- merged metabolomics dataset with reported metabolite names plus PubChem 'title' and RefMet names (e.g. Supplementary Table S2, 391 metabolites)
- pre-merge combined dataset of all study entries (e.g. 547 entries) with source-study attribution

## Outputs

- naming_statistics.csv with same/different-name counts and percentages
- counts of compounds that would not have been merged without MMS (e.g. 213 of 372)
- five-subgroup categorization of multi-source compounds (same name unique / same name also in other studies / same name other studies differ / different names other studies correct / different name)

## How to apply

Load the reproduced merged dataset (e.g. 391 urinary asthma metabolites in Supplementary Table S2) and the pre-merge combined dataset (547 entries) with pandas. For each merged entry, compare the reported metabolite name against the PubChem 'title' name using exact string matching so synonyms (e.g. 3,4-dihydroxycinnamic acid vs caffeic acid) and acronyms (e.g. CAR 12:0 vs lauroylcarnitine) fall into the different-name group. Aggregate counts and percentages of same-name vs different-name classes, checking against the reported 179 (46%) vs 212 (54%) split. For compounds found in more than one source, categorize into the five subgroups: 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', and 'different name'. Sum the 'same name (other studies differ)' and 'different names (other studies correct)' subgroups to derive the count that would not have been merged by name alone (reported as 213 of 372). Assemble all counts into naming_statistics.csv and validate that the recomputed 46%/54% split and 213/372 figure reproduce the reported values.

## Related tools

- **Python** (pandas-based scripting to load the merged and pre-merge datasets, perform exact string comparison of reported names against PubChem title names, aggregate subgroup statistics, and write naming_statistics.csv)
- **PubChem** (source of the canonical 'title' name used as the reference for the same-name vs different-name classification)
- **Metabolomics Workbench / RefMet** (provides the RefMet name retrieved in MMS Step 2 as an additional naming attribute for cross-checking consistency)

## Examples

```
import pandas as pd; merged = pd.read_csv('S2_merged.csv'); same = (merged['reported_name'] == merged['pubchem_title']).sum(); stats = pd.DataFrame({'class': ['same name', 'different name'], 'n': [same, len(merged)-same], 'pct': [same/len(merged)*100, (len(merged)-same)/len(merged)*100]}); stats.to_csv('naming_statistics.csv', index=False)
```

## Evaluation signals

- Recomputed counts reproduce the reported 179 same-name (46%) vs 212 different-name (54%) split over the 391-entry merged set
- Summing 'same name (other studies differ)' and 'different names (other studies correct)' subgroups yields the reported 213 of 372 compounds that would not have been merged without MMS
- Every merged entry is labelled exactly 'same name' or 'different name' by exact string match with the PubChem title, with no nulls
- Multi-source compounds are partitioned into exactly the five reported subgroups with no residual category
- naming_statistics.csv contains the full statistics table with counts and percentages for all subgroups

## Limitations

- Exact string matching sends legitimate synonyms and acronyms (e.g. CAR 12:0 vs lauroylcarnitine) to the different-name group, so the statistic measures reported-name divergence, not chemical distinctness
- The 46%/54% and 213/372 figures depend on the merged 391-metabolite urinary asthma dataset and its 547-entry pre-merge source; other datasets will give different fractions
- Classification relies on the PubChem title name being correctly retrieved during MMS Step 2; missing or mismatched title attributes propagate errors into the statistics

## Evidence

- [results] six out of the ten most prevalent metabolites are named differently in the merged dataset: [evidence span withheld — anchor status: not_found]
- [abstract] MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
- [methods] Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench): "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [methods] three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information): "three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [abstract] This study highlights the need for standardized and unified metabolite datasets to enhance the reproducibility and comparability of metabolomics studies.: "This study highlights the need for standardized and unified metabolite datasets to enhance the reproducibility and comparability of metabolomics studies."
