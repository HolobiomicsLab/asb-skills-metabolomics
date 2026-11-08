---
name: lipid-common-name-decoding
description: Use when input lipid identifiers are common/RefMet names (e.g. 'octadecatrienoic
  acid', 'linolenic acid', 'linoleyl palmitate') rather than CLASS N:N/M:M shorthand,
  and you need carbon:double-bond residues.
when_to_use_negative:
- Input identifiers are already in CLASS N:N/M:M shorthand handled by plain Lipid
  parsing (e.g. 'PG 18:1/20:1') — use .residues() instead of .refmet_residues()
- Data contain total lipid class counts or non-lipid index elements that make_residues_table()
  would need to trim via the 'unwanted' parameter rather than individual molecular
  lipid names
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3353
tools:
- name: liputils
  role: Python module whose Lipid class parses lipid string identifiers; .refmet_residues()
    decodes common/RefMet names into fatty acid residue tuples
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds lipidomics tables (lipid names as index, samples in columns) and the
    decoded residue results for CSV export
- name: RefMet
  role: Nomenclature standard that defines compliant names; its online translator
    batch-converts lipid IDs to RefMet form before decoding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:aaca5b881b5bbb791dec48b590e11f223e036b3f3de0216be9b65b8dc74707c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - RefMet
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lipid-common-name-decoding

## Summary

Decodes standardized/common lipid names (RefMet-compliant textual identifiers such as 'arachidonic acid' or 'linoleyl palmitate') into their component fatty acid residue tuples using the liputils Python module's .refmet_residues() method, beyond what shorthand parsers like lipyd.name offer.

## When to use

Use when input lipid identifiers are common/RefMet names (e.g. 'octadecatrienoic acid', 'linolenic acid', 'linoleyl palmitate') rather than CLASS N:N/M:M shorthand, and you need carbon:double-bond residues.

## When NOT to use

- Input identifiers are already in CLASS N:N/M:M shorthand handled by plain Lipid parsing (e.g. 'PG 18:1/20:1') — use .residues() instead of .refmet_residues()
- Data contain total lipid class counts or non-lipid index elements that make_residues_table() would need to trim via the 'unwanted' parameter rather than individual molecular lipid names

## Inputs

- RefMet-compliant lipid names as Python strings (e.g. 'octadecatrienoic acid', 'linolenic acid', 'FA(18:3)', 'linoleyl palmitate')
- Optionally, a pandas.DataFrame with lipid names as vertical index for batch conversion via make_residues_table()

## Outputs

- Residue tuples (list of 'N:M' residue strings, ambiguity count) per lipid, e.g. (['18:1', '16:0'], 1) for 'linoleyl palmitate'
- A CSV of lipid name, decoded residues, and ambiguity count (e.g. common_name_residues.csv)

## How to apply

Instantiate a liputils Lipid object for each common-name identifier, then call .refmet_residues() to decode RefMet-compliant names into residue tuples. For example, 'octadecatrienoic acid', 'linolenic acid' and FA(18:3) each yield (['18:3'], 1), while the composite 'linoleyl palmitate' yields (['18:1', '16:0'], 1). The second tuple element is the ambiguity count, so validate that it matches expectations; composite compounds like esters of two fatty acids decode into multiple residues. Results (lipid name, decoded residues, ambiguity count) can be collected into a table such as common_name_residues.csv via pandas. If unsure whether names are RefMet-compliant, batch-translate identifiers with RefMet's online translator first.

## Related tools

- **liputils** (Python module whose Lipid class parses lipid string identifiers; .refmet_residues() decodes common/RefMet names into fatty acid residue tuples) — https://github.com/Stemanz/liputils
- **pandas** (Holds lipidomics tables (lipid names as index, samples in columns) and the decoded residue results for CSV export)
- **RefMet** (Nomenclature standard that defines compliant names; its online translator batch-converts lipid IDs to RefMet form before decoding)

## Examples

```
from liputils import Lipid; l = Lipid("linoleyl palmitate"); l.refmet_residues()  # (['18:1', '16:0'], 1)
```

## Evaluation signals

- 'octadecatrienoic acid', 'linolenic acid', and 'FA(18:3)' each return (['18:3'], 1) from refmet_residues()
- 'linoleyl palmitate' returns (['18:1', '16:0'], 1), confirming composite compounds decode into one residue per fatty acid moiety
- Every decoded ambiguity count (second tuple element) matches the expected number of isomer/moiety sets
- Decoded residue lists match the expected carbon:double-bond shorthand reported in the paper's examples; flag any mismatch
- All expected name-residue pairs are present in the output CSV with no dropped rows

## Limitations

- refmet_residues() requires names to be RefMet-compliant; other shorthand formats must go through Lipid.residues() or be batch-translated with RefMet's online translator first
- Ambiguous lipids with unresolved isomers yield multiple residue sets; the ambiguity count must be tracked or handled via drop_ambiguous

## Evidence

- [readme] By adhering to RefMet's nomenclature, any compliant lipid name will be properly managed by liputils's method ``refmet_residues(): "By adhering to RefMet's nomenclature, any compliant lipid name will be properly managed by liputils's method ``refmet_residues()"
- [other] liputils text-decodes common lipid names into residues via refmet_residues, e.g. 'linoleyl palmitate' -> (['18:1', '16:0'], 1): [evidence span withheld — anchor status: not_found]
- [abstract] liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services: "There is no prerequisite data format, as liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
- [readme] if unsure about your data format, you can try and batch-translate your lipid IDs with RefMet's online translator: "If unsure about your data format, you can try and batch-translate your lipid IDs with [RefMet's online translator]"
