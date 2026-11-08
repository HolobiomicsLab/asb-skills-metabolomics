---
name: missing-data-tolerant-meta-analysis-computation
description: Use when combining published fold-changes and P-values across metabolomics
  studies that report only overall statistics (no raw data, error, or variance), with
  incomplete/missing values per compound.
when_to_use_negative:
- Raw data or per-study standard deviations/variances are available — standard meta-analysis
  methods using effect sizes with error estimates are preferable, since amanida targets
  overall results without standard deviation
- Studies report only trend labels (up-/down-regulation) without fold-changes or P-values
  — use amanida's qualitative vote-counting mode instead
- Identifiers are uncurated (mixed chemical names, InChI, InChIKey, SMILES) and harmonization
  to PubChem is not possible — identifier curation is required before reliable per-compound
  combination
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: 'R package performing the missing-data-tolerant weighted meta-analysis: log2
    fold-change averaging weighted by study size and Fisher P-value combination'
  repo: https://github.com/mariallr/amanida
- name: R
  role: runtime environment in which amanida's compute_amanida, amanida_read and visualization
    functions are executed
- name: webchem
  role: used by check_names to convert compound identifiers (names, InChI, InChIKey,
    SMILES) to a common PubChem nomenclature before combination
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:38f1f9b892e71ced3734dc7520864e1d064c3ad88aba5c81e1a9c5d3f643867a
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# missing-data-tolerant-meta-analysis-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study fold-changes (and P-values) from metabolomics systematic reviews into per-compound estimates using amanida's weighted, non-integral meta-analysis, which works without standard deviations and ignores missing entries.

## When to use

Use when combining published fold-changes and P-values across metabolomics studies that report only overall statistics (no raw data, error, or variance), with incomplete/missing values per compound.

## When NOT to use

- Raw data or per-study standard deviations/variances are available — standard meta-analysis methods using effect sizes with error estimates are preferable, since amanida targets overall results without standard deviation
- Studies report only trend labels (up-/down-regulation) without fold-changes or P-values — use amanida's qualitative vote-counting mode instead
- Identifiers are uncurated (mixed chemical names, InChI, InChIKey, SMILES) and harmonization to PubChem is not possible — identifier curation is required before reliable per-compound combination

## Inputs

- CSV/XLS/TXT table of overall study results with columns Id (compound name), P-value, Fold-change, N (number of individuals), Reference — e.g. amanida's sample_data from the colorectal cancer urine volatilome/metabolome systematic review (Mallafré et al., Cancers 2021)
- study size (n) per report used as the weight

## Outputs

- S4 amanida result object with per-compound combined statistics (amanida_result@stat) including weighted fold-change and weighted Fisher P-value
- optional compound annotation table (PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank) when comp.inf = TRUE
- per-compound table of combined fold-changes (e.g. combined_fold_changes.csv)

## How to apply

Load the study-level table (compound identifier, P-value, fold-change, N, reference) with amanida_read in 'quan' mode, then optionally harmonize identifiers to PubChem via check_names before running compute_amanida. Log2-transform each reported fold-change to reduce skewness, reverse inverted (control/case) comparisons by -1/value so all studies share a case-vs-control orientation, and treat missing fold-changes as ignorable (NA-skipping) rather than zero. Average the oriented log2 fold-changes per compound with weights proportional to study size (n) — the weighted average of equation (2) — while P-values are combined via Fisher's method weighted by the number of participants. Access per-compound combined statistics from the S4 result object (amanida_result@stat) and optionally attach compound descriptors (PubChem ID, Molecular Formula, InChIKey, HMDB, KEGG, etc.) by setting comp.inf = TRUE, then visualize discrepancies with volcano_plot.

## Related tools

- **amanida** (R package performing the missing-data-tolerant weighted meta-analysis: log2 fold-change averaging weighted by study size and Fisher P-value combination) — https://github.com/mariallr/amanida
- **R** (runtime environment in which amanida's compute_amanida, amanida_read and visualization functions are executed)
- **webchem** (used by check_names to convert compound identifiers (names, InChI, InChIKey, SMILES) to a common PubChem nomenclature before combination)

## Examples

```
library(amanida); data("sample_data"); amanida_result <- compute_amanida(sample_data, comp.inf = F); volcano_plot(amanida_result, cutoff = c(0.05,4))
```

## Evaluation signals

- Each compound's combined fold-change equals the study-size-weighted average of oriented log2 fold-changes, with NA entries skipped (never counted as zero or 1)
- Inverted (control/case) comparisons contribute the reciprocal negative fold-change (-1/value), so all estimates share case-vs-control orientation
- The result object exposes both quantitative statistics (@stat) and vote-counting (@vote) tables with one row per harmonized compound
- Combined P-values match weighted Fisher's method with weights proportional to the number of participants per study
- Volcano plot with cutoff = c(0.05, 2) flags only compounds exceeding both P-value and fold-change thresholds, and discrepancies between studies are detectable at a glance

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for a meta-analysis, so coverage per compound is partial and missing entries reduce the evidence base
- The method requires only P-value and fold-change; it cannot exploit within-study variance or raw data, so between-study heterogeneity is not modeled
- Identifier harmonization depends on PubChem lookup via webchem; unresolvable or duplicated IDs must be curated manually before computation

## Evidence

- [readme] readme fold-change combination: [evidence span withheld — anchor status: external_doc]
- [readme] readme compute step: [evidence span withheld — anchor status: external_doc]
- [readme] readme sample_data provenance: [evidence span withheld — anchor status: external_doc]
