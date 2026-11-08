---
name: predicate-based-residue-index-filtering
description: Use when a liputils-produced residue count table (residues as DataFrame
  index, samples as columns) must be subset by chain-length and unsaturation, e.g.
  restricting to residues ≤30 carbons because low-resolution compounds like PE(P-40:6)/PE(O-40:7)
  are ambiguous.
when_to_use_negative:
- The input is still a raw lipidomics table of lipid identifiers rather than a residues
  table — run make_residues_table() first
- You need to remove ambiguous/isobaric lipids themselves rather than filter residue
  chain properties — use residues(drop_ambiguous=True) instead
- All residues should be kept, e.g. when the dataset contains only high-resolution,
  unambiguous annotations and no long-chain cutoff is warranted
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Provides the saturated() and max_carbon() predicates mixed in a list comprehension
    to select which residues to keep in the table index
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the residues-count DataFrame whose index is filtered by the boolean
    predicate mask
- name: Python
  role: Execution environment for the predicate list comprehension and CSV export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:47518d2c42886443a63c0ca8a7bf1adacac2f89c2bb165c8705493d329fae1e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# predicate-based-residue-index-filtering

## Summary

Applies liputils' saturated() and max_carbon() predicates to filter the fatty-acyl residue index of a residues-count table, keeping e.g. only unsaturated residues up to a chosen carbon cutoff (C-30 in the ST000668 dietary intervention reanalysis).

## When to use

Use when a liputils-produced residue count table (residues as DataFrame index, samples as columns) must be subset by chain-length and unsaturation, e.g. restricting to residues ≤30 carbons because low-resolution compounds like PE(P-40:6)/PE(O-40:7) are ambiguous.

## When NOT to use

- The input is still a raw lipidomics table of lipid identifiers rather than a residues table — run make_residues_table() first
- You need to remove ambiguous/isobaric lipids themselves rather than filter residue chain properties — use residues(drop_ambiguous=True) instead
- All residues should be kept, e.g. when the dataset contains only high-resolution, unambiguous annotations and no long-chain cutoff is warranted

## Inputs

- Residues-count pandas.DataFrame from liputils make_residues_table() (residue names as vertical index, samples as columns, submolar fractions or residue counts)
- Fatty-acyl residue strings in N:M notation (e.g. '12:0', '24:1', '26:3')
- Chosen maximum carbon cutoff n (e.g. 24 in the README example, 30 in the ST000668 analysis)

## Outputs

- Filtered residue count table (submolar/residue counts per sample) retaining only residues matching the predicate, written to filtered_residues_table.csv
- Boolean predicate truth-value lists used as the index mask

## How to apply

After building a residues table with make_residues_table() (cleanup=True drops unwanted strings such as 'total', 'fc', 'tc'), import saturated and max_carbon from liputils and combine them in a boolean list comprehension over the residue index. saturated(residue) returns True for zero double bonds (e.g. saturated('12:0') is True), while max_carbon(residue, n) returns True when the carbon count is at most n (max_carbon('12:0', 16) is True; max_carbon('21:3', 16) is False). Choose the carbon cutoff based on data quality — in the Metabolomics Workbench ST000668 dietary intervention reanalysis only residues up to 30 carbon atoms were retained because the original data include low-resolution compounds such as PE(P-40:6)/PE(O-40:7) that can originate from many species. Invert saturated() with 'not' to keep unsaturated residues, as in [not saturated(x) and max_carbon(x, 24) for x in my_lipids], then use the boolean mask to subset the table index. Write the filtered submolar/residue-count table to filtered_residues_table.csv.

## Related tools

- **liputils** (Provides the saturated() and max_carbon() predicates mixed in a list comprehension to select which residues to keep in the table index) — https://github.com/Stemanz/liputils
- **pandas** (Holds the residues-count DataFrame whose index is filtered by the boolean predicate mask)
- **Python** (Execution environment for the predicate list comprehension and CSV export)

## Examples

```
from liputils import saturated, max_carbon; keep = [not saturated(x) and max_carbon(x, 24) for x in my_lipids]
```

## Evaluation signals

- saturated('12:0') == True
- max_carbon('12:0', 16) == True and max_carbon('21:3', 16) == False
- [not saturated(x) and max_carbon(x, 24) for x in ['12:0', '17:1', '24:0', '24:1', '24:2', '26:3']] == [False, True, False, True, True, False]
- filtered_residues_table.csv exists with the expected filtered residue-index rows and sample columns
- In the ST000668 reanalysis, no residue longer than 30 carbon atoms remains in the filtered table

## Limitations

- The C-30 cutoff is a judgment call: low-resolution compounds like PE(P-40:6)/PE(O-40:7) can originate from a broad variety of compounds, so long-chain residues are excluded rather than resolved
- Predicates only operate on residue strings in N:M notation; residues must first be correctly extracted (cleanup=True removes unwanted strings like 'total', 'fc', 'tc')
- Combining saturated() and max_carbon() filters the index but does not resolve isobaric ambiguity — use drop_ambiguous separately for that

## Evidence

- [readme] predicates on residues: "To focus on particular residues, it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
- [readme] predicate examples: ">>> my_lipids = ["12:0", "17:1", "24:0", "24:1", "24:2", "26:3"]

>>> [not saturated(x) and max_carbon(x, 24) for x in my_lipids]
[False, True, False, True, True, False]"
- [results] C-30 cutoff rationale: "We included in our analysis all residues up 30 carbon atoms in length"
- [methods] table cleanup: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
