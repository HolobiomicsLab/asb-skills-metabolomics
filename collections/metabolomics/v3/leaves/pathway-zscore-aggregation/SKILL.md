---
name: pathway-zscore-aggregation
description: Use when you have a full (unprefiltered) KO, gene, or compound abundance
  table plus sample metadata and need threshold-free pathway enrichment with permutation-calibrated
  reporter scores.
when_to_use_negative:
- Input has been prefiltered to a list of 'significant' features, discarding background
  information — GRSA is threshold-free and requires the full background table
- Features cannot be mapped to any pathway database and no custom_modulelist can be
  constructed (e.g. unmapped compound IDs not standardized to KEGG C numbers)
- The goal is to enumerate only a small pre-defined significant list via hypergeometric
  overlap rather than aggregate Z-scores over all pathway members
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: ReporterScore (R package)
  role: 'Implements the GRSA reporter-score computation: pvalue2zs Z-score conversion,
    KO-to-pathway mapping via Module_htable, permutation-based null standardization,
    and FDR correction'
  repo: https://github.com/Asa12138/ReporterScore
- name: R
  role: Runtime environment; ReporterScore is installed from CRAN or via devtools::install_github
- name: devtools
  role: Used to install the development version of ReporterScore and its dependency
    pcutils from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:646c2a804c2c54fbc38cdce167ae97697cd29297d6e789660d8d87218d0bf6ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - ReporterScore (R package)
  - devtools
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# pathway-zscore-aggregation

## Summary

Computes pathway-level reporter scores by converting per-feature differential P-values into (signed or unsigned) Z-scores, aggregating them over KEGG pathway/module members, and standardizing against a permutation-derived null distribution, as implemented in the GRSA method of the R package ReporterScore.

## When to use

Use when you have a full (unprefiltered) KO, gene, or compound abundance table plus sample metadata and need threshold-free pathway enrichment with permutation-calibrated reporter scores.

## When NOT to use

- Input has been prefiltered to a list of 'significant' features, discarding background information — GRSA is threshold-free and requires the full background table
- Features cannot be mapped to any pathway database and no custom_modulelist can be constructed (e.g. unmapped compound IDs not standardized to KEGG C numbers)
- The goal is to enumerate only a small pre-defined significant list via hypergeometric overlap rather than aggregate Z-scores over all pathway members

## Inputs

- KO/gene/compound feature abundance table (rownames = feature ids such as K00001, PEX11A, or C00024; colnames = samples; not prefiltered)
- Experimental metadata table (rownames = samples matching abundance table colnames; grouping variable may be categorical, multiple time points, or continuous)
- Pathway database (built-in KEGG pathway-KO / module-KO Module_htable, GO-gene list, organism-specific pathway-gene table, or a custom modulelist)

## Outputs

- reporter_scores.csv with a reporter-score column and a corrected-P (adjusted P-value) column for every pathway
- Permutation-calibrated pathway P-values standardized against the null pathway Z-score distribution

## How to apply

Load KO-level differential statistics (P-values and log fold changes), e.g. from the packaged KO_abundance_test dataset, and convert each feature's P-value to a signed (mode='mixed') or unsigned Z-score via pvalue2zs with P-value adjustment by p.adjust.method1. Map individual KOs to pathways using the Module_htable database (built-in KEGG pathway-KO and module-KO databases, updatable with update_KO_file()), filtering pathways by min_exist_KO and max_exist_KO coverage thresholds. Aggregate member-feature Z-scores into the pathway-level reporter score, then generate a null distribution of pathway Z-scores by label permutations (perm argument) and standardize each observed score against this null to get permutation P-values. Apply multiple-testing correction with p.adjust.method2 and write the reporter-score table (with adjusted P-values) to reporter_scores.csv. Do not prefilter the input abundance table, because GRSA is threshold-free and considers all KOs in a pathway rather than a pre-defined significant list.

## Related tools

- **ReporterScore (R package)** (Implements the GRSA reporter-score computation: pvalue2zs Z-score conversion, KO-to-pathway mapping via Module_htable, permutation-based null standardization, and FDR correction) — https://github.com/Asa12138/ReporterScore
- **R** (Runtime environment; ReporterScore is installed from CRAN or via devtools::install_github)
- **devtools** (Used to install the development version of ReporterScore and its dependency pcutils from GitHub)

## Examples

```
reporter_score(kodf=KO_abundance, group="Group", metadata=metadata, method="wilcox.test", mode="mixed", p.adjust.method1="BH", p.adjust.method2="BH")
```

## Evaluation signals

- reporter_scores.csv exists and contains a reporter-score column and a corrected-P column for every pathway
- All adjusted P-values in reporter_scores.csv fall within [0,1]
- Every feature P-value is transformed to a Z-score via pvalue2zs; with mode='mixed' Z-scores carry sign matching the direction of the log fold change
- Pathways failing the min_exist_KO / max_exist_KO coverage thresholds are excluded from the output
- Reported pathway P-values are permutation-based: they must be reproducible for a fixed seed and consistent with standardization against the null pathway Z-score distribution generated with the perm argument

## Limitations

- GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations
- Reporter P-value quality depends on the permutation budget (perm argument); the README notes using an up-to-date pathway database is very important (update_KO_file via KEGG API)
- Metadata rownames and abundance-table colnames must match (or partially match) or samples will not be paired correctly

## Evidence

- [abstract] We implemented GRSA in an R package, ReporterScore, integrating with a powerful visualization module and updatable pathway databases: "We implemented GRSA in an R package, ReporterScore, integrating with a powerful visualization module and updatable pathway databases"
- [methods] reporter_score signature: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [discussion] all KOs considered vs hypergeometric: "GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list"
- [readme] threshold-free input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] built-in pathway databases: [evidence span withheld — anchor status: not_found]
