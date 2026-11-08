---
name: metabolite-duplicate-merging-with-provenance
description: Use when a merged metabolite dataset built from InChIKey-based integration
  contains both protonated/deprotonated forms of the same compound, causing one chemical
  entity to occupy multiple rows with conflicting source identifiers.
when_to_use_negative:
- The dataset has already been curated for conjugated base/acid compounds in a prior
  MMS run, so the optional first curation step would be redundant
- Rows differ in more than the protonation-indicating InChIKey layer (different molecular
  skeletons), meaning they are genuinely distinct compounds and must not be merged
- No InChIKey column is available, since grouping relies on the first 14 characters
  of the 27-character InChIKey
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: Python (pandas)
  role: Loads the merged metabolite dataset, extracts InChIKey skeleton prefixes,
    groups and merges conjugate pairs, and writes the curated CSV plus merge log
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:d34be252084a58829edcafed9282f12492a23a2f1200c53129da42ad202969fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python (pandas)
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolite-duplicate-merging-with-provenance

## Summary

Optional first step of the MMS Step 3 curation process that collapses conjugated base/acid compound duplicates (e.g., palmitate vs. palmitic acid) in a merged metabolite table by grouping on the first 14 characters of the 27-character InChIKey, retaining the non-conjugated form, and preserving source provenance in a Reference column.

## When to use

Use when a merged metabolite dataset built from InChIKey-based integration contains both protonated/deprotonated forms of the same compound, causing one chemical entity to occupy multiple rows with conflicting source identifiers.

## When NOT to use

- The dataset has already been curated for conjugated base/acid compounds in a prior MMS run, so the optional first curation step would be redundant
- Rows differ in more than the protonation-indicating InChIKey layer (different molecular skeletons), meaning they are genuinely distinct compounds and must not be merged
- No InChIKey column is available, since grouping relies on the first 14 characters of the 27-character InChIKey

## Inputs

- Merged metabolite dataset (CSV or pandas DataFrame) with one row per compound, including a 27-character InChIKey column and a Reference (source provenance) column

## Outputs

- Curated CSV table with one row per non-conjugated compound and a Reference field carrying concatenated source labels on every retained row
- Merge log file listing every merged conjugate pair: removed InChIKey, retained InChIKey, and merged sources

## How to apply

Load the merged metabolite dataset (one row per compound, with InChIKey and a Reference column) using pandas in Python. Compute the first 14 characters of each 27-character InChIKey — the molecular-skeleton block — and group rows sharing the same skeleton prefix; within each group, flag conjugated base/acid pairs whose InChIKeys differ only in the final protonation-indicating layer (e.g., palmitate vs. palmitic acid). For each flagged pair, delete the conjugated form and keep the non-conjugated compound, concatenating the source labels of both entries into the Reference column of the retained row. Write the curated table to CSV and a separate log file listing every merged pair (removed InChIKey, retained InChIKey, merged sources). The rationale, per MMS, is to rectify disparities for conjugated base/acid compounds before the subsequent missing-attributes and synonym-checking curation steps, so downstream pathway analysis is not distorted by duplicate chemical entities.

## Related tools

- **Python (pandas)** (Loads the merged metabolite dataset, extracts InChIKey skeleton prefixes, groups and merges conjugate pairs, and writes the curated CSV plus merge log)

## Examples

```
df = pd.read_csv('merged_metabolites.csv'); df['skel'] = df['InChIKey'].str[:14]; df.groupby('skel').apply(merge_conjugate_pairs).to_csv('curated_metabolites.csv', index=False)
```

## Evaluation signals

- The curated table exists in CSV format and contains a Reference field on every retained row
- The number of merged pairs recorded in the log equals the row-count reduction from input to curated table
- Expert check confirms every retained compound is the non-conjugated form of its pair
- Retained rows' Reference columns contain the concatenated sources of both original entries, so provenance is complete after merging

## Limitations

- This curation step is explicitly optional within the MMS three-step curation process, so skipping it leaves conjugated base/acid duplicates unrectified
- The skeleton-prefix heuristic only detects pairs whose InChIKeys differ solely in the final protonation-indicating layer; other disparities (missing attributes, synonyms) are handled by the later curation steps, not this one

## Evidence

- [abstract] a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information): "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [abstract] Merging strategy framing: "we introduce the Metabolites Merging Strategy (MMS), offering a systematic framework to harmonize multiple metabolite datasets for enhanced interstudy comparability"
- [methods] Merge mechanics and provenance: [evidence span withheld — anchor status: not_found]
- [methods] Skeleton-prefix grouping criterion: [evidence span withheld — anchor status: not_found]
