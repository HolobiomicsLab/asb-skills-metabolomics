---
name: inchikey-protonation-layer-interpretation
description: Use when a merged metabolite table contains multiple InChIKeys that share
  the same 14-character molecular-skeleton prefix (e.g. palmitate vs. palmitic acid)
  and entries must be reduced to one non-conjugated compound per pair.
when_to_use_negative:
- The input is a single-source table where conjugated base/acid duplicates were already
  resolved upstream (MMS marks this curation step as optional).
- InChIKeys are absent or non-standard, so the 14-character skeleton prefix and protonation
  layer cannot be compared reliably.
- Rows differ in skeleton or stereochemistry blocks, not just the protonation layer
  — those are distinct compounds, not conjugate pairs.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python
  role: Scripting language used to implement the skeleton-prefix grouping and pair-merging
    logic in the manual-curation step
- name: pandas
  role: Library used to load the merged metabolite table, extract the 14-character
    InChIKey prefix, group rows, and write the curated CSV and merge log
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:f02b5c8654777d514f53ad3ee2e75c00c0e99749494d6762fccf90f921067f89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# inchikey-protonation-layer-interpretation

## Summary

Identifies conjugated base/acid compound pairs in a merged metabolite dataset by grouping InChIKeys on their first 14 skeleton characters and comparing the protonation-indicating final layer, enabling consolidation of duplicate acid/base forms with preserved source provenance.

## When to use

Use when a merged metabolite table contains multiple InChIKeys that share the same 14-character molecular-skeleton prefix (e.g. palmitate vs. palmitic acid) and entries must be reduced to one non-conjugated compound per pair.

## When NOT to use

- The input is a single-source table where conjugated base/acid duplicates were already resolved upstream (MMS marks this curation step as optional).
- InChIKeys are absent or non-standard, so the 14-character skeleton prefix and protonation layer cannot be compared reliably.
- Rows differ in skeleton or stereochemistry blocks, not just the protonation layer — those are distinct compounds, not conjugate pairs.

## Inputs

- Merged metabolite dataset (CSV) with one row per compound, columns including 27-character InChIKey and Reference (source) column

## Outputs

- Curated metabolite table (CSV) with conjugated forms removed and concatenated source labels in the Reference column of retained rows
- Merge log file listing each conjugate pair: removed InChIKey, retained InChIKey, and merged sources

## How to apply

Load the merged metabolite table (one row per compound with its 27-character InChIKey and a Reference column) into a pandas DataFrame. Compute the first 14 characters of each InChIKey — the molecular-skeleton block — and group rows sharing the same prefix. Within each skeleton group, flag conjugated base/acid pairs whose InChIKeys differ only in the protonation-indicating final layer, as in the palmitate/palmitic-acid example from the MMS Step 3 optional manual-curation step. For each flagged pair, delete the conjugated entry and retain the non-conjugated compound, concatenating the source labels of both entries into the retained row's Reference column so provenance is not lost. Write the curated table to CSV and a separate log file listing every merged pair (removed InChIKey, retained InChIKey, merged sources). The rationale is that protonation-state duplicates otherwise inflate compound counts and confound cross-study comparability that the MMS framework targets.

## Related tools

- **Python** (Scripting language used to implement the skeleton-prefix grouping and pair-merging logic in the manual-curation step)
- **pandas** (Library used to load the merged metabolite table, extract the 14-character InChIKey prefix, group rows, and write the curated CSV and merge log)

## Examples

```
import pandas as pd; df = pd.read_csv('merged_metabolites.csv'); df['skel'] = df['InChIKey'].str[:14]; grouped = df.groupby('skel')
```

## Evaluation signals

- The curated CSV exists and every retained row has a populated Reference field.
- The number of pairs recorded in the merge log equals the row-count reduction from input to curated table.
- Every retained compound is verified by expert check to be the non-conjugated form of its merged pair.
- No two retained rows share the same 14-character InChIKey skeleton prefix.
- All removed InChIKeys in the log differ from their retained counterpart only in the final (protonation) layer of the 27-character key.

## Limitations

- The article describes this as an optional manual-curation step, so fully automated runs may miss pairs requiring expert judgment.
- Provenance preservation depends entirely on correctly concatenating sources into the Reference column; errors here lose track of merged origins.
- The pairing logic is validated on conjugated base/acid examples (e.g. palmitate vs. palmitic acid); other protonation-layer variations may need manual inspection.

## Evidence

- [methods] conjugated base/acid compound entries are deleted (merged), with their source recorded in the Reference column of the retained non-conjugated compound: [evidence span withheld — anchor status: not_found]
- [abstract] a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step): "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step)"
- [methods] Extract the first 14 characters of each 27-character InChIKey (the molecular-skeleton block) and group rows sharing the same skeleton prefix: [evidence span withheld — anchor status: not_found]
