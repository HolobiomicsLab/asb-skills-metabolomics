---
name: lipid-residue-extraction
description: Use when you have a processed lipidomics table (one lipid identifier
  per row, one sample per column, e.g. from pandas.read_excel) and need to reprocess
  analyte annotations into per-sample fatty acid residue abundances rather than individual
  molecular lipid levels.
when_to_use_negative:
- The input is raw mass-spectrometry data (mzML, centroided peaks) rather than a processed,
  final lipidomic data table — liputils parses textual lipid identifiers, not spectra.
- The analysis goal is tracking individual molecular lipid species or ratios rather
  than aggregate fatty acid residue composition (residue extraction deliberately discards
  the lipid class context).
- Lipid identifiers cannot be interpreted by the parser (neither generic 'CLASS N:N/M:M'
  format, isomer groups, nor RefMet names) and cannot be batch-translated with RefMet's
  online translator — residues may be silently wrong or dropped.
edam_operation: http://edamontology.org/operation_3287
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Core module providing the Lipid class, make_residues_table(), refmet_residues(),
    saturated() and max_carbon() to extract fatty acid residues from lipid identifiers
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Provides the DataFrame input format (vertical index of lipid names, samples
    in columns) that make_residues_table() consumes and returns
- name: RefMet
  role: Reference nomenclature; non-compliant IDs can be batch-translated with RefMet's
    online translator so refmet_residues() can manage them
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:c7ad3ec9c8c90ed2834bb48976ddbeaea3922cc58b7aa8d19d605634c8cd4c1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - RefMet
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_3287
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipid-residue-extraction

## Summary

Extract individual fatty acid residues (e.g. 18:1, 20:4, 26:0) from processed lipidomic data tables using the liputils Python module, which parses lipid string identifiers from RefMet-compliant or generic 'CLASS N:N/M:M/...' formats into residue counts in submolar fractions.

## When to use

Use when you have a processed lipidomics table (one lipid identifier per row, one sample per column, e.g. from pandas.read_excel) and need to reprocess analyte annotations into per-sample fatty acid residue abundances rather than individual molecular lipid levels.

## When NOT to use

- The input is raw mass-spectrometry data (mzML, centroided peaks) rather than a processed, final lipidomic data table — liputils parses textual lipid identifiers, not spectra.
- The analysis goal is tracking individual molecular lipid species or ratios rather than aggregate fatty acid residue composition (residue extraction deliberately discards the lipid class context).
- Lipid identifiers cannot be interpreted by the parser (neither generic 'CLASS N:N/M:M' format, isomer groups, nor RefMet names) and cannot be batch-translated with RefMet's online translator — residues may be silently wrong or dropped.

## Inputs

- pandas.DataFrame lipidomics table with lipid names as vertical index and samples as columns (e.g. loaded with pandas.read_excel using index_col=1, header=2)
- Lipid identifiers in generic 'CLASS N:N/M:M/../..' format, isomer-group format 'CLASS(N:N/M:M)(...)', or fully RefMet-compliant names (e.g. 'linolenic acid', 'FA(18:3)')
- Optional list of unwanted index strings (default: ['total', 'fc', 'tc'])

## Outputs

- pandas DataFrame of residue counts in submolar fractions, residue names (e.g. 18:1, 20:4) as index and samples as columns
- Optionally a residue/ambiguity tuple from Lipid.residues() or Lipid.refmet_residues(), e.g. (['18:1', '16:0'], 1) for 'linoleyl palmitate'

## How to apply

Load the lipidomics data into a pandas.DataFrame with lipid names as the vertical index and samples as columns (use dataframe.T if needed). Call liputils' make_residues_table(), which drops non-numerical columns and trims unwanted index entries such as total lipid class counts (default unwanted list: 'total', 'fc', 'tc', further specifiable via the unwanted parameter). For non-RefMet-compliant analyte names of the form 'CLASS N:N/M:M/...', the function must be called with liptype=None so the Lipid class's text-based parser is used; for isomer groups like 'TAG 48:2 total (...)(...)', residues() returns all residues plus an ambiguity count, and drop_ambiguous=True rejects ambiguous lipids or divides each residue by its uncertainty when False. Filter the residue index with the saturated() and max_carbon() helper functions (e.g. keep residues up to 30 carbon atoms as in the ST000668 dietary re-analysis). Save the returned residues DataFrame (residue names as index, samples as columns) as the deliverable, e.g. a tab-separated residues_table.csv.

## Related tools

- **liputils** (Core module providing the Lipid class, make_residues_table(), refmet_residues(), saturated() and max_carbon() to extract fatty acid residues from lipid identifiers) — https://github.com/Stemanz/liputils
- **pandas** (Provides the DataFrame input format (vertical index of lipid names, samples in columns) that make_residues_table() consumes and returns)
- **RefMet** (Reference nomenclature; non-compliant IDs can be batch-translated with RefMet's online translator so refmet_residues() can manage them)

## Examples

```
from liputils import make_residues_table; res = make_residues_table(df, liptype=None, drop_ambiguous=True); res.to_csv('residues_table.csv', sep='\t')
```

## Evaluation signals

- The output residue index contains parsed fatty acid residues (e.g. 18:1, 20:4) rather than original lipid class identifiers or total-count rows ('total', 'fc', 'tc' removed).
- Documented parser examples reproduce exactly: Lipid('PG 18:1/20:1').residues() returns (['18:1', '20:1'], 1); the TAG 48:2 total isomer group returns 9 residues with ambiguity 3; with drop_ambiguous=True it returns ([], 0).
- refmet_residues() decodes 'octadecatrienoic acid', 'linolenic acid' and 'FA(18:3)' each to (['18:3'], 1), and 'linoleyl palmitate' to (['18:1', '16:0'], 1).
- Filter invariant: if max_carbon(x, 30) was applied as in the ST000668 re-analysis, every residue in the index has 30 carbons or fewer; saturated('12:0') is True and max_carbon('21:3', 16) is False.
- The saved output table (e.g. residues_table.csv) exists, uses the expected separator (tab-separated for residues_table.csv), and has samples as columns.

## Limitations

- Isobaric/isomer ambiguity: unresolved lipid isomer groups carry an ambiguity count; drop_ambiguous=True discards them entirely, otherwise each residue is divided by its uncertainty, diluting absolute values.
- make_residues_table() defaults to RefMet nomenclature — non-RefMet-compliant tables must explicitly pass liptype=None or identifiers will not be parsed correctly.
- Unwanted index entries are only removed if cleanup=True and only for strings in the 'unwanted' list (default ['total', 'fc', 'tc']); other non-lipid rows must be specified manually.
- The amount-to-molecule conversion assumes picomoles by default; other units must be given explicitly via the unit parameter.

## Evidence

- [abstract] liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services: "There is no prerequisite data format, as liputils extracts residues from RefMet-compliant textual identifiers and from annotations of other commercially available services"
- [methods] make_residues_table drops non-numerical columns and trims unwanted lipids: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [readme] GUI wraps make_residues_table for one-step residue conversion: "The GUI acts as a wrapper for make_residues_table()`` enabling a fast two-click conversion of any source table."
- [readme] PG 18:1/20:1 residues example: ">>> l.residues()
(['18:1', '20:1'], 1)"
- [readme] saturated and max_carbon residue filters: "it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
- [results] results: residues up to 30 carbon atoms: "We included in our analysis all residues up 30 carbon atoms in length"
