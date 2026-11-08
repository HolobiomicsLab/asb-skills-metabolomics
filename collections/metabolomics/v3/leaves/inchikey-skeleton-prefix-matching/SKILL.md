---
name: inchikey-skeleton-prefix-matching
description: Use when merging metabolite datasets that report heterogeneous non-systematic
  identifiers and need a systematic key to collapse conjugated base/acid pairs or
  duplicate entries before manual curation.
when_to_use_negative:
- When the input dataset already contains only unique, canonical InChIKeys with no
  conjugated base/acid duplicates — skeleton matching would collapse distinct protonation
  states that the user intentionally kept separate.
- When the dataset's InChIKeys are missing, truncated, or malformed (fewer than 27
  characters), because the 14-character skeleton block cannot be reliably extracted.
- When the goal is synonym or duplicate detection that depends on name-level equivalence
  rather than structural equivalence — that is a later curation step ('synonym checking
  (duplicated information)') in the MMS, not this skill.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: pandas
  role: Used to load the merged metabolite dataset, extract the first 14 InChIKey
    characters via string slicing, group rows by the skeleton prefix, and write both
    the curated CSV table and the merge log file.
- name: Python
  role: The scripting language in which the skeleton-prefix matching and conjugate-pair
    merging are implemented as part of the optional manual-curation step of MMS.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:b8741ac34320cb7114e6f34f4a406bbf7490697cb2216530c4ef8cc1ad741073
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# inchikey-skeleton-prefix-matching

## Summary

Groups metabolite entries across merged datasets by comparing the first 14 characters (molecular-skeleton block) of their InChIKeys, so that structurally identical compounds are consolidated even when protonation state, charge, or conjugation differs between databases.

## When to use

Use when merging metabolite datasets that report heterogeneous non-systematic identifiers and need a systematic key to collapse conjugated base/acid pairs or duplicate entries before manual curation.

## When NOT to use

- When the input dataset already contains only unique, canonical InChIKeys with no conjugated base/acid duplicates — skeleton matching would collapse distinct protonation states that the user intentionally kept separate.
- When the dataset's InChIKeys are missing, truncated, or malformed (fewer than 27 characters), because the 14-character skeleton block cannot be reliably extracted.
- When the goal is synonym or duplicate detection that depends on name-level equivalence rather than structural equivalence — that is a later curation step ('synonym checking (duplicated information)') in the MMS, not this skill.

## Inputs

- Merged metabolite dataset containing one row per compound with its InChIKey and a Reference (source) column
- InChIKey strings (27 characters) for each compound, or original compound names requiring InChI translation prior to skeleton matching

## Outputs

- Curated metabolite table with one row per non-conjugated compound and a Reference column that lists all merged source identifiers
- Log file enumerating every merged conjugate pair (removed InChIKey, retained InChIKey, merged sources)

## How to apply

First translate each metabolite name to its full 27-character InChIKey (using the systematic InChI layer generated from the compound's structure) so every row in the merged dataset carries an unambiguous chemical identifier. Extract the first 14 characters of each InChIKey — the molecular-skeleton (connectivity) block — and group rows that share the identical skeleton prefix. Within each skeleton group, flag rows that differ only in the final protonation-indicating layer (e.g., palmitate vs. palmitic acid), because the article notes that a 'conjugated base/acid compounds (optional step)' is the first curation stage of the MMS. When a conjugated pair is found, delete the conjugated entry, keep the non-conjugated compound, and append the deleted entry's source into the Reference column of the retained row to preserve provenance. Write the curated table (one row per non-conjugated compound with complete Reference provenance) to a CSV file and write a separate log file listing every merged conjugate pair — removed InChIKey, retained InChIKey, and merged sources — so that the provenance of every consolidation is auditable.

## Related tools

- **pandas** (Used to load the merged metabolite dataset, extract the first 14 InChIKey characters via string slicing, group rows by the skeleton prefix, and write both the curated CSV table and the merge log file.)
- **Python** (The scripting language in which the skeleton-prefix matching and conjugate-pair merging are implemented as part of the optional manual-curation step of MMS.)

## Examples

```
python collapse_conjugates.py --input merged_metabolites.csv --inchikey_column inchikey --reference_column reference --output curated_metabolites.csv --log merge_log.txt
```

## Evaluation signals

- The curated table exists in CSV format and every retained row carries a populated Reference field.
- The number of entries recorded as merged in the log file equals the row-count reduction from the input dataset to the curated table.
- An expert check confirms that every retained compound is the non-conjugated form of its conjugated pair (e.g., the acid form, not the base form).
- No two distinct skeleton prefixes remain within the output — each unique 14-character InChIKey block maps to exactly one retained row.
- The merged-source provenance in the Reference column matches the 'merged sources' recorded for each pair in the log file.

## Limitations

- The skeleton-prefix match relies on complete, correctly translated InChIKeys; compounds whose names cannot be translated to an InChI will not be matched and must be handled separately.
- Merging conjugated base/acid compounds is explicitly an OPTIONAL step in the MMS three-step curation process; its omission is acceptable when conjugated forms should be kept distinct.
- Provenance is preserved only by appending source labels to the Reference column — the original conjugated entry's own attributes (e.g., its molecular weight) are lost from the output table and survive only in the log.

## Evidence

- [abstract] a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information): "a meticulous three-step curation process is used to rectify disparities for conjugated base/acid compounds (optional step), missing attributes, and synonym checking (duplicated information)"
- [methods] In the optional first manual-curation step, conjugated base/acid compound entries are deleted (merged), with their source recorded in the Reference column of the retained non-conjugated compound.: [evidence span withheld — anchor status: not_found]
- [abstract] Step 1: translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed): "Step 1: translation and merging of the different datasets by employing InChIKeys for data integration, encompassing the translation of metabolite names (if needed)"
