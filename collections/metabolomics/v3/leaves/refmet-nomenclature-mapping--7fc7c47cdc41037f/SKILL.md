---
name: refmet-nomenclature-mapping
description: Use when lipid identifiers in a processed lipidomic table are RefMet-compliant
  textual names (e.g. 'FA(18:3)', 'octadecatrienoic acid') rather than the generic
  'CLASS N:N/M:M' format, and you need per-residue carbon:double-bond strings.
when_to_use_negative:
- Identifiers already follow the generic 'CLASS N:N/M:M/../..' or 'CLASS(...)(...)'
  isomer-group format — use the standard Lipid.residues() parser instead of refmet_residues().
- You need to reject isobarically ambiguous lipids outright (drop_ambiguous=True on
  make_residues_table) rather than decode RefMet common names.
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Provides the Lipid class and .refmet_residues() method that text-decode RefMet-compliant
    names into fatty acid residues with ambiguity counts
  repo: https://github.com/Stemanz/liputils
- name: RefMet
  role: Nomenclature standard and online batch translator used to convert non-compliant
    lipid IDs into RefMet-compliant names before parsing
- name: pandas
  role: Holds the lipidomic table (lipid names as index, samples in columns) and collects
    decoded residues into an output DataFrame
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:54edf502c5d6f9514f35468706395dd28b3db191bb6c63062605e95d7ff05185
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
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# refmet-nomenclature-mapping

## Summary

Decodes RefMet-compliant lipid identifiers, including standardized common names like 'linolenic acid' and composite compounds like 'linoleyl palmitate', into their fatty acid residue lists with ambiguity counts using liputils' Lipid.refmet_residues() method.

## When to use

Use when lipid identifiers in a processed lipidomic table are RefMet-compliant textual names (e.g. 'FA(18:3)', 'octadecatrienoic acid') rather than the generic 'CLASS N:N/M:M' format, and you need per-residue carbon:double-bond strings.

## When NOT to use

- Identifiers already follow the generic 'CLASS N:N/M:M/../..' or 'CLASS(...)(...)' isomer-group format — use the standard Lipid.residues() parser instead of refmet_residues().
- You need to reject isobarically ambiguous lipids outright (drop_ambiguous=True on make_residues_table) rather than decode RefMet common names.

## Inputs

- RefMet-compliant lipid name strings (e.g. 'FA(18:3)', 'linolenic acid', 'linoleyl palmitate')
- pandas.DataFrame with lipid names as vertical index and samples as columns (optionally batch-translated to RefMet form)

## Outputs

- Tuple of (residue list of 'N:N' strings, ambiguity count) per lipid, e.g. (['18:3'], 1)
- Table (CSV/DataFrame) mapping lipid name to decoded residues and ambiguity counts

## How to apply

Instantiate a liputils Lipid object for each identifier and call .refmet_residues(), which parses any RefMet-compliant name into a tuple of (residue list, ambiguity count) — e.g. 'octadecatrienoic acid', 'linolenic acid' and 'FA(18:3)' each return (['18:3'], 1). For composite common names such as 'linoleyl palmitate', the method returns the component residues (['18:1', '16:0'], 1). If identifiers are not RefMet-compliant, first batch-translate them with RefMet's online translator before parsing. Collect name, residues, and ambiguity count per lipid into a pandas DataFrame (e.g. common_name_residues.csv) and validate against expected shorthand from the paper's examples.

## Related tools

- **liputils** (Provides the Lipid class and .refmet_residues() method that text-decode RefMet-compliant names into fatty acid residues with ambiguity counts) — https://github.com/Stemanz/liputils
- **RefMet** (Nomenclature standard and online batch translator used to convert non-compliant lipid IDs into RefMet-compliant names before parsing)
- **pandas** (Holds the lipidomic table (lipid names as index, samples in columns) and collects decoded residues into an output DataFrame)

## Examples

```
from liputils import Lipid; lip = Lipid("linoleyl palmitate"); lip.refmet_residues()  # -> (['18:1', '16:0'], 1)
```

## Evaluation signals

- Lipid('octadecatrienoic acid').refmet_residues() == (['18:3'], 1) and same result for 'linolenic acid' and 'FA(18:3)'.
- Lipid('linoleyl palmitate').refmet_residues() returns (['18:1', '16:0'], 1).
- Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1) as the parser sanity check.
- Every decoded (residues, ambiguity) tuple matches the expected shorthand reported in the paper's examples; mismatches are flagged.
- Ambiguity counts are reported for every decoded name (1 for unambiguous RefMet names).

## Limitations

- Non-RefMet-compliant identifiers must be batch-translated first via RefMet's online translator; liputils cannot parse arbitrary vendor annotations directly through refmet_residues().
- Ambiguity information must be handled downstream — ambiguous isomer groups can return multiple residues whose counts reflect uncertainty (each residue divided by its uncertainty count) unless rejected.
- No changelog found for the repository, so behavior may change between versions.

## Evidence

- [readme] By adhering to RefMet's nomenclature, any compliant lipid name will be properly managed by liputils's method ``refmet_residues(): "By adhering to RefMet's nomenclature, any compliant lipid name will be properly managed by liputils's method ``refmet_residues()"
- [readme] >>> lip1.refmet_residues()                                                
(['18:3'], 1): ">>> lip1.refmet_residues()                                                
(['18:3'], 1)"
- [readme] if unsure about your data format, you can try and batch-translate your lipid IDs with [RefMet's online translator]: "If unsure about your data format, you can try and batch-translate your lipid IDs with [RefMet's online translator]"
- [abstract] liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services: "There is no prerequisite data format, as liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
- [full_text] Add RefMet-compliant handling (refmet_residues) mapping common names such as 'linolenic acid' or 'FA(18:3)' to residue strings like '18:3'.: [evidence span withheld — anchor status: not_found]
