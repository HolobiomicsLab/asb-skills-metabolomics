---
name: fold-change-weighted-averaging
description: Use when individual metabolomics studies report only fold-change (no
  raw data, error, or variance) for the same compounds and you need a case-vs-control
  combined effect size, e.g. input tables with Id, P-value, Fold-change, N, Reference
  columns.
when_to_use_negative:
- When raw data or standard deviations are available for each study — then conventional
  meta-analysis methods that use variance are appropriate instead
- When studies report only qualitative trend labels (up/down-regulated) with no fold-change
  values — use amanida's vote-counting (amanida_vote) instead
- When the comparison orientation is unknown, since -1/value reversal of control/case
  entries cannot be assigned correctly
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package whose quantitative meta-analysis performs the log2 transformation,
    orientation reversal and study-size-weighted averaging of fold-changes
  repo: https://github.com/mariallr/amanida
- name: webchem
  role: Used by check_names to harmonize compound identifiers to PubChem IDs before
    combining, enabling duplicate checking
- name: R
  role: Runtime environment in which the amanida functions (amanida_read, compute_amanida)
    are called
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:8cb33d69f5ec5ee1e31eabd316e8dd68e84ec8340c94012810aff0144403b583
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fold-change-weighted-averaging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study fold-change values reported for metabolomics compounds into a single per-compound estimate by log2-transforming the fold-changes and averaging them with weights proportional to study size (n), as implemented in the amanida R package for meta-analysis without standard deviations.

## When to use

Use when individual metabolomics studies report only fold-change (no raw data, error, or variance) for the same compounds and you need a case-vs-control combined effect size, e.g. input tables with Id, P-value, Fold-change, N, Reference columns.

## When NOT to use

- When raw data or standard deviations are available for each study — then conventional meta-analysis methods that use variance are appropriate instead
- When studies report only qualitative trend labels (up/down-regulated) with no fold-change values — use amanida's vote-counting (amanida_vote) instead
- When the comparison orientation is unknown, since -1/value reversal of control/case entries cannot be assigned correctly

## Inputs

- csv/xls/xlsx/txt table with columns: compound Id, P-value, Fold-change, N (study size), Reference (e.g. Supplementary Table S2 amino acid concentrations from six Lee et al. studies)
- Optional trend/reference column identifiers specified via the coln vector, in the order Id, P-value, Fold-change, N, References

## Outputs

- Per-compound table of combined (weighted-average) fold-changes, e.g. written to combined_fold_changes.csv
- S4 amanida result object; the combined statistics are accessible via the @stat slot

## How to apply

Load the input table (csv/xls/xlsx/txt) via amanida_read in mode = "quan" so each row carries the compound Id, P-value, Fold-change, study size (N) and Reference. Log2-transform each study's reported fold-change to reduce skewness; reverse inverted (control/case) comparisons by applying -1/value so all studies share the case-vs-control orientation. Treat missing fold-change entries as NA to be skipped rather than as zero. Average the oriented log2 fold-changes per compound with weights proportional to the number of participants in each study, as in amanida's equation (2); optionally run check_names first so IDs are harmonized to PubChem IDs and duplicates merged. Interpret results against a fold-change cut-off (amanida's report default for biological meaningfulness is around 2; in the Lee et al. amino-acid reanalysis all 21 combined fold-changes fell below 1.5 and were therefore judged not biologically meaningful even where combined P < 0.05).

## Related tools

- **amanida** (R package whose quantitative meta-analysis performs the log2 transformation, orientation reversal and study-size-weighted averaging of fold-changes) — https://github.com/mariallr/amanida
- **webchem** (Used by check_names to harmonize compound identifiers to PubChem IDs before combining, enabling duplicate checking)
- **R** (Runtime environment in which the amanida functions (amanida_read, compute_amanida) are called)

## Examples

```
datafile <- amanida_read(input_file, mode = "quan", coln, separator=";"); amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Every study's fold-change has been log2-transformed and, where the comparison was control/case, replaced by -1/value so orientations agree
- Missing fold-change entries are skipped (NA) and never contribute as zeros to the weighted average
- Each compound contributes exactly one combined estimate (duplicates removed via identifier harmonization to PubChem IDs)
- Weights on the per-study log2 fold-changes are proportional to study size N, matching equation (2)
- For the Lee et al. amino-acid data, all 21 combined fold-changes must remain below 1.5 (and below the cut-off of 2 for biological meaningfulness)

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for a meta-analysis; the method only needs P-value and fold-change but cannot incorporate standard deviations
- Requires study size (n) to be reported, since it is the weight; rows missing N or fold-change are effectively ignored
- Combined fold-changes below the biological meaningfulness cut-off (≈1.5–2) mean statistical significance does not imply a meaningful relative change

## Evidence

- [abstract] P-values are combined via Fisher's method and fold-changes by averaging, both weighted by study size (n): "P-values are combined via Fisher’s method and fold-changes by averaging, both weighted by the study size (n)."
- [readme] Fold-change combination with log transform and n weighting: [evidence span withheld — anchor status: external_doc]
- [abstract] Rationale: only p-value and fold-change are needed: "Here, we present a meta-analysis approach using only the most reported statistical parameters in this field: P-value and fold-change."
- [results] Combined P < 0.05 does not guarantee a biologically meaningful change: [evidence span withheld — anchor status: not_found]
- [readme] compute_amanida returns an S4 object with stat slot: [evidence span withheld — anchor status: external_doc]
