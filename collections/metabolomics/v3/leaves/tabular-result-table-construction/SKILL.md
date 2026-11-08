---
name: tabular-result-table-construction
description: Use when a weighted meta-analysis (e.g. amanida's Fisher-method P-value
  combination) has produced per-compound statistics held in an S4 result object and
  they must be exported as a structured table such as sample_combined_pvalues.csv.
when_to_use_negative:
- Raw per-study data with disclosed standard deviations are available — then conventional
  meta-analysis tools using effect sizes and variance apply, not overall-results-only
  tables
- The input is qualitative only (identifier plus trend labels with no P-values or
  fold-changes) and only vote-counting via amanida_vote is required
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: amanida
  role: Produces the per-compound meta-analysis tables (compute_amanida @stat/@vote,
    amanida_report) that are assembled into the result table
  repo: github.com/mariallr/amanida
- name: R
  role: Environment in which the S4 result slots are accessed and the table is assembled
    and written to CSV
- name: webchem
  role: Supports harmonizing compound identifiers to PubChem ID before duplicate checking,
    so table rows are keyed consistently
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:7237b16488d37b5d63d22d33c7b0c9dbadab92649d2bd516826652f147ca394c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tabular-result-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles per-compound meta-analysis outputs (combined P-values, fold-change averages, and vote-counts) produced by the amanida package into flat tabular result files, such as a per-compound combined-P-value CSV, so they can be shared or downstream-consumed.

## When to use

Use when a weighted meta-analysis (e.g. amanida's Fisher-method P-value combination) has produced per-compound statistics held in an S4 result object and they must be exported as a structured table such as sample_combined_pvalues.csv.

## When NOT to use

- Raw per-study data with disclosed standard deviations are available — then conventional meta-analysis tools using effect sizes and variance apply, not overall-results-only tables
- The input is qualitative only (identifier plus trend labels with no P-values or fold-changes) and only vote-counting via amanida_vote is required

## Inputs

- amanida S4 result object from compute_amanida with @stat and @vote tables
- sample_data dataset (compounds from the colorectal cancer urine volatilome/metabolome systematic review)
- Input csv/xls/xlsx/txt dataset with columns Compound Name, P-value, Fold-change, N total, References

## Outputs

- Per-compound combined-P-value table written as sample_combined_pvalues.csv
- Quantitative meta-analysis table accessible via amanida_result@stat
- Vote-counting table accessible via amanida_result@vote

## How to apply

After running amanida_read on the input dataset (csv/xls/xlsx/txt with columns in the fixed order Id, P-value, Fold-change, N total, References) and compute_amanida on the loaded data, the results are returned as an S4 object whose slots separate the quantitative and qualitative tables. Assemble the quantitative per-compound rows from the combined P-value statistics by accessing `amanida_result@stat`, and the qualitative vote-counting rows from `amanida_result@vote`, preserving the per-compound granularity (one row per compound/metabolite identifier). If comp.inf = TRUE was used and check_names was run first, rows are keyed by harmonized PubChem ID with retrieved descriptors, so duplicate identifiers must be checked before assembling the table. Finally write the assembled per-compound table out as CSV (e.g. sample_combined_pvalues.csv); alternatively, amanida_report consolidates reading, computation and tabular reporting in a single step with parameters like pvalue_cutoff = 0.05, fc_cutoff, and votecount_lim.

## Related tools

- **amanida** (Produces the per-compound meta-analysis tables (compute_amanida @stat/@vote, amanida_report) that are assembled into the result table) — github.com/mariallr/amanida
- **R** (Environment in which the S4 result slots are accessed and the table is assembled and written to CSV)
- **webchem** (Supports harmonizing compound identifiers to PubChem ID before duplicate checking, so table rows are keyed consistently)

## Examples

```
amanida_result <- compute_amanida(datafile, comp.inf = F); write.csv(amanida_result@stat, "sample_combined_pvalues.csv")
```

## Evaluation signals

- Output CSV contains one row per compound from the input dataset, keyed by a unique identifier (PubChem ID when comp.inf = TRUE)
- Each combined P-value is reproducible from the per-study P-values and study sizes (n) via the weighted Fisher's-method combination with gamma-distributed non-integral weights
- Quantitative and qualitative results are correctly separated: amanida_result@stat for the adapted meta-analysis and amanida_result@vote for vote-counting (+1 up, -1 down, 0 no trend, divided by number of reports)
- No duplicate compound identifiers remain in the assembled table after the Metabolites Merging Strategy harmonization step
- File is readable CSV with columns consistent with the amanida_report column_id ordering (Compound Name, P-value, Fold-change, N total, References for quantitative inputs)

## Limitations

- Valid only for overall (non-integral) results — P-value and fold-change without standard deviations; metabolomics studies rarely disclose the full statistical information needed otherwise
- Duplicate identifiers distort the per-compound table unless check_names/comp.inf harmonization to PubChem ID has been performed first
- Supplementary data content of the source article was not extractable, so column-level specifications beyond those in the README/workflow may be incomplete

## Evidence

- [readme] Compound vote-counting votes definition: [evidence span withheld — anchor status: external_doc]
- [readme] Selecting comp.inf option retrieves descriptors for the result table: [evidence span withheld — anchor status: external_doc]
