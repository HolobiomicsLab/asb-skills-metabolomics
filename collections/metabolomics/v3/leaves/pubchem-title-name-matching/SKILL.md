---
name: pubchem-title-name-matching
description: Use when a merged metabolomics dataset (e.g., 391 urinary asthma metabolites)
  has been assembled via InChIKeys and you must quantify name consistency against
  PubChem title names or estimate name-only merge failures.
when_to_use_negative:
- The goal is to perform the merge itself via InChIKeys (MMS Step 1) rather than audit
  name consistency of an already-merged dataset
- The dataset has no PubChem title names or no retrieved name attributes from MMS
  Step 2, so exact string comparison against a reference name is impossible
- You need fuzzy/synonym-aware matching; this audit intentionally uses exact string
  matching so synonyms fall into the different-name group
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python (pandas)
  role: Load the merged (391-entry) and pre-merge (547-entry) datasets and perform
    exact string comparison of reported metabolite names against PubChem title names,
    then aggregate counts and percentages
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:8f04634b6ff6f22985eed8adc371402453c05f62547d3201124cbc5a6a30d974
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python (pandas)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pubchem-title-name-matching

## Summary

A post-merge name-consistency audit that labels each metabolite in a merged dataset as carrying its PubChem 'title' name or a different (synonym/acronym) name, and quantifies how many compounds would have gone unmerged without the Metabolites Merging Strategy (MMS).

## When to use

Use when a merged metabolomics dataset (e.g., 391 urinary asthma metabolites) has been assembled via InChIKeys and you must quantify name consistency against PubChem title names or estimate name-only merge failures.

## When NOT to use

- The goal is to perform the merge itself via InChIKeys (MMS Step 1) rather than audit name consistency of an already-merged dataset
- The dataset has no PubChem title names or no retrieved name attributes from MMS Step 2, so exact string comparison against a reference name is impossible
- You need fuzzy/synonym-aware matching; this audit intentionally uses exact string matching so synonyms fall into the different-name group

## Inputs

- Reproduced merged metabolite dataset (391 metabolites, Supplementary Table S2) with reported metabolite names and PubChem 'title' names
- Pre-merge combined dataset (547 entries) from the urinary asthma studies

## Outputs

- naming_statistics.csv with counts and percentages of same-name vs different-name metabolites
- Five-subgroup classification of multi-source compounds (same name unique/other studies, different names correct/other)
- Count of compounds that would have gone unmerged without MMS (expected: 213 of 372)

## How to apply

Load the reproduced merged dataset (391 metabolites, Supplementary Table S2) and the pre-merge combined dataset (547 entries) with pandas. For each merged entry, compare the reported metabolite name against the PubChem 'title' name using exact string matching, labeling it 'same name' or 'different name' — this deliberately sends synonyms (e.g., 3,4-dihydroxycinnamic acid vs caffeic acid) and acronyms (e.g., CAR 12:0 vs lauroylcarnitine) into the different-name group because exact string matching is the point of the audit. Aggregate counts and percentages of the two classes, then categorize compounds found in more than one source into the five reported subgroups: 'same name (unique metabolites)', 'same name (also in other studies)', 'same name (other studies differ)', 'different names (other studies correct)', and 'different name'. Finally, sum the 'same name (other studies differ)' and 'different name (other studies correct)' subgroups to derive how many compounds would not have been merged by name alone, and assemble all counts and percentages into a statistics table (naming_statistics.csv). The rationale, per the MMS paper, is that name-based merging is unreliable — the title name from PubChem retrieved via the InChIKey in MMS Step 2 is the reference name standard.

## Related tools

- **Python (pandas)** (Load the merged (391-entry) and pre-merge (547-entry) datasets and perform exact string comparison of reported metabolite names against PubChem title names, then aggregate counts and percentages)

## Examples

```
df['same_name'] = df['metabolite_name'] == df['pubchem_title']; stats = df['same_name'].value_counts(normalize=True) * 100
```

## Evaluation signals

- Recomputed same/different-name split over the 391-entry merged set reproduces 179 (46%) vs 212 (54%)
- Sum of 'same name (other studies differ)' and 'different name (other studies correct)' subgroups equals 213 of 372 compounds
- Synonym and acronym pairs (e.g., 3,4-dihydroxycinnamic acid vs caffeic acid, CAR 12:0 vs lauroylcarnitine) are classified as different-name, not same-name
- Counts and percentages sum correctly across the five subgroups and are written to naming_statistics.csv

## Limitations

- Exact string matching means legitimate synonyms and acronyms are counted as 'different name' even when they refer to the same compound
- Statistics depend on the quality of PubChem title names retrieved via InChIKey; missing attributes propagate as errors
- The unmerged-without-MMS estimate (213 of 372) applies only to the 372 multi-source compounds, not the full merged set

## Evidence

- [abstract] descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench): "descriptors of name (title name from PubChem and RefMet name from Metabolomics Workbench)"
- [abstract] we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [results] In the merged dataset, 179 metabolites (46%) share the PubChem title name and 212 (54%) have different names: [evidence span withheld — anchor status: not_found]
