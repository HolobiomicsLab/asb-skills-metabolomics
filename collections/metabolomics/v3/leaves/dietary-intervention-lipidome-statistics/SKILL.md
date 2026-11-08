---
name: dietary-intervention-lipidome-statistics
description: Use when reprocessing a published human plasma lipidomics dataset (e.g.
  ST000668) to test whether dietary conditions shift the fatty-acid residue makeup
  of the lipidome, rather than changes in individual molecular lipid species.
when_to_use_negative:
- Your input is already a residue/feature table — running make_residues_table() again
  would double-transform the data
- The analysis goal is abundance changes of individual molecular lipid species or
  lipid class totals rather than fatty-acid residue makeup
- Lipid identifiers contain unresolved isobaric/isomeric ambiguities you intend to
  keep and weight, since this workflow sets drop_ambiguous=True to reject them
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: liputils
  role: Converts individual molecular lipid identifiers into fatty-acid residue counts
    via make_residues_table(); provides saturated() and max_carbon() filters and the
    drop_ambiguous parameter
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (vertical index of lipid names, samples in columns)
    and the resulting residue count table for grouping and statistics
- name: Python
  role: Execution environment for the liputils re-analysis workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:73b065a4941535c0b835f06c2a0ea73192b1c18618a0d658f56dd8313676ca9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# dietary-intervention-lipidome-statistics

## Summary

Statistical re-analysis of dietary intervention lipidomics (Metabolomics Workbench ST000668) in which lipid identifiers are converted to fatty-acid residue counts with liputils, then grouped by diet (Unsaturated fats, Saturated fats, Overfeeding) to detect per-residue percentage changes, e.g. the significant increase of residue 26:0 (p = 0.018).

## When to use

Use when reprocessing a published human plasma lipidomics dataset (e.g. ST000668) to test whether dietary conditions shift the fatty-acid residue makeup of the lipidome, rather than changes in individual molecular lipid species.

## When NOT to use

- Your input is already a residue/feature table — running make_residues_table() again would double-transform the data
- The analysis goal is abundance changes of individual molecular lipid species or lipid class totals rather than fatty-acid residue makeup
- Lipid identifiers contain unresolved isobaric/isomeric ambiguities you intend to keep and weight, since this workflow sets drop_ambiguous=True to reject them

## Inputs

- Metabolomics Workbench dietary intervention lipidomics dataset ST000668 (https://doi.org/10.21228/M85P6M)
- pandas.DataFrame with lipid names as vertical index and samples as columns (abundances in submolar fractions)
- RefMet-compliant lipid identifiers (or non-compliant IDs batch-translated via RefMet's online translator)

## Outputs

- Residue count table (pandas.DataFrame) of fatty-acid residues per sample, restricted to residues of 30 carbons or fewer
- Per-residue percentage-change table versus the Unsaturated fats baseline for Saturated fats and Overfeeding conditions
- Reproduced significance statistics, e.g. 26:0 increase with p = 0.018

## How to apply

Download the dietary intervention dataset from Metabolomics Workbench (study ST000668, https://doi.org/10.21228/M85P6M) and load it into a pandas.DataFrame with lipid names as a vertical index and samples as columns, batch-translating lipid IDs with RefMet's online translator if they are not RefMet-compliant. Convert identifiers to residue counts with liputils' make_residues_table(), calling it with drop_ambiguous=True so isobaric/isomeric lipids with unresolved ambiguities are rejected rather than overcounted. Filter the residue index with the saturated() and max_carbon() functions, keeping only residues up to 30 carbon atoms in length as done in the published analysis. Group residue abundances (in submolar fractions) by dietary condition (Unsaturated fats, Saturated fats, Overfeeding) and compute per-residue percentage changes versus the Unsaturated fats baseline. Compare reproduced statistics against the published result: residue 26:0 significantly increased under Saturated fats and Overfeeding versus Unsaturated fats (p = 0.018), with Overfeeding preferentially impacting residues longer than 20 carbons.

## Related tools

- **liputils** (Converts individual molecular lipid identifiers into fatty-acid residue counts via make_residues_table(); provides saturated() and max_carbon() filters and the drop_ambiguous parameter) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (vertical index of lipid names, samples in columns) and the resulting residue count table for grouping and statistics)
- **Python** (Execution environment for the liputils re-analysis workflow)

## Examples

```
from liputils import make_residues_table, saturated, max_carbon; res = make_residues_table(df, drop_ambiguous=True); keep = [x for x in res.index if max_carbon(x, 30)]
```

## Evaluation signals

- The reproduced 26:0 percentage-change table yields a statistically significant increase under Saturated fats and Overfeeding versus Unsaturated fats with p = 0.018, matching the published figure (Fig. 3A)
- The residue index contains only residues of 30 carbon atoms or fewer (verifiable by applying max_carbon() to every index entry)
- No ambiguous isobaric lipids contributed to the residue counts (drop_ambiguous=True returned empty residue lists for entries like 'TAG 48:2 total (14:0/16:0/18:2)...')
- Overall residue makeup shows no dramatic dietary effect, consistent with the reported finding that only long-chain saturated residues (e.g. 26:0) changed significantly
- Residue abundances are expressed in submolar fractions, and non-numerical columns and total lipid class counts were dropped during make_residues_table() processing

## Limitations

- Lipids with unresolved isobaric/isomeric ambiguity are dropped entirely when drop_ambiguous=True, which can discard data
- Non-RefMet-compliant identifiers must be batch-translated with RefMet's online translator before processing, otherwise parsing may fail
- The dietary effect detected was weak overall: no dramatic effect on residue makeup, with only residue 26:0 reaching statistical significance (p = 0.018)

## Evidence

- [results] Only residues up to 30 carbons filter: "We included in our analysis all residues up 30 carbon atoms in length"
- [results] Statistically significant 26:0 change: [evidence span withheld — anchor status: not_found]
- [methods] Data loading format: "The accepted format is a vertical index with lipid names, and samples in column."
- [readme] drop_ambiguous parameter: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [readme] RefMet batch translation: "If unsure about your data format, you can try and batch-translate your lipid IDs with [RefMet's online translator]"
