---
name: fold-change-log2-transformation
description: Use when a metabolomics meta-analysis has only per-study fold-changes
  (no standard deviations or raw data) and per-compound combined effect sizes must
  be computed from heterogeneous study reports.
when_to_use_negative:
- Raw data or standard deviations are available for each study — an inverse-variance
  meta-analysis is then preferable to the weighted fold-change average
- Only qualitative trend labels (up/down-regulated) are reported — use amanida_vote
  vote-counting instead of fold-change combination
- Identifiers have not been curated/harmonized to PubChem IDs (via check_names) —
  duplicate or mismatched compound names will corrupt the per-compound combination
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: R package providing amanida_read (mode = "quan"), compute_amanida (log2 fold-change
    averaging weighted by n), and volcano_plot for the combined results
  repo: https://github.com/mariallr/amanida
- name: webchem
  role: Used by check_names to harmonize compound identifiers to PubChem IDs before
    combination, ensuring fold-changes from the same compound are merged
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:7ebef774f5085191e5e768133e62ad39e8cd098de279d7a459392350980dd6ef
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# fold-change-log2-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Log2-transform per-study fold-change values and combine them via a weighted average (weights proportional to study size n) for metabolomics meta-analysis with amanida, including orientation reversal of inverted comparisons and NA-skipping of missing entries.

## When to use

Use when a metabolomics meta-analysis has only per-study fold-changes (no standard deviations or raw data) and per-compound combined effect sizes must be computed from heterogeneous study reports.

## When NOT to use

- Raw data or standard deviations are available for each study — an inverse-variance meta-analysis is then preferable to the weighted fold-change average
- Only qualitative trend labels (up/down-regulated) are reported — use amanida_vote vote-counting instead of fold-change combination
- Identifiers have not been curated/harmonized to PubChem IDs (via check_names) — duplicate or mismatched compound names will corrupt the per-compound combination

## Inputs

- CSV/XLSX/TXT table of per-study overall results with columns Id (compound name), P-value, Fold-change, N (number of individuals), References
- amanida data object produced by amanida_read(mode = "quan", coln = c("Compound Name", "P-value", "Fold-change", "N total", "References"))

## Outputs

- Per-compound combined fold-change table (amanida_result@stat), written e.g. to combined_fold_changes.csv
- S4 amanida result object with @stat (adapted meta-analysis) and @vote (vote-counting) slots

## How to apply

Load the per-study report table (Id, P-value, Fold-change, N, References) with amanida_read in mode = "quan". Reverse any inverted (control/case) comparisons by applying -1/value to their fold-changes so all studies share the case-vs-control orientation, and treat missing fold-change entries as ignorable (NA-skipping) rather than zero. Log2-transform each reported fold-change to reduce skewness, then average the oriented log2 values per compound with weights proportional to study size (n), as in equation (2). In practice this is wrapped by compute_amanida(datafile), which returns an S4 object whose combined statistics live in the @stat slot. Rationale: metabolomics studies rarely disclose variance, so fold-changes are combined by averaging weighted by the number of participants instead of an inverse-variance method.

## Related tools

- **amanida** (R package providing amanida_read (mode = "quan"), compute_amanida (log2 fold-change averaging weighted by n), and volcano_plot for the combined results) — https://github.com/mariallr/amanida
- **webchem** (Used by check_names to harmonize compound identifiers to PubChem IDs before combination, ensuring fold-changes from the same compound are merged)

## Examples

```
library(amanida); coln <- c("Compound Name", "P-value", "Fold-change", "N total", "References"); datafile <- amanida_read(input_file, mode = "quan", coln, separator = ";"); amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Every input fold-change for inverted (control/case) comparisons appears in the output with reversed orientation (-1/value) before log2 transformation
- Missing fold-change entries are skipped (NA-skipping), not imputed as zero — zero-filling would bias the weighted average toward log2(0)
- The combined per-compound log2 fold-change equals the study-size-weighted average of oriented log2 fold-changes (weights proportional to n), per equation (2)
- Output combined effect sizes are symmetric around 1 in raw fold-change space (log2 values symmetric around 0), and compounds with discordant study trends yield combined values near 0 in log2 space
- The @stat slot of the returned S4 object contains one row per harmonized compound, with duplicate identifiers merged after check_names

## Limitations

- Requires each study to report a fold-change; metabolomics studies rarely disclose all statistical information needed, and entries lacking fold-changes are simply dropped
- The weighted average uses study size (n) as weight and cannot account for within-study variance since standard deviations are typically unavailable
- Identifier harmonization (PubChem conversion via check_names, based on the Metabolites Merging Strategy) must precede combination, or duplicates/aliases will be treated as distinct compounds

## Evidence

- [abstract] P-values are combined via Fisher's method and fold-changes by averaging, both weighted by study size (n): "The P-values are combined via Fisher’s method and fold-changes by averaging, both weighted by the study size (n)."
- [intro] statistical significance and relative change are combined using weighted methods which do no require standard deviation, weighted Fisher for p-values and weighted average for fold-change: "statistical significance and relative change are combined using weighted methods which do no require standard deviation, weighted Fisher for p-values and weighted average for fold-change, where the"
- [readme] Fold-change combination: logarithmic transformation for average with weighting by number of participants: [evidence span withheld — anchor status: external_doc]
- [readme] compute_amanida returns an S4 object with adapted meta-analysis in @stat: [evidence span withheld — anchor status: external_doc]
- [readme] sample dataset from the colorectal cancer urine systematic review: [evidence span withheld — anchor status: external_doc]
