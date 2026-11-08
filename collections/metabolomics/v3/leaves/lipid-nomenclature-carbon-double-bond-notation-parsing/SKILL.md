---
name: lipid-nomenclature-carbon-double-bond-notation-parsing
description: Use when a lipidomic data table carries lipid identifiers like 'PG 18:1/20:1'
  or 'PE(P-40:6)' and the goal is to extract fatty-acyl residues (carbon:double-bond
  notation) and retain only residues meeting carbon-count or saturation criteria.
when_to_use_negative:
- The input is already a residue-level table with no lipid string identifiers to parse
- Lipid identifiers are not parseable as 'CLASS N:N/M:M' or RefMet-compliant names
  and cannot be batch-translated via RefMet's online translator
- The analysis requires the original individual molecular lipid species rather than
  aggregated fatty-acyl residues
edam_operation: http://edamontology.org/operation_3695
tools:
- name: liputils
  role: Provides the Lipid class (.residues(), .refmet_residues(), drop_ambiguous),
    make_residues_table(), and the saturated()/max_carbon() chain-filtering predicates
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (lipid names as index, samples as columns) that
    make_residues_table() consumes and that the filtered residue table is written
    from
- name: Python
  role: Language in which the saturated()/max_carbon() predicates and list-comprehension
    filtering are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:f7706939638a63f8f162d4df6534e0b961a7124d63794bfb8d1c3c5964ca9402
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-nomenclature-carbon-double-bond-notation-parsing

## Summary

Parses lipid string identifiers (e.g. 'CLASS N:N/M:M' or RefMet-compliant names) into individual fatty-acyl residues with liputils, then chain-filters residues with the saturated() and max_carbon() predicates. This enables reprocessing of lipidomic tables to track specific carbon-chain classes, such as keeping only unsaturated residues up to a 30-carbon cutoff.

## When to use

Use when a lipidomic data table carries lipid identifiers like 'PG 18:1/20:1' or 'PE(P-40:6)' and the goal is to extract fatty-acyl residues (carbon:double-bond notation) and retain only residues meeting carbon-count or saturation criteria.

## When NOT to use

- The input is already a residue-level table with no lipid string identifiers to parse
- Lipid identifiers are not parseable as 'CLASS N:N/M:M' or RefMet-compliant names and cannot be batch-translated via RefMet's online translator
- The analysis requires the original individual molecular lipid species rather than aggregated fatty-acyl residues

## Inputs

- pandas.DataFrame of lipidomic data with lipid names as vertical index and samples in columns (e.g. 'liputils sample data.xlsx')
- Lipid string identifiers in the form 'CLASS N:N/M:M', 'CLASS(N:N/M:M)(isomers)', or RefMet-compliant residue names
- Carbon-atom cutoff n (e.g. 16, 24, or 30 carbons)

## Outputs

- Residue count table (submolar fractions/residue counts per sample) from make_residues_table()
- Boolean predicate values from saturated(residue) and max_carbon(residue, n)
- Filtered residue table written to filtered_residues_table.csv

## How to apply

Load the lipidomics data into a pandas DataFrame with lipid names as the vertical index and samples as columns, then extract residues per identifier with the liputils Lipid class (.residues() or make_residues_table(cleanup=True), which trims unwanted strings like 'total', 'fc', 'tc'). Implement the predicate saturated(residue), returning True when a residue such as '12:0' has zero double bonds, and max_carbon(residue, n), returning True when the carbon count is at most n (e.g. max_carbon('12:0', 16) is True but max_carbon('21:3', 16) is False). Combine them in a list comprehension such as [not saturated(x) and max_carbon(x, 24) for x in residues] to select unsaturated residues under a carbon cutoff; in the ST000668 dietary-intervention reprocessing the authors kept all residues up to 30 carbon atoms to exclude low-resolution compounds like PE(P-40:6)/PE(O-40:7). Apply the predicate to the residue index of the make_residues_table() output and write the filtered residue count table to filtered_residues_table.csv.

## Related tools

- **liputils** (Provides the Lipid class (.residues(), .refmet_residues(), drop_ambiguous), make_residues_table(), and the saturated()/max_carbon() chain-filtering predicates) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (lipid names as index, samples as columns) that make_residues_table() consumes and that the filtered residue table is written from)
- **Python** (Language in which the saturated()/max_carbon() predicates and list-comprehension filtering are implemented)

## Examples

```
from liputils import saturated, max_carbon; keep = [x for x in residue_index if not saturated(x) and max_carbon(x, 30)]
```

## Evaluation signals

- saturated('12:0') returns True
- max_carbon('12:0', 16) returns True and max_carbon('21:3', 16) returns False
- [not saturated(x) and max_carbon(x, 24) for x in ['12:0', '17:1', '24:0', '24:1', '24:2', '26:3']] yields [False, True, False, True, True, False]
- Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1)
- filtered_residues_table.csv exists with the expected residue-index rows and sample columns

## Limitations

- Ambiguous lipid isomers yield multiple residue sets unless residues() is called with drop_ambiguous=True, in which case ambiguous lipids return empty residue lists
- Low-resolution compounds such as PE(P-40:6)/PE(O-40:7) can originate from a broad variety of compounds, motivating the 30-carbon cutoff but excluding long-chain information
- Unwanted index strings like 'total', 'fc', 'tc' (total lipid class counts) must be trimmed via cleanup/unwanted, otherwise they pollute the residue table

## Evidence

- [readme] mix saturated/max_carbon predicates: "To focus on particular residues, it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
- [readme] documented predicate examples: "[not saturated(x) and max_carbon(x, 24) for x in my_lipids]
[False, True, False, True, True, False]"
- [methods] Lipid class extracts residues: "The Lipid class takes care of extracting information from the lipid name"
- [results] 30-carbon cutoff rationale: "We included in our analysis all residues up 30 carbon atoms in length"
- [abstract] accepted identifier formats: "There is no prerequisite data format, as liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
