---
name: tabular-lipidomics-data-validation
description: Use when a processed lipidomics table carries lipid identifiers such
  as 'PG 18:1/20:1' or 'FA(18:3)' and you must verify that liputils' Lipid parser
  extracts the expected fatty acid residues and ambiguity counts before downstream
  residue analysis.
when_to_use_negative:
- Input is raw mass spectrometry data rather than a processed tabular table of lipid
  identifiers — liputils parses textual lipid names, not spectra
- Lipid index contains total lipid class counts or other non-molecular entries — make_residues_table()'s
  cleanup (default unwanted list ['total', 'fc', 'tc']) trims these rather than validating
  them
- Identifiers are not RefMet-compliant and not in the supported generic/isomer-group
  formats — batch-translate IDs with RefMet's online translator first
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: liputils
  role: Python module providing the Lipid class, residues(), refmet_residues(), and
    make_residues_table() used to parse and validate lipid identifiers
  repo: github.com/Stemanz/liputils
- name: pandas
  role: Loads the lipidomics table with lipid names as vertical index and samples
    in columns before parsing
- name: Python
  role: Runtime environment in which liputils parsing and validation is executed (Jupyter
    Notebook tutorial provided)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:041aee0d66a7cbde03fc4ef3bc61a9a6309277b4b26221f49b8027e0fcb8a4a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tabular-lipidomics-data-validation

## Summary

A liputils-based method for validating that fatty acid residues can be correctly parsed from lipid identifiers (generic 'CLASS N:N/M:M', isomer-group, and RefMet-compliant names) in processed lipidomics tables, reproducing documented residue/ambiguity outputs on the package's sample data.

## When to use

Use when a processed lipidomics table carries lipid identifiers such as 'PG 18:1/20:1' or 'FA(18:3)' and you must verify that liputils' Lipid parser extracts the expected fatty acid residues and ambiguity counts before downstream residue analysis.

## When NOT to use

- Input is raw mass spectrometry data rather than a processed tabular table of lipid identifiers — liputils parses textual lipid names, not spectra
- Lipid index contains total lipid class counts or other non-molecular entries — make_residues_table()'s cleanup (default unwanted list ['total', 'fc', 'tc']) trims these rather than validating them
- Identifiers are not RefMet-compliant and not in the supported generic/isomer-group formats — batch-translate IDs with RefMet's online translator first

## Inputs

- pandas.DataFrame of lipidomics data with lipid names as vertical index and samples in columns
- Lipid identifier strings in 'CLASS N:N/M:M/../..' format
- Isomer-group identifiers like 'CLASS(N:N/M:M)(...)'
- RefMet-compliant names (e.g. 'linolenic acid', 'FA(18:3)', 'linoleyl palmitate')
- Sample data file 'liputils sample data.xlsx'

## Outputs

- Tuple of (residue list, ambiguity count) per lipid, e.g. (['18:1', '20:1'], 1)
- Empty residue list ([], 0) for ambiguous identifiers when drop_ambiguous=True
- Validated residues DataFrame covering all lipid identifiers in the input table

## How to apply

Load the source table (e.g. 'liputils sample data.xlsx') into a pandas.DataFrame with lipid names as the vertical index and samples in columns, the format liputils accepts. Instantiate Lipid objects for representative identifiers and call residues() to obtain (residue list, ambiguity count); support both the generic 'CLASS N:N/M:M/../..' format and isomer groups like 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)'. For RefMet-compliant names ('octadecatrienoic acid', 'linolenic acid', 'FA(18:3)'), use refmet_residues() instead. Decide the drop_ambiguous policy: True discards ambiguous isomer groups (returning ([], 0)), while False returns all residues with residues divided by their uncertainty count. Finally, run the parser over every identifier and confirm outputs match documented expectations such as 'PG 18:1/20:1' -> (['18:1', '20:1'], 1).

## Related tools

- **liputils** (Python module providing the Lipid class, residues(), refmet_residues(), and make_residues_table() used to parse and validate lipid identifiers) — github.com/Stemanz/liputils
- **pandas** (Loads the lipidomics table with lipid names as vertical index and samples in columns before parsing)
- **Python** (Runtime environment in which liputils parsing and validation is executed (Jupyter Notebook tutorial provided))

## Examples

```
from liputils import Lipid; l = Lipid('PG 18:1/20:1', amount=0.012512); assert l.residues() == (['18:1', '20:1'], 1)
```

## Evaluation signals

- Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1)
- Lipid('TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)').residues() returns 9 residues with ambiguity count 3, and ([], 0) with drop_ambiguous=True
- refmet_residues() on 'octadecatrienoic acid', 'linolenic acid', and 'FA(18:3)' each yields (['18:3'], 1)
- refmet_residues() on composite 'linoleyl palmitate' yields (['18:1', '16:0'], 1)
- make_residues_table() drops non-numerical columns and removes unwanted index entries (default ['total', 'fc', 'tc']) from the transformed table

## Limitations

- Ambiguous isomer groups must be handled explicitly: either all residues are returned with an ambiguity count, or drop_ambiguous=True discards the lipid entirely
- Only the generic 'CLASS N:N/M:M/../..' format, the isomer-group format, and fully RefMet-compliant naming are supported; other annotation styles should be translated via RefMet first
- Non-molecular index entries such as total lipid class counts must be removed via cleanup/unwanted parameters before parsing

## Evidence

- [readme] PG 18:1/20:1 example: ">>> l.residues()
(['18:1', '20:1'], 1)"
- [readme] TAG ambiguity example: ">>> l.residues()                
(['14:0', '16:0', '18:2', '14:0', '16:1', '18:1', '16:0', '16:1', '16:1'], 3)"
- [readme] drop_ambiguous behavior: ">>> l.residues(drop_ambiguous=True)          
([], 0)"
- [readme] RefMet parsing examples: ">>> lip1.refmet_residues()                                                
(['18:3'], 1)"
- [methods] accepted data format: "The accepted format is a vertical index with lipid names, and samples in column."
- [methods] supported lipid formats: "liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
