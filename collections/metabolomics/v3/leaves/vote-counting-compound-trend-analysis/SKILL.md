---
name: vote-counting-compound-trend-analysis
description: Use when a metabolomics meta-analysis dataset (csv/xls/xlsx/txt of compound
  name, trend/behaviour, and reference) needs qualitative vote-counting and visual
  verification that reports agree on up/down regulation.
when_to_use_negative:
- You have P-values and fold-changes and need quantitative combined significance/effect
  size — use amanida's quantitative mode (compute_amanida) or volcano_plot instead
  of vote-counting.
- The dataset lacks trend/behaviour labels (only raw intensities or spectral data)
  — vote-counting requires identifier and trend labels per report.
- You need error/variance-based meta-analysis of raw or integral data with standard
  deviations; amanida explicitly targets non-integral data where SD is not disclosed.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Performs the qualitative vote-counting (amanida_vote), the mixed-type explore_plot
    visualization, and identifier harmonization via check_names
  repo: https://github.com/mariallr/amanida
- name: R
  role: 'Runtime environment: amanida is an R package installed from CRAN or GitHub
    via devtools'
  repo: https://github.com/mariallr/amanida
- name: webchem
  role: Converts chemical-name/InChI/InChIKey/SMILES identifiers to PubChem IDs during
    check_names before vote-counting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:613905640dd99ee540997b305060f58021ad82df5f185f5e58395739f19d11b6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  repo_url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# vote-counting-compound-trend-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Qualitative meta-analysis skill that performs compound vote-counting (+1 up, −1 down, 0 no trend, divided by number of reports) and renders a mixed-type explore plot to check whether metabolomics reports agree on trend direction, e.g. Hippuric acid and Phenol in the CRC urinary sample_data.

## When to use

Use when a metabolomics meta-analysis dataset (csv/xls/xlsx/txt of compound name, trend/behaviour, and reference) needs qualitative vote-counting and visual verification that reports agree on up/down regulation.

## When NOT to use

- You have P-values and fold-changes and need quantitative combined significance/effect size — use amanida's quantitative mode (compute_amanida) or volcano_plot instead of vote-counting.
- The dataset lacks trend/behaviour labels (only raw intensities or spectral data) — vote-counting requires identifier and trend labels per report.
- You need error/variance-based meta-analysis of raw or integral data with standard deviations; amanida explicitly targets non-integral data where SD is not disclosed.

## Inputs

- csv, xls/xlsx or txt dataset with columns Id (compound name), Trend (up- or down-regulated behaviour), Reference — readable via amanida_read(mode = "qual")
- Bundled example dataset sample_data (CRC urinary volatilome/metabolome from Mallafré et al., Cancers 2021, 13(11), 2534)

## Outputs

- S4 object with vote-counting table accessed via vote_results@vote
- explore plot (mixed-type: vote-counting plus reports divided by trend) e.g. saved as explore_plot.png
- Subset table of compounds with more than one report and consistent trend (e.g. significant_compounds.csv with Hippuric acid, Phenol)

## How to apply

Read the dataset with `amanida_read` in mode = "qual" using coln = c("Compound Name", "Behaviour", "References"); optionally harmonize identifiers via `check_names` (webchem/PubChem, per the Metabolites Merging Strategy). Run `amanida_vote(data_votes)` to compute vote-counting, where votes are +1 for up-regulation, -1 for down-regulation and 0 for no trend, total votes divided by the number of reports. Render `explore_plot` with type = "mix" and a counts cut-off to subset compounds with more than one report and reveal discrepancies (reports up- and down-regulated). Compounds whose rows all show the same trend are consistent; compounds appearing in both directions are discrepant. Optionally wrap everything with `amanida_report` using a votecount_lim minimum and analysis_type = "qual".

## Related tools

- **amanida** (Performs the qualitative vote-counting (amanida_vote), the mixed-type explore_plot visualization, and identifier harmonization via check_names) — https://github.com/mariallr/amanida
- **R** (Runtime environment: amanida is an R package installed from CRAN or GitHub via devtools) — https://github.com/mariallr/amanida
- **webchem** (Converts chemical-name/InChI/InChIKey/SMILES identifiers to PubChem IDs during check_names before vote-counting)

## Examples

```
library(amanida); data("sample_data"); explore_plot(sample_data, type = "mix", counts = 1)
```

## Evaluation signals

- Hippuric acid and Phenol appear as compounds with more than one report and statistical significance, with all reports showing the same trend (consistent).
- Vote-counting table vote_results@vote contains only values in [-1, 1]: total votes divided by number of reports, with votes +1 (up), -1 (down), 0 (no trend).
- explore_plot with type = "mix" and counts cut-off shows discrepant compounds (both up- and down-regulated reports); consistent compounds should NOT appear as discrepant.
- Input was read with the exact qual-mode column order (Id, Trend, Reference); mis-ordered coln would mislabel the Behaviour column.

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for a meta-analysis, so vote-counting may rest on identifier plus trend label alone.
- Duplicate compound names across studies can distort votes unless check_names (PubChem ID harmonization per the Metabolites Merging Strategy) is run first.
- explore_plot type = "mix" requires a counts cut-off (e.g. counts = 1); compounds below the cut-off are dropped from the discrepancy view.

## Evidence

- [readme] Compound vote-counting: votes are +1 for up-regulation, -1 for down-regulation and 0 if no trend. The total votes are divided by the number of reports.: [evidence span withheld — anchor status: external_doc]
- [readme] type = "mix": subset the data by a cut-off value indicated by the counts parameter and show compounds with discrepancies (reports up-regulated and down-regulated): [evidence span withheld — anchor status: external_doc]
