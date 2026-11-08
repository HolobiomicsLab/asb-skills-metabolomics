---
name: residue-abundance-percent-change-analysis
description: Use when a lipidomics dataset with individual molecular lipid annotations
  (e.g., from Metabolomics Workbench) must be converted to fatty acid residue counts
  and compared as per-residue percentage changes between experimental groups or conditions.
when_to_use_negative:
- Input lipid identifiers cannot be made RefMet-compliant or parsed by liputils' Lipid
  class (e.g., non-standard class N:N/M:M formats), so residue extraction would silently
  drop rows
- The goal is analysis of individual molecular lipid species rather than aggregate
  fatty-acid residue makeup, where make_residues_table() collapses isomer/residue
  information
- The dataset contains a large fraction of isobarically ambiguous lipids and drop_ambiguous=True
  would discard too much of the index to support group comparisons
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: liputils
  role: Extracts fatty acid moieties from individual molecular lipid identifiers via
    Lipid parsing, make_residues_table(), refmet_residues(), and drop_ambiguous; provides
    saturated() and max_carbon() filters
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (lipid names as vertical index, samples in columns)
    that make_residues_table() consumes and returns the residue count DataFrame
- name: Python
  role: Execution environment for the liputils workflow, including a step-by-step
    Jupyter Notebook in the Supplemental methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:18602ace9b38f59a28fbc8bdd3d37baee058664fbca4341c9b381e940669a2e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# residue-abundance-percent-change-analysis

## Summary

Reprocesses lipidomic tables into fatty-acid residue abundance tables with liputils and computes per-residue percentage changes across dietary conditions, as demonstrated on Metabolomics Workbench study ST000668 (human plasma dietary intervention).

## When to use

Use when a lipidomics dataset with individual molecular lipid annotations (e.g., from Metabolomics Workbench) must be converted to fatty acid residue counts and compared as per-residue percentage changes between experimental groups or conditions.

## When NOT to use

- Input lipid identifiers cannot be made RefMet-compliant or parsed by liputils' Lipid class (e.g., non-standard class N:N/M:M formats), so residue extraction would silently drop rows
- The goal is analysis of individual molecular lipid species rather than aggregate fatty-acid residue makeup, where make_residues_table() collapses isomer/residue information
- The dataset contains a large fraction of isobarically ambiguous lipids and drop_ambiguous=True would discard too much of the index to support group comparisons

## Inputs

- Lipidomics table (pandas.DataFrame) with lipid names as vertical index and samples as columns, e.g., Metabolomics Workbench study ST000668
- Lipid identifiers in RefMet-compliant format or batch-translated via RefMet's online translator
- Sample-to-condition mapping (e.g., Unsaturated fats, Saturated fats, Overfeeding)

## Outputs

- Residue count table (submolar fractions) produced by make_residues_table()
- Filtered residue index restricted to residues up to 30 carbons
- Per-residue percentage-change table versus baseline condition with p-values (e.g., 26:0 change, p = 0.018)

## How to apply

Download the dietary intervention dataset (e.g., Metabolomics Workbench ST000668, doi:10.21228/M85P6M) and load it into a pandas.DataFrame with lipid names as the vertical index and samples as columns, batch-translating lipid IDs with RefMet's online translator if they are not RefMet-compliant. Convert identifiers to residue counts using liputils' make_residues_table() with drop_ambiguous=True to reject isobarically ambiguous lipids, then filter the residue index with the saturated() and max_carbon() functions to keep only residues up to 30 carbon atoms in length. Group residue abundances (in submolar fractions) by dietary condition (Unsaturated fats, Saturated fats, Overfeeding) and compute per-residue percentage changes versus the Unsaturated fats baseline. Validate by confirming statistically significant changes match published values (residue 26:0 increased under Saturated fats and Overfeeding, p = 0.018) and that the residue index contains only residues of 30 carbons or fewer.

## Related tools

- **liputils** (Extracts fatty acid moieties from individual molecular lipid identifiers via Lipid parsing, make_residues_table(), refmet_residues(), and drop_ambiguous; provides saturated() and max_carbon() filters) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (lipid names as vertical index, samples in columns) that make_residues_table() consumes and returns the residue count DataFrame)
- **Python** (Execution environment for the liputils workflow, including a step-by-step Jupyter Notebook in the Supplemental methods)

## Examples

```
from liputils import make_residues_table, saturated, max_carbon; res = make_residues_table(df, drop_ambiguous=True)
```

## Evaluation signals

- The reproduced 26:0 percentage-change table yields a statistically significant increase under Saturated fats and Overfeeding versus Unsaturated fats with p = 0.018 as reported
- The residue index contains only residues of 30 carbons or fewer (validated via max_carbon filtering)
- make_residues_table() was called with drop_ambiguous=True and no ambiguous isobaric lipids remain in the residue table
- Residue abundances are expressed in submolar fractions and grouped by dietary condition before computing percentages versus the Unsaturated fats baseline
- PCA on residues and individual molecular lipids of the original data can be reproduced to confirm group separation consistent with the published analysis

## Limitations

- Isobarically ambiguous lipid annotations must be rejected with drop_ambiguous=True, discarding those lipids entirely rather than resolving them
- Only residues up to 30 carbon atoms in length were included in the analysis; longer residues are excluded
- The reanalysis showed no dramatic overall dietary effect on residue makeup; only residue 26:0 was statistically significant, so conclusions from small per-residue changes should be cautious
- Non-numerical columns and unwanted index strings (defaults ['total', 'fc', 'tc']) are dropped by cleanup, which may need customization via the 'unwanted' parameter

## Evidence

- [results] Saturated fats and Overfeeding boosted long-chain saturated residues, but only residue 26:0 showed a statistically significant change (Fig. 3A): [evidence span withheld — anchor status: not_found]
- [methods] it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [results] We included in our analysis all residues up 30 carbon atoms in length: "We included in our analysis all residues up 30 carbon atoms in length"
- [readme] it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard: "To focus on particular residues, it is possible to mix saturated() and max_carbon() to dictate which residues to keep in the index and which to discard"
- [readme] Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column.: "Lipidomics data should be loaded in a pandas.DataFrame table. The accepted format is a vertical index with lipid names, and samples in column."
