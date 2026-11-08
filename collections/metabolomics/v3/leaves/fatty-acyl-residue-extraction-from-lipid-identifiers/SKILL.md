---
name: fatty-acyl-residue-extraction-from-lipid-identifiers
description: Use when input is a lipidomics table whose row index contains lipid identifiers
  such as 'PG 18:1/20:1' or RefMet-compliant names, and the goal is to reprocess analyte-level
  data into per-sample fatty-acid residue counts or filtered residue tables.
when_to_use_negative:
- Input is already a residue-count/feature table (rows are residues, not lipid identifiers)
  — no extraction is needed
- Lipid identifiers are neither of the form CLASS N:N/M:M/.., mass-isomer bracketed,
  nor RefMet-compliant — batch-translate IDs with RefMet's online translator first
- The analysis requires intact individual molecular lipid species rather than aggregated
  fatty-acyl residues, since this skill deliberately discards the parent-lipid context
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: liputils
  role: Python module providing the Lipid class, residues(), refmet_residues(), make_residues_table(),
    GUI, and the saturated()/max_carbon() filtering predicates
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: 'Required input container format: lipidomics data loaded as a pandas.DataFrame
    with lipid names as vertical index and samples in columns'
- name: Jupyter Notebook
  role: Step-by-step instructions for the whole analysis are provided as a Jupyter
    Notebook in the Supplemental methods
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:ff59619c8b07b0702d0c1a11e151444efb4061cf91dccd7f6b8702bc421e305f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Jupyter Notebook
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fatty-acyl-residue-extraction-from-lipid-identifiers

## Summary

Extracts individual fatty-acyl moieties (e.g. '18:1', '20:4') from complex lipid string identifiers into a residue-count table using the liputils Python module, enabling lipidome-wide tracking of carbon chains independently of their parent lipids.

## When to use

Use when input is a lipidomics table whose row index contains lipid identifiers such as 'PG 18:1/20:1' or RefMet-compliant names, and the goal is to reprocess analyte-level data into per-sample fatty-acid residue counts or filtered residue tables.

## When NOT to use

- Input is already a residue-count/feature table (rows are residues, not lipid identifiers) — no extraction is needed
- Lipid identifiers are neither of the form CLASS N:N/M:M/.., mass-isomer bracketed, nor RefMet-compliant — batch-translate IDs with RefMet's online translator first
- The analysis requires intact individual molecular lipid species rather than aggregated fatty-acyl residues, since this skill deliberately discards the parent-lipid context

## Inputs

- pandas.DataFrame with lipid names as vertical index and samples as columns (e.g. 'liputils sample data.xlsx')
- Lipid identifiers in the form 'CLASS N:N/M:M/..' or 'CLASS(N:N/M:M/..)(other mass isomers)' or RefMet-compliant residue naming
- Optional 'unwanted' list of index strings to remove (default: ['total', 'fc', 'tc'])

## Outputs

- pandas.DataFrame of residue counts (submolar fractions or absolute residue counts with absolute_amount=True) per sample, with residues as the index
- Filtered residue tables such as filtered_residues_table.csv retaining only residues passing saturated()/max_carbon() predicates
- Per-lipid residue lists with ambiguity counts, e.g. residues() -> (['18:1', '20:1'], 1)

## How to apply

Load the data into a pandas.DataFrame with lipid names as the vertical index and samples as columns (matching the liputils accepted format). Call make_residues_table() — or its GUI wrapper — to parse each identifier via the Lipid class (residues() / refmet_residues()), dropping non-numerical columns and unwanted strings like 'total', 'fc', 'tc' via cleanup=True. Use drop_ambiguous=False to keep isobaric residues divided by their uncertainty count, or True to reject non-unambiguous lipids such as 'TAG 48:2 total (...)'. Restrict the residue index with the saturated() and max_carbon() predicates, e.g. keep unsaturated residues up to 24 carbons via [not saturated(x) and max_carbon(x, 24) for x in residues]; in the ST000668 dietary-intervention reanalysis a C-30 cutoff was used because low-resolution compounds like PE(P-40:6)/PE(O-40:7) originate from a broad variety of species. Write the resulting residue-count table (submolar/residue counts per sample) to CSV, e.g. filtered_residues_table.csv.

## Related tools

- **liputils** (Python module providing the Lipid class, residues(), refmet_residues(), make_residues_table(), GUI, and the saturated()/max_carbon() filtering predicates) — https://github.com/Stemanz/liputils
- **pandas** (Required input container format: lipidomics data loaded as a pandas.DataFrame with lipid names as vertical index and samples in columns)
- **Jupyter Notebook** (Step-by-step instructions for the whole analysis are provided as a Jupyter Notebook in the Supplemental methods)

## Examples

```
from liputils import make_residues_table, saturated, max_carbon; res = make_residues_table(df); keep = [x for x in res.index if not saturated(x) and max_carbon(x, 24)]
```

## Evaluation signals

- saturated('12:0') returns True and max_carbon('12:0', 16) returns True while max_carbon('21:3', 16) returns False
- The list comprehension [not saturated(x) and max_carbon(x, 24) for x in ['12:0', '17:1', '24:0', '24:1', '24:2', '26:3']] yields [False, True, False, True, True, False]
- Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1); residues(drop_ambiguous=True) on the TAG 48:2 total multi-isobar returns ([], 0)
- RefMet round-trip: Lipid('linolenic acid').refmet_residues(), Lipid('octadecatrienoic acid').refmet_residues(), and Lipid('FA(18:3)').refmet_residues() all return (['18:3'], 1)
- The output CSV exists with residue-index rows (e.g. only unsaturated residues ≤ the C-30 cutoff) and one column per sample, matching the expected residue counts

## Limitations

- Unresolved isobar/isomer ambiguities (e.g. TAG 48:2 with three possible masses) either inflate residue counts proportionally or, with drop_ambiguous=True, remove the lipid entirely
- Low-resolution compounds such as PE(P-40:6)/PE(O-40:7) can originate from a broad variety of species, which motivated restricting analysis to residues up to 30 carbon atoms
- The accepted identifier formats are limited to CLASS N:N/M:M/.., bracketed mass-isomer forms, and RefMet-compliant names; other formats require prior batch translation via RefMet

## Evidence

- [methods] Lipidomics data should be loaded in a pandas.DataFrame table.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] make_residues_table cleanup: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [readme] predicate mixing: "To focus on particular residues, it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
- [results] C-30 cutoff rationale: "We included in our analysis all residues up 30 carbon atoms in length"
- [readme] drop_ambiguous behavior: "drop_ambiguous: <bool> don't take isobars into consideration. Defaults to False. If True, each residue is divided by its uncertainty."
- [readme] RefMet compliance: "By adhering to RefMet's nomenclature, any compliant lipid name will be properly managed by liputils's method ``refmet_residues()"
