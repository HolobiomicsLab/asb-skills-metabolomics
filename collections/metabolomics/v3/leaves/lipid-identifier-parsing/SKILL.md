---
name: lipid-identifier-parsing
description: Use when a processed lipidomics table (lipid identifiers as vertical
  index, samples as columns) needs its analyte names decomposed into fatty acid residues
  such as 18:1 or 20:4, whether names follow the generic 'CLASS N:N/M:M' form, isomer-group
  form, or RefMet nomenclature.
when_to_use_negative:
- The data are raw mass-spectrometry spectra or peak lists — liputils operates on
  processed, final lipidomic data tables with annotated identifiers, not on instrument
  output
- The table index contains aggregate entries such as total lipid class counts ('total',
  'fc', 'tc') that should be trimmed rather than parsed as molecular lipids
- Identifiers are not parseable annotations (e.g. 'CLASS N:N/M:M', isomer groups,
  or RefMet names) — ambiguous or free-text names outside these formats should first
  be batch-translated with RefMet's online translator
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Python module whose Lipid class and make_residues_table() parse lipid string
    identifiers to extract fatty acid residues; also provides saturated(), max_carbon(),
    refmet_residues(), and a GUI wrapper
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics data as a DataFrame with lipid names as vertical index
    and samples in columns, the format make_residues_table() accepts
- name: RefMet
  role: Reference nomenclature standard; compliant identifiers are handled via .refmet_residues(),
    and RefMet's online translator can batch-convert unknown lipid IDs
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:e5e9341b62dc7237a9ecb16ee5a19676ffe2c60e9a10f90a56753967fb358dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - RefMet
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lipid-identifier-parsing

## Summary

Parse lipid string identifiers (e.g. 'PG 18:1/20:1', 'TAG 48:2 total (14:0/16:0/18:2)(...)', RefMet names like 'linolenic acid' or 'FA(18:3)') into their constituent fatty acid residues using the liputils Python module's Lipid class and make_residues_table(). This enables residue-level reprocessing of processed lipidomic data tables without mass spectrometry expertise.

## When to use

Use when a processed lipidomics table (lipid identifiers as vertical index, samples as columns) needs its analyte names decomposed into fatty acid residues such as 18:1 or 20:4, whether names follow the generic 'CLASS N:N/M:M' form, isomer-group form, or RefMet nomenclature.

## When NOT to use

- The data are raw mass-spectrometry spectra or peak lists — liputils operates on processed, final lipidomic data tables with annotated identifiers, not on instrument output
- The table index contains aggregate entries such as total lipid class counts ('total', 'fc', 'tc') that should be trimmed rather than parsed as molecular lipids
- Identifiers are not parseable annotations (e.g. 'CLASS N:N/M:M', isomer groups, or RefMet names) — ambiguous or free-text names outside these formats should first be batch-translated with RefMet's online translator

## Inputs

- pandas.DataFrame of processed lipidomic data with lipid string identifiers as vertical index and samples as columns (e.g. an .xlsx supplementary table loaded via pandas.read_excel with index_col=1, header=2)
- Individual lipid identifier strings in the form 'CLASS N:N/M:M/../..' or 'CLASS(N:N/M:M)(other mass isomers)', or RefMet-compliant names
- liputils sample data file 'liputils sample data.xlsx' for testing

## Outputs

- A residues pandas.DataFrame (residue names such as 18:1, 20:4 as index; samples as columns), optionally saved as a tab-separated CSV like residues_table.csv
- Per-lipid tuple of (residue list, ambiguity count), e.g. (['18:1', '20:1'], 1) for 'PG 18:1/20:1'

## How to apply

Load the lipidomics spreadsheet into a pandas.DataFrame with lipid names as a vertical index and samples in columns (e.g. pandas.read_excel with index_col=1 and header=2). Instantiate Lipid objects or call make_residues_table(df); the text-based parser recognizes the generic format 'CLASS N:N/M:M/../..' and the isomer-group format 'CLASS(N:N/M:M)(...)'. If identifiers are non-RefMet, call make_residues_table() with liptype=None (the default assumes RefMet nomenclature); for RefMet-compliant names use .refmet_residues(), which maps names like 'linolenic acid' and 'FA(18:3)' to residues such as ['18:3']. Handle isomer ambiguities via residues(drop_ambiguous=True) to reject ambiguous lipids, or keep all residues divided by the ambiguity count when drop_ambiguous=False; make_residues_table() also drops non-numerical columns and trims unwanted index entries (default unwanted list: 'total', 'fc', 'tc'). Optionally filter the residue index with saturated() and max_carbon() (e.g. the paper kept residues up to 30 carbon atoms) and save the returned DataFrame with residue names as index and samples as columns.

## Related tools

- **liputils** (Python module whose Lipid class and make_residues_table() parse lipid string identifiers to extract fatty acid residues; also provides saturated(), max_carbon(), refmet_residues(), and a GUI wrapper) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics data as a DataFrame with lipid names as vertical index and samples in columns, the format make_residues_table() accepts)
- **RefMet** (Reference nomenclature standard; compliant identifiers are handled via .refmet_residues(), and RefMet's online translator can batch-convert unknown lipid IDs)

## Examples

```
from liputils import make_residues_table; l = Lipid("PG 18:1/20:1"); l.residues(); res = make_residues_table(df)
```

## Evaluation signals

- 'PG 18:1/20:1' parses to residues ['18:1', '20:1'] with ambiguity count 1; 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)' yields 9 residues with ambiguity 3
- 'octadecatrienoic acid', 'linolenic acid' and 'FA(18:3)' all return (['18:3'], 1) from refmet_residues()
- With drop_ambiguous=True, the ambiguous TAG example returns ([], 0); with False, each residue is divided by its uncertainty count
- The output residues table has parsed fatty acid residues (e.g. 18:1, 20:4) as index rather than original lipid class identifiers, with sample columns retained and non-numerical columns and 'total'/'fc'/'tc' entries removed
- saturated('12:0') is True, max_carbon('12:0', 16) is True, and max_carbon('21:3', 16) is False for index-filtering logic

## Limitations

- Isomer-ambiguous identifiers (e.g. TAG total isomer groups) either introduce ambiguity counts that must be managed, or must be dropped entirely via drop_ambiguous=True
- The default RefMet nomenclature assumption requires liptype=None for non-RefMet-compliant tables; unparseable IDs may need batch translation with RefMet's online translator
- Unwanted index strings like total lipid class counts must be handled — make_residues_table() trims only the default list ['total', 'fc', 'tc'] unless the unwanted parameter is extended

## Evidence

- [abstract] liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services: "There is no prerequisite data format, as liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
- [readme] The accepted format is a vertical index with lipid names, and samples in column.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed, like total lipid class counts."
- [readme] It is possible to reject non unambiguous lipids altogether by calling .residues() with the drop_ambiguous parameter: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [methods] The Lipid class takes care of extracting information from the lipid name: "The Lipid class takes care of extracting information from the lipid name"
- [results] We included in our analysis all residues up 30 carbon atoms in length: "We included in our analysis all residues up 30 carbon atoms in length"
