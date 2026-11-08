---
name: lipidomics-table-reprocessing
description: Use when a processed lipidomics table (lipid names as index, samples
  as columns) contains analyte identifiers in 'CLASS N:N/M:M' or RefMet form and you
  need per-sample fatty acid residue counts.
when_to_use_negative:
- The input is already a residue-level table or feature matrix rather than a table
  with one analyte per row and one sample per column
- The analysis requires raw mass-spectrometry processing (peak picking, alignment)
  — liputils works on processed, final lipidomic data, not instrument files
- Lipid identifiers follow a nomenclature that is neither 'CLASS N:N/M:M/..' format,
  bracketed isomer format, nor RefMet-compliant
edam_operation: http://edamontology.org/operation_3234
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Core module providing Lipid, make_residues_table(), refmet_residues(), saturated()
    and max_carbon() to parse lipid identifiers, extract fatty acid residues, and
    filter residues by chain properties
  repo: github.com/Stemanz/liputils
- name: pandas
  role: Loads and holds the lipidomics table as a DataFrame with lipid names as index
    and samples in columns, as required by make_residues_table()
- name: Python
  role: Language in which the reprocessing workflow (Lipid parsing, residue filtering
    via list comprehension, CSV export) is implemented
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:9fcb4c2fa4112368a322f5e1441b568eabcb325fd8fcef3236faf2dcd0bd995b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3234
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipidomics-table-reprocessing

## Summary

Reprocess final lipidomic data tables with liputils to extract individual fatty acid moieties from lipid analyte identifiers, yielding residue-count tables that can be filtered by chain length and saturation using saturated() and max_carbon().

## When to use

Use when a processed lipidomics table (lipid names as index, samples as columns) contains analyte identifiers in 'CLASS N:N/M:M' or RefMet form and you need per-sample fatty acid residue counts.

## When NOT to use

- The input is already a residue-level table or feature matrix rather than a table with one analyte per row and one sample per column
- The analysis requires raw mass-spectrometry processing (peak picking, alignment) — liputils works on processed, final lipidomic data, not instrument files
- Lipid identifiers follow a nomenclature that is neither 'CLASS N:N/M:M/..' format, bracketed isomer format, nor RefMet-compliant

## Inputs

- pandas.DataFrame lipidomics table with lipid names as vertical index and samples as columns (e.g. liputils sample data.xlsx loaded via pandas.read_excel with index_col=1, header=2)
- Lipid analyte identifiers in 'CLASS N:N/M:M/..' format, 'CLASS(N:N/M:M)(isomers)' format, or RefMet-compliant residue naming

## Outputs

- residues pandas.DataFrame with fatty acid residue names (e.g. 18:1, 20:4) as index and samples as columns, in original units or absolute molecule counts
- Filtered residue count table (e.g. filtered_residues_table.csv) retaining unsaturated residues up to a chosen carbon cutoff
- Tab-separated CSV such as residues_table.csv for the transformed table

## How to apply

Load the lipidomics spreadsheet into a pandas.DataFrame with lipid names as the vertical index and samples in columns (e.g. pandas.read_excel with index_col=1, header=2). Call make_residues_table() on the DataFrame — it drops non-numerical columns and trims unwanted index entries such as total lipid class counts (default unwanted list: 'total', 'fc', 'tc'); pass liptype=None when analyte names are non-RefMet 'CLASS N:N/M:M/...' strings so the Lipid class parses them. To count absolute molecule numbers rather than keeping original units, use the absolute_amount parameter; ambiguous isomers can be dropped via drop_ambiguous or residues()/make_residues_table() with drop_ambiguous=True, where each residue is divided by its uncertainty. Filter residues by mixing saturated(residue) (True when zero double bonds) and max_carbon(residue, n) (True when carbon count ≤ n); for the Metabolomics Workbench ST000668 dietary intervention reprocessing, only residues up to 30 carbon atoms were retained because low-resolution compounds like PE(P-40:6)/PE(O-40:7) can originate from a broad variety of compounds. Save the resulting residue table (residue names as index, samples as columns), e.g. as a tab-separated CSV, and verify the index contains parsed residues like 18:1 or 20:4 rather than original lipid class identifiers.

## Related tools

- **liputils** (Core module providing Lipid, make_residues_table(), refmet_residues(), saturated() and max_carbon() to parse lipid identifiers, extract fatty acid residues, and filter residues by chain properties) — github.com/Stemanz/liputils
- **pandas** (Loads and holds the lipidomics table as a DataFrame with lipid names as index and samples in columns, as required by make_residues_table())
- **Python** (Language in which the reprocessing workflow (Lipid parsing, residue filtering via list comprehension, CSV export) is implemented)

## Examples

```
from liputils import make_residues_table, saturated, max_carbon; res = make_residues_table(df); keep = [r for r in res.index if not saturated(r) and max_carbon(r, 24)]
```

## Evaluation signals

- Predicate truth values match the documented examples: saturated('12:0') == True; max_carbon('12:0', 16) == True; max_carbon('21:3', 16) == False; [not saturated(x) and max_carbon(x, 24) for x in ['12:0','17:1','24:0','24:1','24:2','26:3']] == [False, True, False, True, True, False]
- residues_table.csv exists, is tab-separated, and its index contains parsed fatty acid residues (e.g. 18:1, 20:4) rather than original lipid class identifiers
- Non-numerical columns and unwanted index entries (total, fc, tc) are absent from the returned residue table
- Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1); with drop_ambiguous=True, an ambiguous TAG like 'TAG 48:2 total (14:0/16:0/18:2)...' yields ([], 0)
- For RefMet inputs, refmet_residues() on 'octadecatrienoic acid', 'linolenic acid' and 'FA(18:3)' all return (['18:3'], 1)

## Limitations

- make_residues_table() defaults to RefMet nomenclature; passing non-RefMet-compliant tables without liptype=None causes incorrect extraction
- Low-resolution compound identifiers (e.g. PE(P-40:6)/PE(O-40:7)) can originate from a broad variety of compounds, motivating the 30-carbon cutoff rather than trusting long-chain residues
- By default isobaric/ambiguous lipids are kept and residues from all mass isomers are extracted; use drop_ambiguous=True to reject them (each residue then divided by its uncertainty)
- Number of molecules is computed from amount assuming picomoles by default; other units must be set explicitly (e.g. unit='femtomoles')

## Evidence

- [readme] The accepted format is a vertical index with lipid names, and samples in column.: "The accepted format is a vertical index with lipid names, and samples in column."
- [methods] make_residues_table drops non-numerical columns and trims unwanted lipids: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [full_text] Because the input table is not RefMet-compliant, make_residues_table() must be called with the liptype=None parameter: [evidence span withheld — anchor status: not_found]
- [full_text] The C-30 cutoff was applied because the original data includes low-resolution compounds: [evidence span withheld — anchor status: not_found]
- [readme] mix saturated() and max_carbon() to dictate which residues to keep: "it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
