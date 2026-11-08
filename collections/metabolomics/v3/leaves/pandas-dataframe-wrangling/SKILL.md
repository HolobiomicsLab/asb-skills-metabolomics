---
name: pandas-dataframe-wrangling
description: Use when a published supplementary lipidomics table (e.g. a srep44503
  Excel sheet, one analyte per row, one sample per column) must be reformatted for
  downstream residue extraction with liputils.
when_to_use_negative:
- The table is already a processed residues table with residue names as the index
  — re-running would re-parse residue identifiers instead of lipid class names
- Lipid identifiers are fully RefMet-compliant and the default make_residues_table()
  nomenclature already applies
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Loads and reshapes the lipidomics table via read_excel (index_col=1, header=2)
    into the DataFrame layout liputils expects
- name: liputils
  role: Provides make_residues_table() and the Lipid class that parse CLASS N:N/M:M
    identifiers and drop unwanted/non-numerical entries
  repo: https://github.com/Stemanz/liputils
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:e1c9c1aa8a6f914a52830b2ca95f430f04fd97224b34217bd44500c1a1caa06c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pandas-dataframe-wrangling

## Summary

Reshape published lipidomics spreadsheets into the vertical-index (lipid names as index, samples as columns) pandas.DataFrame layout that liputils requires, then convert and export the processed residue-count table.

## When to use

Use when a published supplementary lipidomics table (e.g. a srep44503 Excel sheet, one analyte per row, one sample per column) must be reformatted for downstream residue extraction with liputils.

## When NOT to use

- The table is already a processed residues table with residue names as the index — re-running would re-parse residue identifiers instead of lipid class names
- Lipid identifiers are fully RefMet-compliant and the default make_residues_table() nomenclature already applies

## Inputs

- Supplementary lipidomics spreadsheet (.xlsx) with one analyte per row and one sample per column
- pandas.DataFrame with lipid names as vertical index and samples in columns

## Outputs

- Residues pandas.DataFrame with parsed fatty acid residues (e.g. 18:1, 20:4) as index and samples as columns
- Tab-separated CSV file named residues_table.csv

## How to apply

Load the supplementary spreadsheet with pandas.read_excel using index_col=1 and header=2 so lipid names form the vertical index and samples the columns — the accepted format is 'a vertical index with lipid names, and samples in column'. Pass the DataFrame to liputils' make_residues_table(), which drops non-numerical columns and trims unwanted index entries (default unwanted list: total, fc, tc) such as total lipid class counts. If analyte names are not RefMet-compliant (e.g. of the form 'CLASS N:N/M:M/...'), call make_residues_table() with liptype=None so the Lipid class parses them, since the function otherwise defaults to RefMet nomenclature. Save the returned residues DataFrame — residue names as index, samples as columns — as a tab-separated CSV (residues_table.csv).

## Related tools

- **pandas** (Loads and reshapes the lipidomics table via read_excel (index_col=1, header=2) into the DataFrame layout liputils expects)
- **liputils** (Provides make_residues_table() and the Lipid class that parse CLASS N:N/M:M identifiers and drop unwanted/non-numerical entries) — https://github.com/Stemanz/liputils

## Examples

```
import pandas as pd; from liputils import make_residues_table; df = pd.read_excel('srep44503_table.xlsx', index_col=1, header=2); res = make_residues_table(df, liptype=None); res.to_csv('residues_table.csv', sep='\t')
```

## Evaluation signals

- residues_table.csv exists and is tab-separated
- The output index contains parsed fatty acid residues (e.g. 18:1, 20:4) rather than original lipid class identifiers (e.g. PC, TAG)
- Non-numerical columns and unwanted index entries matching the default list (total, fc, tc) have been dropped
- Row/column counts of the residues table match the number of parsed residues and original samples

## Limitations

- make_residues_table() defaults to RefMet nomenclature; non-RefMet analyte names will not parse unless liptype=None is passed
- Index entries containing the default unwanted strings ('total', 'fc', 'tc') are trimmed, so custom total-count labels need the unwanted parameter
- Non-numerical columns are dropped automatically, so any metadata columns in the source sheet are discarded

## Evidence

- [methods] load DataFrame: "The accepted format is a vertical index with lipid names, and samples in column."
- [methods] make_residues_table transforms table: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [readme] unwanted default list: "unwanted: <list> <set> <tuple> Strings that must be removed from the lipid index. Defaults to ["total", "fc", "tc"]"
- [full_text] non-RefMet handling: [evidence span withheld — anchor status: not_found]
- [readme] generic lipid format: "liputils accepts a generic lipid format in the form of CLASS N:N/M:M/../.. or CLASS(N:N/M:M/../..)(other mass isomers)"
