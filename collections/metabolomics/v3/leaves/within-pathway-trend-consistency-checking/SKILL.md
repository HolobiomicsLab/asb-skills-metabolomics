---
name: within-pathway-trend-consistency-checking
description: Use when comparing directed vs mixed reporter_score modes on a KO abundance
  table and you must confirm that directed-mode enriched pathways show consistent
  within-pathway KO directions while mixed-mode sign cancellation precludes trend
  interpretation.
when_to_use_negative:
- The KO abundance table has been prefiltered before enrichment — GRSA is threshold-free
  and expects the full background
- You need quantitative pathway trend direction from mixed-mode reporter score signs
  — those signs are uninterpretable due to cancellation of opposing-trend KOs
- The task is differential KO testing only, with no pathway-level aggregation or mode
  comparison required
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: ReporterScore
  role: R package implementing GRSA; provides ko.test, reporter_score (mode=directed/mixed),
    get_reporter_score, plot_report, plot_enrich_res and the packaged KO_abundance/metadata
    data used for the mode comparison
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: companion dependency installed from GitHub alongside the development version
    of ReporterScore
  repo: https://github.com/Asa12138/pcutils
- name: R
  role: runtime environment for running the directed vs mixed mode comparison and
    writing the CSV comparison table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:434d163f6f0c970420287c247a7470d105a2994d7e26d5a04187d73344d4d82a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - pcutils
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# within-pathway-trend-consistency-checking

## Summary

Verifies that pathways enriched in directed-mode GRSA (ReporterScore) carry concordant within-pathway KO trend directions, whereas mixed-mode reporter score signs are uninterpretable because opposing-trend KOs cancel. Grounded in the directed/mixed mode distinction of Peng et al. 2024, Briefings in Bioinformatics (doi:10.1093/bib/bbae116).

## When to use

Use when comparing directed vs mixed reporter_score modes on a KO abundance table and you must confirm that directed-mode enriched pathways show consistent within-pathway KO directions while mixed-mode sign cancellation precludes trend interpretation.

## When NOT to use

- The KO abundance table has been prefiltered before enrichment — GRSA is threshold-free and expects the full background
- You need quantitative pathway trend direction from mixed-mode reporter score signs — those signs are uninterpretable due to cancellation of opposing-trend KOs
- The task is differential KO testing only, with no pathway-level aggregation or mode comparison required

## Inputs

- KO abundance table (rownames = K numbers like K00001, colnames = samples; unfiltered, read counts or normalized values)
- metadata table (rownames = samples matching abundance colnames, grouping column such as Group/Group2)
- built-in KEGG pathway-KO / module-KO modulelist (2023-08 version, loadable via load_KOlist())

## Outputs

- per-KO statistics from ko.test (ko_stat)
- directed-mode pathway-level reporter score table with p-value2zs signs preserving KO direction
- mixed-mode pathway-level reporter score table
- side-by-side directed vs mixed enrichment comparison (e.g. grsa_mode_comparison.csv)
- within-pathway KO trend plots (plot_report / plot_enrich_res)

## How to apply

Load the packaged KO_abundance table and metadata from the ReporterScore R package and check that rownames(metadata) match colnames(KO_abundance). Run ko.test on the KO abundance data against the metadata grouping (e.g. the WT/Group variable) to get per-KO statistics. Call reporter_score with mode set to the directed option so pvalue2zs converts KO p-values to signed statistics preserving within-pathway direction, then get_reporter_score to aggregate pathway-level reporter scores; rerun with mode=mixed for comparison. For each enriched pathway in each mode, inspect within-pathway KO trends (e.g. via plot_report or plot_enrich_res) and confirm the qualitative criterion: directed-mode enriched pathways have concordant KO directions, while mixed-mode signs cancel when KOs trend oppositely. Export the directed- and mixed-mode enrichment tables side by side (e.g. to grsa_mode_comparison.csv) using export_report_table-style output.

## Related tools

- **ReporterScore** (R package implementing GRSA; provides ko.test, reporter_score (mode=directed/mixed), get_reporter_score, plot_report, plot_enrich_res and the packaged KO_abundance/metadata data used for the mode comparison) — https://github.com/Asa12138/ReporterScore
- **pcutils** (companion dependency installed from GitHub alongside the development version of ReporterScore) — https://github.com/Asa12138/pcutils
- **R** (runtime environment for running the directed vs mixed mode comparison and writing the CSV comparison table)

## Examples

```
ko_stat <- ko.test(KO_abundance, "Group", metadata); res_dir <- reporter_score(ko_stat, "Group", metadata, mode="directed"); res_mix <- reporter_score(ko_stat, "Group", metadata, mode="mixed")
```

## Evaluation signals

- The comparison table contains enriched pathways for BOTH directed and mixed modes
- Directed-mode enriched pathways exhibit consistent within-pathway KO trend directions when inspected via plot_report / plot_enrich_res
- Mixed-mode reporter score signs cannot be mapped to a pathway-level increasing/decreasing trend because signed Z-scores of opposing-trend KOs cancel
- rownames(metadata) match colnames(KO_abundance) (all(rownames(metadata) %in% colnames(KO_abundance)) is TRUE) before ko.test
- Output tables follow the export_report_table-style schema with pathway-level reporter scores and adjusted p-values

## Limitations

- In mixed mode, the increasing or decreasing trend of an enriched pathway cannot be determined from the reporter score signs because opposing-trend KOs cancel each other's signed Z-scores
- GRSA exhibits slightly lower specificity, so enrichment results require additional experimental validation
- The pathway database version matters — ReporterScore ships a 2023-08 KEGG pathway-KO/module-KO database and update_KO_file() should be used to fetch the latest via the KEGG API

## Evidence

- [intro] directed vs mixed mode sign interpretability: "However, RSA is often misused due to a lack of specific tools and a systematic understanding of the algorithm"
- [methods] reporter_score signature with mode argument: "- `reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)`"
- [methods] directed mode preserves within-pathway direction via pvalue2zs: [evidence span withheld — anchor status: not_found]
- [readme] threshold-free, unfiltered input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] metadata/abundance sample matching check: "all(rownames(metadata) %in% colnames(KO_abundance))
## TRUE"
