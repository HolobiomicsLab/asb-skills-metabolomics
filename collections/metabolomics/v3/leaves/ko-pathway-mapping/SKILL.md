---
name: ko-pathway-mapping
description: Use when a KO (or gene/compound) abundance table must be linked to KEGG
  pathway or module gene sets before computing pathway-level enrichment statistics
  such as reporter scores.
when_to_use_negative:
- Input features are not mappable to KEGG identifiers (e.g. unannotated compounds
  lacking C numbers) — README requires converting compound IDs to C numbers first
- You need gene sets from a source other than KEGG/GO and cannot express them as a
  modulelist
- The abundance table has been prefiltered, destroying the background KO pool that
  GRSA's threshold-free mapping and null permutation require
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0622
tools:
- name: ReporterScore
  role: R package providing the built-in KEGG pathway/module-KO databases (load_KOlist,
    Module_htable) and the min_exist_KO/max_exist_KO coverage filtering used during
    KO-to-pathway mapping
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed alongside ReporterScore for the GitHub development install
  repo: https://github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:6e724508d63449fc6360f67cd1c755c1a78d5294f88583d99a8dcc45bb8b5fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - pcutils
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# KO-pathway-mapping

## Summary

Maps individual KEGG K-number features to KEGG pathway/module definitions using the ReporterScore package's built-in pathway databases (e.g. Module_htable), applying min_exist_KO and max_exist_KO coverage filters so only adequately covered pathways are scored in GRSA.

## When to use

Use when a KO (or gene/compound) abundance table must be linked to KEGG pathway or module gene sets before computing pathway-level enrichment statistics such as reporter scores.

## When NOT to use

- Input features are not mappable to KEGG identifiers (e.g. unannotated compounds lacking C numbers) — README requires converting compound IDs to C numbers first
- You need gene sets from a source other than KEGG/GO and cannot express them as a modulelist
- The abundance table has been prefiltered, destroying the background KO pool that GRSA's threshold-free mapping and null permutation require

## Inputs

- KO abundance table (rownames are K numbers such as 'K03169', colnames are samples)
- KO-level differential statistics (P-values and log fold changes)
- Built-in KEGG pathway-KO / module-KO database (KOlist / Module_htable, 2023-08 version)
- Optional custom modulelist from custom_modulelist() or organism-specific database from custom_modulelist_from_org()

## Outputs

- Pathway-to-observed-KO mappings for each retained pathway
- Coverage-filtered pathway set (respecting min_exist_KO and max_exist_KO) passed to reporter-score computation

## How to apply

Load the built-in KEGG pathway-KO or module-KO database shipped with ReporterScore (KOlist from load_KOlist(), or the Module_htable pathway database packaged with KO_abundance_test), optionally refreshing it with update_KO_file() via the KEGG API. Each pathway entry lists its member KOs; map the rownames of the KO abundance/statistics table (e.g. 'K03169', 'K00001') onto these member lists. Filter pathways by coverage: retain a pathway only if the number of observed member KOs falls between min_exist_KO and max_exist_KO, which removes sparsely observed or excessively large gene sets. In reporter_score(), these mappings are consumed alongside the feature type ('ko', 'gene', or 'compound') and type ('pathway' or 'module') arguments; a custom gene set can be supplied via custom_modulelist(). The mapping output feeds directly into aggregation of member-feature Z-scores into pathway-level reporter scores in get_reporter_score().

## Related tools

- **ReporterScore** (R package providing the built-in KEGG pathway/module-KO databases (load_KOlist, Module_htable) and the min_exist_KO/max_exist_KO coverage filtering used during KO-to-pathway mapping) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed alongside ReporterScore for the GitHub development install) — https://github.com/Asa12138/pcutils

## Examples

```
KOlist <- load_KOlist(); res <- reporter_score(KO_abundance, group="Group", metadata=metadata, feature="ko", type="pathway", modulelist=KOlist, min_exist_KO=3, max_exist_KO=500)
```

## Evaluation signals

- Every pathway retained in the mapping has a member-KO count within [min_exist_KO, max_exist_KO]
- All mapped feature IDs resolve to entries in the pathway database (e.g. K numbers present in KOlist)
- Each pathway's mapping is consistent with the built-in 2023-08 KEGG version, or has been refreshed with update_KO_file()
- Metadata rownames match abundance-table colnames (all(rownames(metadata) %in% colnames(KO_abundance))) so mapped samples align
- Downstream reporter_scores.csv contains a row for every pathway that passed the coverage filter

## Limitations

- Built-in databases are the 2023-08 KEGG version; using outdated pathway definitions can change mappings, so update_KO_file()/update_org_pathway() should be run for current analyses
- Databases are incomplete, which complicates defining true negatives and can affect which pathways pass coverage filters
- No version number is stated for the ReporterScore package, limiting reproducibility of the exact database mapping

## Evidence

- [readme] Built-in KEGG databases are versioned and updatable: [evidence span withheld — anchor status: external_doc]
- [methods] Coverage thresholds are function arguments of reporter_score: "`reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)`"
- [readme] Rownames of abundance table must be feature ids such as K numbers: [evidence span withheld — anchor status: external_doc]
- [discussion] GRSA considers all KOs in a pathway, motivating complete mapping: "GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list"
- [readme] Input tables should not be prefiltered to preserve background: [evidence span withheld — anchor status: external_doc]
