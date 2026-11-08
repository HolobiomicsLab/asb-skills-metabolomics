---
name: liputils-exported-api-usage
description: Use when a lipidomics table with RefMet-compliant lipid names as the
  row index must be reprocessed into residue counts, or when residues must be kept/discarded
  by predicates such as saturated() and max_carbon(residue, n) cutoffs.
when_to_use_negative:
- The input is already a residue-level feature table rather than individual molecular
  lipid identifiers needing residue extraction
- Lipid IDs are non-standard and have not been batch-translated with RefMet's online
  translator — use refmet_residues() or the translator first
- The analysis requires absolute molecule counts; then absolute_amount=True must be
  set rather than keeping source submolar units
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Core Python module providing Lipid, make_residues_table(), saturated(), max_carbon(),
    refmet_residues(), and a GUI wrapper for converting lipidomic tables to residue
    counts
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: 'Required input container: lipidomics data must be loaded as a pandas.DataFrame
    with lipid names as index and samples in columns'
- name: Python
  role: Runtime environment in which the liputils API is invoked (Jupyter Notebook
    protocols documented in the paper's supplemental methods)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:84352c86f575289b367a38a66f685ab672add07c8b6d7204d7253b2dbc4e6eda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# liputils-exported-api-usage

## Summary

Programmatic use of the liputils Python API (Lipid, make_residues_table, saturated, max_carbon) to extract fatty-acyl residues from RefMet-style lipid identifiers and filter residues by unsaturation and carbon-length cutoffs in a pandas DataFrame of lipidomic samples.

## When to use

Use when a lipidomics table with RefMet-compliant lipid names as the row index must be reprocessed into residue counts, or when residues must be kept/discarded by predicates such as saturated() and max_carbon(residue, n) cutoffs.

## When NOT to use

- The input is already a residue-level feature table rather than individual molecular lipid identifiers needing residue extraction
- Lipid IDs are non-standard and have not been batch-translated with RefMet's online translator — use refmet_residues() or the translator first
- The analysis requires absolute molecule counts; then absolute_amount=True must be set rather than keeping source submolar units

## Inputs

- pandas.DataFrame with lipid names as vertical index and samples as columns (.xlsx or CSV, e.g. liputils sample data.xlsx)
- RefMet-compliant lipid identifiers of form 'CLASS N:N/M:M/..' or 'CLASS(N:N/M:M/..)(isomers)'
- Optional unwanted-string list via the cleanup/unwanted parameters (defaults to ['total','fc','tc'])

## Outputs

- Residue count table (submolar fractions per sample) returned by make_residues_table()
- Filtered residue table restricted by saturated()/max_carbon() predicates, written to filtered_residues_table.csv
- Per-lipid residue lists and ambiguity counts from Lipid.residues()

## How to apply

Load the lipidomic table into a pandas.DataFrame with lipid names as the vertical index and samples as columns, then call make_residues_table(df) with cleanup=True to drop unwanted strings such as 'total', 'fc', 'tc'. Use the Lipid class (e.g. Lipid('PG 18:1/20:1').residues()) to extract residues and ambiguity counts, optionally with drop_ambiguous=True to reject unresolved isomer mixtures like 'TAG 48:2 total (...)(...)(...)'. Define boolean predicates saturated(residue) (True for zero double bonds, e.g. '12:0') and max_carbon(residue, n) (True when carbon count ≤ n), and validate them against the README examples: saturated('12:0')==True, max_carbon('12:0',16)==True, max_carbon('21:3',16)==False, and [not saturated(x) and max_carbon(x,24) for x in ['12:0','17:1','24:0','24:1','24:2','26:3']] == [False, True, False, True, True, False]. Apply the combined predicate to the residue index to keep only unsaturated residues under the chosen carbon cutoff — in the Metabolomics Workbench ST000668 dietary-intervention reprocessing, residues up to 30 carbons were retained because low-resolution compounds such as PE(P-40:6)/PE(O-40:7) originate from many species. Write the filtered residue count table (submolar/residue counts per sample) to filtered_residues_table.csv.

## Related tools

- **liputils** (Core Python module providing Lipid, make_residues_table(), saturated(), max_carbon(), refmet_residues(), and a GUI wrapper for converting lipidomic tables to residue counts) — https://github.com/Stemanz/liputils
- **pandas** (Required input container: lipidomics data must be loaded as a pandas.DataFrame with lipid names as index and samples in columns)
- **Python** (Runtime environment in which the liputils API is invoked (Jupyter Notebook protocols documented in the paper's supplemental methods))

## Examples

```
from liputils import make_residues_table, saturated, max_carbon; res = make_residues_table(df, cleanup=True); keep = [x for x in res.index if not saturated(x) and max_carbon(x, 30)]
```

## Evaluation signals

- Predicate truth values reproduce README examples: saturated('12:0')==True, max_carbon('12:0',16)==True, max_carbon('21:3',16)==False
- List comprehension over ['12:0','17:1','24:0','24:1','24:2','26:3'] with (not saturated(x) and max_carbon(x,24)) yields [False, True, False, True, True, False]
- Lipid('PG 18:1/20:1').residues() returns (['18:1','20:1'], 1); refmet_residues() on 'octadecatrienoic acid', 'linolenic acid' and 'FA(18:3)' all return (['18:3'], 1)
- filtered_residues_table.csv exists with the expected residue-index rows and sample columns after applying the ≤30-carbon cutoff
- make_residues_table() output has non-numerical columns dropped and unwanted strings ('total', 'fc', 'tc') removed from the lipid index

## Limitations

- Ambiguous isomer mixtures (e.g. TAG 48:2 total with three unresolved compositions) yield multi-residue lists unless drop_ambiguous=True is set, in which case they are discarded entirely (returns ([], 0))
- The carbon cutoff is analysis-specific: only residues up to 30 carbons were included for ST000668 because low-resolution compounds like PE(P-40:6)/PE(O-40:7) are ambiguous as to origin
- replace_nan should not be set to False per the README docstring guidance

## Evidence

- [readme] README predicate examples: ">>> my_lipids = ["12:0", "17:1", "24:0", "24:1", "24:2", "26:3"]

>>> [not saturated(x) and max_carbon(x, 24) for x in my_lipids]"
- [readme] make_residues_table behavior: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [methods] input format: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [results] C-30 cutoff rationale: "We included in our analysis all residues up 30 carbon atoms in length"
- [methods] GUI wrapper: "The swiftest way to convert your lipidomic data into a residue count (usually given in submolar fractions) is by using **liputils' built-in GUI**"
