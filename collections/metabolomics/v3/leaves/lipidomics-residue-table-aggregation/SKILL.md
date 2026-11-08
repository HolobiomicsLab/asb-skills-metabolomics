---
name: lipidomics-residue-table-aggregation
description: Use when you have a final lipidomic abundance table (lipid species as
  row index, samples as columns) and need per-residue fatty acid abundances aggregated
  across replicate groups for cross-sample or cross-cell-type comparison.
when_to_use_negative:
- Input is already a residue-level table rather than individual molecular lipid species
  — re-running make_residues_table() would be a no-op or error
- Analysis requires intact individual molecular lipid species or lipid-class totals
  (which make_residues_table intentionally trims as 'unwanted')
- Lipid identifiers are free-text annotations not in 'CLASS N:N/M:M' or RefMet format
  and cannot be batch-translated — parsing may fail or drop residues
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: liputils
  role: 'Core Python module: Lipid class parses lipid identifiers to extract fatty
    acid residues; make_residues_table() converts the whole abundance table; saturated()/max_carbon()
    filter the residue index'
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (lipid names as index, samples as columns) and
    the transformed residue table; used for replicate aggregation and ranking
- name: Python
  role: Runtime environment for liputils and pandas aggregation workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:d41d81c271366437c12a8b7c888aa66fcb68877350fe06df8db1a982fcdc219a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lipidomics-residue-table-aggregation

## Summary

Converts processed lipidomic abundance tables into fatty-acid residue count tables with liputils' make_residues_table(), then aggregates replicate samples per biological group to compare residue abundances across cell types (e.g., PHHs, HLCs, HepG2) or conditions.

## When to use

Use when you have a final lipidomic abundance table (lipid species as row index, samples as columns) and need per-residue fatty acid abundances aggregated across replicate groups for cross-sample or cross-cell-type comparison.

## When NOT to use

- Input is already a residue-level table rather than individual molecular lipid species — re-running make_residues_table() would be a no-op or error
- Analysis requires intact individual molecular lipid species or lipid-class totals (which make_residues_table intentionally trims as 'unwanted')
- Lipid identifiers are free-text annotations not in 'CLASS N:N/M:M' or RefMet format and cannot be batch-translated — parsing may fail or drop residues

## Inputs

- pandas.DataFrame of processed lipidomic abundances with lipid names as vertical index and samples as columns (submolar amounts, e.g. pmol/ug)
- Lipid identifiers in 'CLASS N:N/M:M/..' format or RefMet-compliant names (translatable via RefMet's online batch translator)
- Sample-to-group mapping defining replicates per cell type or condition (e.g., HLC, PHH, HepG2; culture medium vs cells)

## Outputs

- Residue count/abundance table (pandas DataFrame) with fatty acid residues (e.g., 16:0, 18:1, 18:2, 20:4) as index and samples as columns
- Per-group mean residue abundance summary (e.g., residue_abundance_summary.csv) with per-pattern verification flags

## How to apply

Load the lipidomic data into a pandas.DataFrame with lipid names as the vertical index and samples as columns; make_residues_table() drops non-numerical columns and trims unwanted index entries (default unwanted strings: 'total', 'fc', 'tc') such as total lipid class counts. Parse each lipid identifier (format 'CLASS N:N/M:M/..' or RefMet-compliant names) via the Lipid class, extracting residues with residues() or refmet_residues(); decide whether to keep ambiguous isomers (drop_ambiguous=False keeps all, dividing each residue by its uncertainty) and whether to count absolute molecule numbers (absolute_amount=True, picomoles by default, unit configurable e.g. 'femtomoles'). Optionally filter the residue index with saturated() and max_carbon() helpers (e.g., keeping only residues up to 30 carbon atoms, as in the dietary-intervention analysis). Then aggregate replicate columns per group (mean residue abundance per cell type), rank residues within a group of interest (e.g., HepG2 medium: 18:1, 16:0, 18:2, 20:4 top), and compare group means (e.g., absolute distances showing HLC 16:0/18:1 closer to PHHs) before writing the summary (residue_abundance_summary.csv). Rationale: residue-level reanalysis reveals global lipidome shifts that single-lipid or ratio-focused analyses can miss ('unable to see the forest for the trees').

## Related tools

- **liputils** (Core Python module: Lipid class parses lipid identifiers to extract fatty acid residues; make_residues_table() converts the whole abundance table; saturated()/max_carbon() filter the residue index) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (lipid names as index, samples as columns) and the transformed residue table; used for replicate aggregation and ranking)
- **Python** (Runtime environment for liputils and pandas aggregation workflow)

## Examples

```
from liputils import make_residues_table; res = make_residues_table(df)
```

## Evaluation signals

- Ranking check: after aggregation, the most abundant residues in HepG2 medium are 18:1, 16:0, 18:2 and 20:4
- Distance check: HLC group means for 16:0 and 18:1 are closer to PHH than to HepG2 means (trend need not be statistically significant)
- Schema check: output residue table retains numeric sample columns, with non-numerical columns and unwanted index entries ('total', 'fc', 'tc') removed
- Round-trip spot check: parsing a known lipid (e.g., 'PG 18:1/20:1') yields residues ['18:1','20:1'] with ambiguity count 1
- With drop_ambiguous=True, ambiguous species (e.g., TAG with multiple isomer triples) contribute zero residues; with False, residue counts are divided by the ambiguity factor

## Limitations

- Lipid identifiers must be in 'CLASS N:N/M:M/..' form or RefMet-compliant; other naming needs RefMet's online batch translator first
- Ambiguous isomer groups are either dropped entirely (drop_ambiguous=True) or residues are divided by uncertainty count — no isomer-specific resolution
- Molecule-count conversion assumes amounts are picomoles unless the unit parameter (e.g., 'femtomoles') is set; wrong units skew absolute_amount counts
- Total lipid class counts and other unwanted strings are silently trimmed by cleanup and can require a custom 'unwanted' list

## Evidence

- [readme] The accepted format is a vertical index with lipid names, and samples in column.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] make_residues_table drops non-numerical columns and trims unwanted lipids: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed, like total lipid class counts."
- [readme] drop_ambiguous parameter semantics: "drop_ambiguous: <bool> don't take isobars into consideration. Defaults to False. If True, each residue is divided by its uncertainty."
- [results] Top residues in HepG2 medium finding: [evidence span withheld — anchor status: not_found]
- [results] HLC closer to PHH for 16:0 and 18:1: "HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant"
- [results] Residues up to 30 carbons filter: "We included in our analysis all residues up 30 carbon atoms in length"
- [intro] Motivation: global lipidome changes overlooked: "This might lead to the inability to see the forest for the trees, as global changes in the lipidome can be overlooked."
