---
name: conjugate-base-acid-compound-harmonization
description: Use when a merged metabolite table keyed by full 27-character InChIKeys
  contains both protonated and deprotonated forms of the same compound (identical
  first-14-character molecular-skeleton block) and must be deduplicated before attribute
  retrieval.
when_to_use_negative:
- The dataset is not yet merged on InChIKeys — run MMS Step 1 (PubChem Identifier
  Exchange / CTS translation and merging) first
- The table contains no conjugated base/acid duplicates (no two rows share the same
  first-14-character InChIKey skeleton block), in which case this optional step is
  a no-op and should be skipped
- The analysis requires distinguishing protonation states explicitly (e.g., ion-specific
  pathway or charge-dependent modeling) — collapsing conjugates would erase that information
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Python library used to load the merged metabolite dataset, group rows by the
    first-14-character InChIKey skeleton prefix, merge conjugate pairs, and write
    the curated CSV and merge log
- name: PubChem Identifier Exchange Service
  role: Upstream translator that produced the InChIKeys this curation step operates
    on (MMS Step 1)
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:b00c58218ef2355b740e1e2fd2141046b525b7af262b9f03b1276f6108ee0d18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinVestigate
  - ChEBI
  - ClassyFire
  - DrugBank
  - HMDB
  - KEGG
  - LipidMaps
  - Metabolomics Workbench
  - PubChem Identifier Exchange Service
  - Python
  - RefMet
  - pandas
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# conjugate-base-acid-compound-harmonization

## Summary

Optional first manual-curation step of MMS Step 3 that collapses conjugated base/acid compound entries (e.g., palmitate vs. palmitic acid) in an InChIKey-merged metabolite dataset by retaining the non-conjugated form and preserving the deleted entry's source in its Reference column.

## When to use

Use when a merged metabolite table keyed by full 27-character InChIKeys contains both protonated and deprotonated forms of the same compound (identical first-14-character molecular-skeleton block) and must be deduplicated before attribute retrieval.

## When NOT to use

- The dataset is not yet merged on InChIKeys — run MMS Step 1 (PubChem Identifier Exchange / CTS translation and merging) first
- The table contains no conjugated base/acid duplicates (no two rows share the same first-14-character InChIKey skeleton block), in which case this optional step is a no-op and should be skipped
- The analysis requires distinguishing protonation states explicitly (e.g., ion-specific pathway or charge-dependent modeling) — collapsing conjugates would erase that information

## Inputs

- Merged metabolite dataset (CSV, one row per compound) with a 27-character InChIKey column and a Reference/source column
- Log of source labels per compound (e.g., Metabolomics Workbench studies ST001039/ST001048/ST001317, HMDB, PubMed literature articles)

## Outputs

- Curated CSV table with one row per non-conjugated compound and a complete Reference field on every retained row
- Separate merge log listing every conjugate pair (removed InChIKey, retained InChIKey, merged sources)

## How to apply

Load the merged metabolite dataset (one row per compound with its InChIKey and a Reference column) with pandas in Python. Extract the first 14 characters of each 27-character InChIKey — the molecular-skeleton layer — and group rows sharing that prefix. Within each skeleton group, flag conjugated base/acid pairs whose InChIKeys differ only in the final protonation-indicating layer (e.g., palmitate vs. palmitic acid). For each pair, delete the conjugated entry, keep the non-conjugated compound, and concatenate the removed entry's source label into the retained row's Reference column so provenance is not lost. Write the curated table to CSV and a separate log listing every merged pair (removed InChIKey, retained InChIKey, merged sources). In the urinary asthma case study, this step is part of the workflow that reduced 547 pre-merge metabolites to 391 final entries, though the reduction from Step 3c synonym checking is not separable from this card alone.

## Related tools

- **pandas** (Python library used to load the merged metabolite dataset, group rows by the first-14-character InChIKey skeleton prefix, merge conjugate pairs, and write the curated CSV and merge log)
- **PubChem Identifier Exchange Service** (Upstream translator that produced the InChIKeys this curation step operates on (MMS Step 1))

## Examples

```
df['skel'] = df['InChIKey'].str[:14]; merged = df.sort_values('conjugate').groupby('skel', as_index=False).agg({'Reference': lambda s: ';'.join(s), 'InChIKey': 'first'})
```

## Evaluation signals

- Curated table exists in CSV format and every retained row has a non-empty Reference field
- Number of merged pairs recorded in the log equals the row-count reduction from input to curated table
- Expert check confirms every retained compound is the non-conjugated form of its deleted pair (InChIKeys differ only in the protonation layer)
- Each conjugate pair shares an identical first-14-character InChIKey (molecular-skeleton block) before merging
- Downstream, the pre-curation merged table matched 547 metabolite rows before MMS Step 3 curation

## Limitations

- The step is explicitly optional in MMS Step 3 — apply it only when conjugated base/acid disparities are present
- Pair detection relies on the InChIKey protonation layer; compounds whose protonation difference is not encoded in the final InChIKey layer (e.g., zwitterions or salt forms) may be missed
- Provenance for the deleted conjugate is preserved only as concatenated text in the retained row's Reference column, so the removed row's own attribute values (MW, formula) are discarded, not merged

## Evidence

- [methods] Optional first manual-curation step collapses conjugates: [evidence span withheld — anchor status: not_found]
- [methods] Skeleton-layer comparison rule: [evidence span withheld — anchor status: not_found]
- [abstract] Step 3 scope in abstract: "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [results] Dataset-scale validation target: "Following the MMS, a total of 547 metabolites have been retrieved and merged, leading to a final dataset of 391 metabolites."
