---
name: cross-study-metabolite-overlap-classification
description: Use when auditing a merged multi-study metabolite dataset to report the
  same-name vs different-name split against PubChem title names and to quantify unmerged
  compounds without the MMS.
when_to_use_negative:
- The datasets are still unmerged — merging by InChIKeys (MMS Step 1) must be performed
  first; this skill only audits the merged set
- The analysis goal is pathway significance testing rather than name-consistency and
  merge-need quantification
- No PubChem title name (or equivalent reference naming attribute) is available for
  the merged entries, making exact string comparison impossible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python (pandas)
  role: Loads the merged (391-entry) and pre-merge (547-entry) datasets, performs
    exact string comparison of reported metabolite names against PubChem title names,
    aggregates the five naming subgroups, and writes naming_statistics.csv
- name: PubChem
  role: Source of the 'title' reference name used as the ground-truth label when classifying
    each merged metabolite as same-name or different-name
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:d10b0e5c53efefbd9e139e80ccba3b029fe5c9f87abc2b4bca38e7f92a70eece
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem
  - Python
  - Python (pandas)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-study-metabolite-overlap-classification

## Summary

Quantifies how metabolites shared across studies in an MMS-merged urinary asthma metabolomics dataset are reported: whether the reported name matches the PubChem title name, and how many compounds would have been missed if datasets were merged by name alone instead of InChIKeys.

## When to use

Use when auditing a merged multi-study metabolite dataset to report the same-name vs different-name split against PubChem title names and to quantify unmerged compounds without the MMS.

## When NOT to use

- The datasets are still unmerged — merging by InChIKeys (MMS Step 1) must be performed first; this skill only audits the merged set
- The analysis goal is pathway significance testing rather than name-consistency and merge-need quantification
- No PubChem title name (or equivalent reference naming attribute) is available for the merged entries, making exact string comparison impossible

## Inputs

- Reproduced merged dataset of 391 urinary asthma metabolites (Supplementary Table S2) with reported metabolite names and PubChem title names
- Pre-merge combined dataset of 547 metabolite entries from the constituent studies
- PubChem title names retrieved via the InChIKey attribute-retrieval step of the MMS

## Outputs

- naming_statistics.csv with counts and percentages of same-name (179, 46%) vs different-name (212, 54%) metabolites
- Per-metabolite labels assigning each merged entry to one of the five reported naming subgroups
- Count of compounds that would not have been merged without the MMS (213 of 372)

## How to apply

Load the reproduced merged dataset (391 metabolites, Supplementary Table S2) and the pre-merge combined dataset (547 entries) with pandas. For each merged entry, compare the reported metabolite name against the PubChem 'title' name by exact string matching, labeling entries as 'same name' or 'different name' so synonyms (e.g., 3,4-dihydroxycinnamic acid vs caffeic acid) and acronyms (e.g., CAR 12:0 vs lauroylcarnitine) fall into the different-name group. Aggregate counts and percentages of the two classes, then subdivide multi-study compounds into the five reported subgroups: 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', and 'different name'. Sum 'same name (other studies differ)' plus 'different name (other studies correct)' to derive compounds that would not have been merged by name alone (reported as 213 of 372), and write all counts and percentages to naming_statistics.csv.

## Related tools

- **Python (pandas)** (Loads the merged (391-entry) and pre-merge (547-entry) datasets, performs exact string comparison of reported metabolite names against PubChem title names, aggregates the five naming subgroups, and writes naming_statistics.csv)
- **PubChem** (Source of the 'title' reference name used as the ground-truth label when classifying each merged metabolite as same-name or different-name)

## Examples

```
df = pd.read_csv('Supplementary_Table_S2.csv'); df['name_match'] = df['metabolite_name'] == df['pubchem_title']; df.to_csv('naming_statistics.csv')
```

## Evaluation signals

- Recomputed split over the 391-entry merged set must equal the reported 179 (46%) same-name vs 212 (54%) different-name
- Sum of the five naming subgroups must equal 372 for multi-study compounds, and 'same name (other studies differ)' + 'different name (other studies correct)' must equal 213
- Every merged entry must carry exactly one of the five subgroup labels (no missing or double assignments)
- Percentage columns in naming_statistics.csv must sum to 100% across the two same/different classes
- Synonym pairs such as 3,4-dihydroxycinnamic acid vs caffeic acid and CAR 12:0 vs lauroylcarnitine must be counted in the different-name group under exact string matching

## Limitations

- Exact string matching is deliberately strict: synonyms and acronyms are counted as different names, which is the intended behavior for quantifying merge difficulty but would be wrong for semantic identity judgments
- The statistics describe the urinary asthma merged dataset (391 metabolites / 372 multi-study compounds); counts will differ for other study collections
- Results depend on the correctness of the upstream MMS merging by InChIKeys and the attribute-retrieval step that supplies PubChem title names

## Evidence

- [abstract] MMS framework quote: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [abstract] Attribute retrieval quote: "Step 2: Attributes' retrieval from the InChIkey, including descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [abstract] Standardization rationale quote: "This study highlights the need for standardized and unified metabolite datasets to enhance the reproducibility and comparability"
