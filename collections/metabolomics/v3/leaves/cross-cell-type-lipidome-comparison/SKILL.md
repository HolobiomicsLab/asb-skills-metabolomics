---
name: cross-cell-type-lipidome-comparison
description: Use when comparing fatty acid residue abundances across cultured cell
  types (HepG2, HLC, PHH) or their media, to rank abundant residues (18:1, 16:0, 18:2,
  20:4) and assess hepatocyte-likeness of differentiated cells.
when_to_use_negative:
- The data are individual molecular lipid species and the goal is speciated lipidome
  analysis rather than residue-level aggregation — liputils deliberately discards
  the headgroup/species structure when counting residues
- The input table is already a residue-count table (output of make_residues_table),
  so re-running the extraction would be redundant
- Lipid identifiers are neither in CLASS N:N/M:M form nor RefMet-compliant and cannot
  be batch-translated, so residues cannot be parsed reliably
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: liputils
  role: Parses lipid string identifiers (Lipid class, refmet_residues(), make_residues_table(),
    GUI) to extract fatty acid residues and build the residue count table used for
    the cross-cell-type comparison
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (index of lipid names, samples in columns) and
    supports replicate aggregation, ranking and group-mean distance computation
- name: Python
  role: Runtime language for liputils and the pandas-based comparison workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:1b65079b972652971d28aee941a1f1d0825aef8303e72b3142acf5ea07c94ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-cell-type-lipidome-comparison

## Summary

Reprocess lipidomic tables from HepG2 cells, primary human hepatocytes (PHHs) and hepatocyte-like cells (HLCs) — and their culture media — with liputils to extract fatty acid residues, then rank residue abundances per cell type and compare group means to test whether HLCs resemble PHHs (e.g. for 16:0 and 18:1).

## When to use

Use when comparing fatty acid residue abundances across cultured cell types (HepG2, HLC, PHH) or their media, to rank abundant residues (18:1, 16:0, 18:2, 20:4) and assess hepatocyte-likeness of differentiated cells.

## When NOT to use

- The data are individual molecular lipid species and the goal is speciated lipidome analysis rather than residue-level aggregation — liputils deliberately discards the headgroup/species structure when counting residues
- The input table is already a residue-count table (output of make_residues_table), so re-running the extraction would be redundant
- Lipid identifiers are neither in CLASS N:N/M:M form nor RefMet-compliant and cannot be batch-translated, so residues cannot be parsed reliably

## Inputs

- Lipidomic abundance table (e.g. picomole or submolar-fraction values) for PHH, HLC and HepG2 samples and their culture media, with lipid names as row index and samples as columns
- RefMet-compliant or CLASS N:N/M:M-format lipid identifiers

## Outputs

- Per-cell-type mean fatty acid residue abundance table (residues × cell type)
- Ranked list of top HepG2-medium residues (18:1, 16:0, 18:2, 20:4 expected highest)
- HLC-vs-PHH distance flags for residues 16:0 and 18:1
- residue_abundance_summary.csv combining the summary table and verification flags

## How to apply

Load the residue abundance table into a pandas.DataFrame with lipid/residue names as the vertical index and samples (HLC, PHH, HepG2) in columns, as liputils requires. Run liputils' make_residues_table() (or the built-in GUI, which wraps it) to strip fatty acid moieties from individual molecular lipid identifiers, letting it drop non-numerical columns and unwanted strings like total lipid class counts; use drop_ambiguous if unresolved isomer mixtures should be rejected. Aggregate replicate columns per cell type to obtain mean residue abundances for each of the three cell types, then rank residues within the HepG2 medium samples to confirm 18:1, 16:0, 18:2 and 20:4 occupy the top positions. For residues 16:0 and 18:1, compute absolute distances between group means to flag whether HLC abundance is closer to PHH than to HepG2 — the paper reports this closeness even though the trend was not statistically significant, so treat it as a descriptive pattern check. Write the per-cell-type abundance summary together with the verification flags (top-residue ranking, HLC-vs-PHH distance) to a summary CSV.

## Related tools

- **liputils** (Parses lipid string identifiers (Lipid class, refmet_residues(), make_residues_table(), GUI) to extract fatty acid residues and build the residue count table used for the cross-cell-type comparison) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (index of lipid names, samples in columns) and supports replicate aggregation, ranking and group-mean distance computation)
- **Python** (Runtime language for liputils and the pandas-based comparison workflow)

## Examples

```
from liputils import make_residues_table; res = make_residues_table(df)
```

## Evaluation signals

- Top-abundance check: residues 18:1, 16:0, 18:2 and 20:4 rank highest in the HepG2 medium samples after mean aggregation
- Pattern check: for 16:0 and 18:1, the absolute distance between HLC and PHH group means is smaller than the distance between HLC and HepG2 group means (expected non-statistically-significant trend)
- make_residues_table() output schema: numerical residue rows in the index, samples in columns, unwanted strings like 'total', 'fc', 'tc' removed
- Ambiguity handling: residues() returns a tuple (residue list, ambiguity count); ambiguity count > 0 flags lipids where isomers were pooled unless drop_ambiguous=True was set

## Limitations

- The HLC-closer-to-PHH pattern for 16:0 and 18:1 was not statistically significant — it is a descriptive trend, not a confirmed difference
- liputils pools residues from ambiguous isomer mixtures (e.g. TAG 48:2 total with multiple composition sets) and divides each residue by its uncertainty; use drop_ambiguous=True to reject such lipids instead
- Unwanted index entries such as total lipid class counts must be cleaned up (cleanup/unwanted parameters, defaults ['total', 'fc', 'tc']) or they will corrupt the residue table

## Evidence

- [results] HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant: "HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant"
- [methods] make_residues_table drops non-numerical columns and unwanted lipids: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"
- [readme] Lipidomics data loaded in pandas.DataFrame with lipid names as vertical index and samples in columns: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] drop_ambiguous parameter rejects non-unambiguous lipids: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [results] liputils reprocessed PHH, HepG2 and HLC lipidomic data: "by using liputils, we reprocessed data from primary human hepatocytes (PHHs), HepG2 cells and hepatocyte-like cells (HLCs)"
