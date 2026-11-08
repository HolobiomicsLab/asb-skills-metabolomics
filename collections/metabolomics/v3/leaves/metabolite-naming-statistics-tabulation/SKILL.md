---
name: metabolite-naming-statistics-tabulation
description: Use when a merged metabolite dataset must be audited for name consistency
  (e.g. how many entries keep the PubChem title name, how many differ, and how many
  would not have merged by name alone), to justify or validate a metabolite merging
  strategy.
when_to_use_negative:
- Input is a single-study metabolite list with no merging performed — there are no
  cross-study name conflicts to tabulate
- The dataset already carries canonicalized names (e.g. all entries normalized to
  RefMet) and you need compound merging, not a naming-consistency audit
- Names in the dataset are not comparable to PubChem title or RefMet names (e.g. only
  structural identifiers like InChIKeys are present)
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: pandas
  role: Loads the merged and pre-merge datasets and performs the string comparisons,
    grouping, and percentage aggregation behind the naming statistics
- name: PubChem
  role: Supplies the canonical 'title' name that each merged entry's reported name
    is compared against
- name: RefMet
  role: Canonical reference name from Metabolomics Workbench used as an alternative
    naming standard during attribute retrieval
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:52953aec48368ac30bf455d7d370e08387fab943a3b9fb9f46ced9d39ac6495d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem
  - Python
  - RefMet
  - pandas
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolite-naming-statistics-tabulation

## Summary

Quantifies the consistency of the metabolite names reported in a merged metabolomics dataset against canonical database titles (PubChem title, RefMet), tabulating same/different-name splits and the number of compounds that would go unmerged without a merging strategy.

## When to use

Use when a merged metabolite dataset must be audited for name consistency (e.g. how many entries keep the PubChem title name, how many differ, and how many would not have merged by name alone), to justify or validate a metabolite merging strategy.

## When NOT to use

- Input is a single-study metabolite list with no merging performed — there are no cross-study name conflicts to tabulate
- The dataset already carries canonicalized names (e.g. all entries normalized to RefMet) and you need compound merging, not a naming-consistency audit
- Names in the dataset are not comparable to PubChem title or RefMet names (e.g. only structural identifiers like InChIKeys are present)

## Inputs

- Reproduced merged metabolite dataset with 391 entries including reported metabolite name and PubChem title name (Supplementary Table S2)
- Pre-merge combined dataset of 547 metabolite entries from multiple studies
- Per-compound source provenance (which studies report each compound) to identify multi-source compounds

## Outputs

- naming_statistics.csv: counts and percentages of same-name vs different-name entries
- Subgroup counts for 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', 'different name'
- Count of compounds that would not have been merged without the merging strategy (e.g. 213 of 372)

## How to apply

Load the reproduced merged dataset (391 metabolites, Supplementary Table S2) and the pre-merge combined dataset (547 entries) with pandas. Compare each merged entry's reported metabolite name against the PubChem 'title' name using exact string matching, so synonyms (e.g. 3,4-dihydroxycinnamic acid vs caffeic acid) and acronyms (e.g. CAR 12:0 vs lauroylcarnitine) count as different names. Label each entry 'same name' or 'different name', aggregate counts and percentages, and verify against the reported split of 179 (46%) same vs 212 (54%) different. For compounds found in more than one source, further categorize into the five subgroups — 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', and 'different name'. Sum the 'same name (other studies differ)' and 'different names (other studies correct)' subgroups to derive the count of compounds that would not have been merged by name alone, checked against the reported 213 of 372. Assemble all counts and percentages into a statistics table saved as naming_statistics.csv.

## Related tools

- **pandas** (Loads the merged and pre-merge datasets and performs the string comparisons, grouping, and percentage aggregation behind the naming statistics)
- **PubChem** (Supplies the canonical 'title' name that each merged entry's reported name is compared against)
- **RefMet** (Canonical reference name from Metabolomics Workbench used as an alternative naming standard during attribute retrieval)

## Examples

```
import pandas as pd; merged = pd.read_csv('merged_dataset.csv'); stats = (merged['reported_name'] == merged['pubchem_title']).value_counts(); stats.to_csv('naming_statistics.csv')
```

## Evaluation signals

- Recomputed same/different-name split over the 391-entry merged set must equal the reported 179 (46%) same vs 212 (54%) different
- Sum of 'same name (other studies differ)' + 'different names (other studies correct)' must equal the reported 213 of 372 compounds unmerged without MMS
- Percentage columns in naming_statistics.csv must sum to 100% within rounding error
- Every merged entry must be assigned exactly one of the two name classes and, for multi-source compounds, exactly one of the five subgroups
- Exact string match must flag synonym and acronym cases (e.g. 3,4-dihydroxycinnamic acid vs caffeic acid) as different names, not same

## Limitations

- Exact string matching means synonyms and acronyms are counted as 'different name' even when they refer to the same compound — this is intended, but the interpretation is sensitive to this convention
- Statistics depend on the PubChem title and RefMet names retrieved during the MMS attribute-retrieval step; missing or stale attribute retrieval will distort the split
- Counts apply to the reproduced merged dataset (391 metabolites) and pre-merge set (547 entries); other datasets will yield different baselines and cannot be compared directly

## Evidence

- [abstract] Step 1: translation and merging of datasets by employing InChIKeys for data integration, including translation of metabolite names if needed: "Step 1: Translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
- [abstract] RefMet and PubChem titles as naming standards: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [abstract] Synonym checking as a curation step: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [abstract] The study highlights the need for standardized and unified metabolite datasets: "This study highlights the need for standardized and unified metabolite datasets to enhance the reproducibility and comparability of metabolomics studies."
