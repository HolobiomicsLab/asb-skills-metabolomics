---
name: statistical-significance-threshold-filtering
description: Use when you have per-study P-values and fold-changes combined into global
  statistics via amanida and need to select metabolites that pass a significance cutoff
  (combined P < 0.05) and/or an effect-size cutoff (e.g. fold-change < 1.5–2) for
  reporting.
when_to_use_negative:
- Input is raw per-sample concentration data with disclosed standard deviations —
  then standard meta-analysis methods that use variance are appropriate instead of
  amanida's non-integral approach
- Only trend labels (up/down-regulation) are available with no P-values or fold-changes
  — use amanida's qualitative vote-counting (`amanida_vote`) instead of threshold
  filtering on combined statistics
- Metabolite identifiers are not curated/harmonized first — duplicate or non-standard
  IDs will distort the per-metabolite combined statistics before any cutoff is applied
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: amanida
  role: R package providing compute_amanida (combined statistics in the @stat slot),
    amanida_report with pvalue_cutoff and fc_cutoff parameters, and volcano_plot with
    a cutoff argument for threshold-based filtering and visualization
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package is installed and the filtered
    meta-analysis results are written to CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:300a70d907014f924d6722d26e85372dd4fe63c78d94b113ffbea401b22befa9
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# statistical-significance-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters weighted meta-analysis results (as produced by the amanida R package for non-integral metabolomics data) by combined significance and effect-size cutoffs, e.g. combined P < 0.05 and fold-change below a biologically meaningful threshold (2, or a stricter 1.5), so that only statistically and biologically notable metabolites are reported.

## When to use

Use when you have per-study P-values and fold-changes combined into global statistics via amanida and need to select metabolites that pass a significance cutoff (combined P < 0.05) and/or an effect-size cutoff (e.g. fold-change < 1.5–2) for reporting.

## When NOT to use

- Input is raw per-sample concentration data with disclosed standard deviations — then standard meta-analysis methods that use variance are appropriate instead of amanida's non-integral approach
- Only trend labels (up/down-regulation) are available with no P-values or fold-changes — use amanida's qualitative vote-counting (`amanida_vote`) instead of threshold filtering on combined statistics
- Metabolite identifiers are not curated/harmonized first — duplicate or non-standard IDs will distort the per-metabolite combined statistics before any cutoff is applied

## Inputs

- amanida S4 result object from compute_amanida (access @stat for combined statistics, @vote for vote-counting)
- Per-metabolite combined P-values and combined fold-changes (weighted Fisher's method and weighted averaging by study size n)
- Numeric cutoff parameters: pvalue_cutoff (e.g. 0.05) and fc_cutoff (e.g. 2 or 1.5)

## Outputs

- Filtered table of metabolites with combined P < 0.05 (and, separately, fold-change below/above the fc cutoff)
- amanida_results.csv of per-metabolite combined statistics
- Volcano plot labeling only compounds over the selected cutoff

## How to apply

After calling `compute_amanida` on curated data (columns: compound Id, P-value, fold-change, N, Reference), access the `@stat` slot of the returned S4 object to obtain the combined P-value and combined fold-change per metabolite. Apply the significance cutoff (default 0.05, set via `pvalue_cutoff`) and the fold-change cutoff (usually 2, set via `fc_cutoff`; the Lee et al. amino acid case used a stricter 1.5 to flag sub-threshold changes) to partition metabolites into significant vs non-significant and biologically meaningful vs sub-threshold. Report both criteria separately, since a metabolite can be statistically significant while its combined fold-change remains below the biological threshold — in the Lee et al. amino acid reanalysis, 20 of 21 amino acids had combined P < 0.05 yet all 21 combined fold-changes stayed below 1.5. Optionally visualize the partition with `volcano_plot`, which labels only compounds above the selected cutoffs (e.g. `cutoff = c(0.05, 4)`), and write the filtered per-metabolite results to a CSV such as amanida_results.csv.

## Related tools

- **amanida** (R package providing compute_amanida (combined statistics in the @stat slot), amanida_report with pvalue_cutoff and fc_cutoff parameters, and volcano_plot with a cutoff argument for threshold-based filtering and visualization) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package is installed and the filtered meta-analysis results are written to CSV)

## Examples

```
volcano_plot(amanida_result, cutoff = c(0.05,4))
```

## Evaluation signals

- Exactly 20 of the 21 amino acids from Lee et al. Supplementary Table S2 have combined P < 0.05
- All 21 combined fold-changes fall below 1.5, i.e. below the threshold of 2 for a biologically meaningful change
- Combined P-values are consistent with weighted Fisher's method and fold-changes with n-weighted averaging, with study size as the weight
- Volcano plot labels only compounds over the selected cutoff (e.g. cutoff = c(0.05, 4))
- Output CSV contains one row per curated metabolite with combined P-value and fold-change columns

## Limitations

- Metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis; the method relies only on P-value and fold-change without standard deviation
- Statistical significance (combined P < 0.05) does not imply biological relevance — all combined fold-changes can remain below the threshold of 2 even when 20 of 21 metabolites are significant
- The fold-change cutoff is user-set (usually 2 in the README, but 4 in the volcano_plot and amanida_report examples), so threshold choice must be stated explicitly

## Evidence

- [readme] pvalue/fc cutoff parameters: [evidence span withheld — anchor status: external_doc]
- [readme] fc_cutoff definition: [evidence span withheld — anchor status: external_doc]
- [readme] volcano plot cutoff behavior: [evidence span withheld — anchor status: external_doc]
