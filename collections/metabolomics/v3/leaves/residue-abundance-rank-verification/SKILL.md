---
name: residue-abundance-rank-verification
description: Use when lipidomic abundance tables from hepatocyte cell models (PHH,
  HLC, HepG2 and their media) must be converted to fatty acid residue counts and checked
  for cell-type-specific rank patterns.
when_to_use_negative:
- Input identifiers are not parseable as 'CLASS N:N/M:M' or RefMet-compliant names
  and cannot be batch-translated with RefMet's online translator — liputils residue
  extraction will fail
- The question concerns individual molecular lipid species rather than aggregate fatty
  acid residue patterns (e.g. a PCA on individual molecular lipids was also performed
  in the paper)
- The table contains non-numerical columns or total lipid class counts and you are
  not using make_residues_table()'s cleanup/unwanted handling
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: liputils
  role: Python module used to parse lipid identifiers and extract fatty acid moieties
    into a residue count table via make_residues_table() or the built-in GUI
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics data as a DataFrame (lipid names as vertical index, samples
    in columns) and supports replicate aggregation, ranking and distance computation
- name: Python
  role: Language in which the reprocessing, ranking and distance-flagging workflow
    is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:30f340c877f76e11b07e489bd87d5e4579ceba27fe9a391506d341cb8bed485a
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# residue-abundance-rank-verification

## Summary

Reprocesses cultured-cell lipidomic data with liputils to extract fatty acid residue abundances, then verifies cell-type residue patterns: that 18:1, 16:0, 18:2 and 20:4 rank highest in HepG2 culture medium and that HLC 16:0/18:1 levels sit closer to PHHs than to HepG2.

## When to use

Use when lipidomic abundance tables from hepatocyte cell models (PHH, HLC, HepG2 and their media) must be converted to fatty acid residue counts and checked for cell-type-specific rank patterns.

## When NOT to use

- Input identifiers are not parseable as 'CLASS N:N/M:M' or RefMet-compliant names and cannot be batch-translated with RefMet's online translator — liputils residue extraction will fail
- The question concerns individual molecular lipid species rather than aggregate fatty acid residue patterns (e.g. a PCA on individual molecular lipids was also performed in the paper)
- The table contains non-numerical columns or total lipid class counts and you are not using make_residues_table()'s cleanup/unwanted handling

## Inputs

- Lipidomics abundance table (e.g. .xlsx/CSV) with RefMet-compliant or 'CLASS N:N/M:M' lipid identifiers as row index and cell-type samples (HLC, PHH, HepG2, and culture media) as columns, with replicate columns
- liputils-installed Python environment

## Outputs

- Residue-by-sample abundance table (submolar fractions) from make_residues_table()
- Per-cell-type mean residue abundance summary with HepG2-medium top-rank flags and HLC-vs-PHH distance flags
- residue_abundance_summary.csv

## How to apply

Load the lipidomics table into a pandas.DataFrame with lipid names as the vertical index and samples as columns, then transform it with liputils' make_residues_table() (or the built-in GUI wrapper) to obtain a residue table; drop_ambiguous can be used to reject non-unambiguous lipids. Aggregate replicate columns per cell type (HLC, PHH, HepG2) to group means, then rank residues within the HepG2 medium samples and confirm 18:1, 16:0, 18:2 and 20:4 occupy the top positions. As a secondary pattern check, for residues 16:0 and 18:1 compute absolute distances between group means and flag whether HLC abundance is nearer to PHH than to HepG2 — the paper reports this trend was not statistically significant. Write the per-cell-type residue abundance summary together with the per-pattern verification flags to residue_abundance_summary.csv.

## Related tools

- **liputils** (Python module used to parse lipid identifiers and extract fatty acid moieties into a residue count table via make_residues_table() or the built-in GUI) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics data as a DataFrame (lipid names as vertical index, samples in columns) and supports replicate aggregation, ranking and distance computation)
- **Python** (Language in which the reprocessing, ranking and distance-flagging workflow is implemented)

## Examples

```
from liputils import make_residues_table; res = make_residues_table(df); hepg2_med = res.filter(like='HepG2_medium'); print(hepg2_med.mean(axis=1).sort_values(ascending=False).head(4))
```

## Evaluation signals

- Ranking HepG2 medium residues by mean abundance places 18:1, 16:0, 18:2 and 20:4 at the top positions
- For 16:0 and 18:1, |mean_HLC − mean_PHH| < |mean_HLC − mean_HepG2| (HLC closer to PHH), matching the reported non-statistically-significant trend
- The output residue table has residues as the index and one aggregated column per cell type, with non-numerical columns and total lipid class counts removed (make_residues_table defaults: unwanted = ["total", "fc", "tc"])
- residue_abundance_summary.csv exists and contains both the abundance summary and the per-pattern verification flags
- All lipid identifiers in the input table were parsed (each Lipid.residues() call returned residue lists and an ambiguity count), or ambiguous lipids were explicitly dropped with drop_ambiguous=True

## Limitations

- The HLC-closer-to-PHH pattern for 16:0 and 18:1 was not statistically significant, so it should be reported as a trend, not a confirmed difference
- liputils parses textual identifiers only ('CLASS N:N/M:M' or RefMet-compliant names); lipids with unresolved isomer ambiguities must either be kept with an ambiguity count or rejected via drop_ambiguous
- liputils operates on processed, final lipidomic data and does not perform mass spectrometry processing itself

## Evidence

- [other] Reprocessing the cultured-cell lipidomic data with liputils revealed that fatty acids 18:1, 16:0, 18:2 and 20:4 were the most abundant residues in the HepG2 culture medium.: [evidence span withheld — anchor status: not_found]
- [results] HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant: "HLCs were much closer to PHHs with respect to 16:0 and 18:1 residues, higher in HepG2, even though the trend was not statistically significant"
- [readme] Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
- [readme] The GUI acts as a wrapper for make_residues_table(): "The GUI acts as a wrapper for make_residues_table()"
- [results] by using liputils, we reprocessed data from primary human hepatocytes (PHHs), HepG2 cells and hepatocyte-like cells (HLCs): "by using liputils, we reprocessed data from primary human hepatocytes (PHHs), HepG2 cells and hepatocyte-like cells (HLCs)"
