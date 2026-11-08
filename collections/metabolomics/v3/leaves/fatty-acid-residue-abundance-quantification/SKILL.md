---
name: fatty-acid-residue-abundance-quantification
description: Use when lipidomic data annotated with individual molecular lipid identifiers
  (RefMet-compliant or CLASS N:N/M:M format) must be converted into fatty-acid residue
  counts and ranked/compared across sample groups, e.g. cell-type residue patterns.
when_to_use_negative:
- The analysis targets intact individual molecular lipid species or lipid-class totals
  rather than residue-level carbon-chain tracking — make_residues_table() trims non-numerical
  columns and total lipid class counts by design
- Lipid identifiers are non-standard and not translatable to RefMet nomenclature,
  so residues cannot be parsed unambiguously and cannot simply be dropped
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Python module that parses lipid string identifiers and extracts fatty acid
    moieties; make_residues_table() converts the whole lipidomic table into a residue
    count table
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the input lipidomic data as a DataFrame (lipid names as index, samples
    as columns) and the resulting residue table; used for per-cell-type aggregation
    and ranking
- name: Python
  role: Runtime environment in which liputils and pandas are used, optionally via
    liputils' built-in GUI or a Jupyter Notebook workflow
  repo: https://github.com/Stemanz/liputils
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:b45d8d5101c672853489ad80abe7e7326b71cfc1fb0989eb0976c1618e36b75c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fatty-acid-residue-abundance-quantification

## Summary

Quantify fatty acid residue abundances (e.g. 18:1, 16:0, 18:2, 20:4) by reprocessing lipidomic tables of complex lipids with liputils, then compare residue profiles across cell types (HLC, PHH, HepG2) to test residue-pattern findings.

## When to use

Use when lipidomic data annotated with individual molecular lipid identifiers (RefMet-compliant or CLASS N:N/M:M format) must be converted into fatty-acid residue counts and ranked/compared across sample groups, e.g. cell-type residue patterns.

## When NOT to use

- The analysis targets intact individual molecular lipid species or lipid-class totals rather than residue-level carbon-chain tracking — make_residues_table() trims non-numerical columns and total lipid class counts by design
- Lipid identifiers are non-standard and not translatable to RefMet nomenclature, so residues cannot be parsed unambiguously and cannot simply be dropped

## Inputs

- pandas.DataFrame of lipidomic data with lipid names as vertical index and samples in columns (amounts typically in picomoles)
- Lipid identifiers in RefMet-compliant naming, 'CLASS N:N/M:M/..' format, or mass-isomer annotations from commercial services

## Outputs

- Residue abundance table (pandas DataFrame) with fatty acid residues as index and samples/columns as residues in submolar fractions
- Per-cell-type mean residue abundance summary with ranking and HLC-vs-PHH/HepG2 distance flags (e.g. residue_abundance_summary.csv)

## How to apply

Load the lipidomic table into a pandas.DataFrame with lipid names as the vertical index and samples as columns, then run liputils' make_residues_table() (or its GUI wrapper) to strip residues from each lipid identifier; ambiguous lipids such as TAG 48:2 total isomer mixtures can be handled by dividing residues by uncertainty or dropped via drop_ambiguous=True. Aggregate replicate sample columns per cell type (HLC, PHH, HepG2, and their media) to obtain mean residue abundances per residue. Rank residues within the group of interest (e.g. HepG2 medium) by abundance and verify expected top residues (18:1, 16:0, 18:2, 20:4). For targeted residues (16:0, 18:1), compute absolute distances between group means to test whether HLC abundance is closer to PHH than to HepG2, mirroring the paper's reported non-significant trend. Optionally filter residues with saturated() and max_carbon() (e.g. all residues up to 30 carbons) to restrict the analysis.

## Related tools

- **liputils** (Python module that parses lipid string identifiers and extracts fatty acid moieties; make_residues_table() converts the whole lipidomic table into a residue count table) — https://github.com/Stemanz/liputils
- **pandas** (Holds the input lipidomic data as a DataFrame (lipid names as index, samples as columns) and the resulting residue table; used for per-cell-type aggregation and ranking)
- **Python** (Runtime environment in which liputils and pandas are used, optionally via liputils' built-in GUI or a Jupyter Notebook workflow) — https://github.com/Stemanz/liputils

## Examples

```
from liputils import make_residues_table; res = make_residues_table(df)
```

## Evaluation signals

- Ranking check: residues 18:1, 16:0, 18:2 and 20:4 occupy the top positions when HepG2 medium samples are ranked by abundance
- Distance check: for residues 16:0 and 18:1, absolute distances between group means show HLC abundance nearer to PHH than to HepG2 (trend may be statistically non-significant)
- Residue table schema: vertical index contains RefMet-compliant residue names (e.g. '18:1', '16:0') and non-numerical columns and total lipid class counts have been dropped
- Ambiguity accounting: ambiguous lipids either contribute all isomer residues with the correct ambiguity count (e.g. 3 for a TAG 48:2 total) or are excluded with drop_ambiguous=True
- Units consistency: residue amounts remain in the original table's units (submolar fractions by default) unless absolute_amount=True is set

## Limitations

- Lipid isomer ambiguities (e.g. 'TAG 48:2 total (14:0/16:0/18:2)(...)') inflate residue counts per lipid; residues must be divided by the uncertainty count or such lipids dropped with drop_ambiguous=True
- make_residues_table() automatically trims unwanted strings (default ['total', 'fc', 'tc']) and non-numerical columns, which may silently remove rows/columns the user intended to keep
- Comparative trends such as HLC 16:0/18:1 closeness to PHHs were not statistically significant in the source study, so distance-based flags are indicative rather than confirmatory

## Evidence

- [methods] The accepted format is a vertical index with lipid names, and samples in column.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] make_residues_table() drops non-numerical columns and unwanted lipids: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [results] HLCs closer to PHHs for 16:0 and 18:1 residues: "HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant"
- [readme] drop_ambiguous parameter rejects non unambiguous lipids: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [results] liputils reprocessed PHH, HepG2 and HLC lipidomic data: "by using liputils, we reprocessed data from primary human hepatocytes (PHHs), HepG2 cells and hepatocyte-like cells (HLCs)"
