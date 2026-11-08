---
name: cross-tool-enrichment-performance-comparison
description: Use when choosing among metabolomics pathway enrichment tools on runtime
  grounds, or when replicating the paper's runtime comparison of EnrichMet vs MetaboAnalystR
  vs FELLA on a fixed metabolite list.
when_to_use_negative:
- The goal is accuracy or biological agreement between tools rather than speed — runtime
  comparison alone does not validate enrichment result concordance
- The input is not a fixed metabolite identifier list shared across tools (e.g. raw
  metabolomics matrices or differential results only usable by one tool), which breaks
  the controlled runtime comparison
edam_operation: http://edamontology.org/operation_3955
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: EnrichMet
  role: Subject of the benchmark; performs Fisher's Exact Test pathway enrichment,
    MetSEA via fgsea, and network analysis in ~3 s on the 94-metabolite set
  repo: https://github.com/biodatalab/enrichmet.git
- name: MetaboAnalystR
  role: Baseline comparator; pathway analysis timed at approximately 28 seconds on
    the same metabolite list
- name: FELLA
  role: Network-based baseline comparator; pathway analysis timed at approximately
    14 minutes on the same metabolite list
- name: R
  role: Execution environment in which all three tools' runtimes are recorded
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:5f526bf70d8767cee65faa87f0b46db22f0c07e81258724a44338c73d972d371
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - FELLA
  - MetaboAnalystR
  - R
  edam_operation: http://edamontology.org/operation_3955
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cross-tool-enrichment-performance-comparison

## Summary

Benchmarks the wall-clock runtime of metabolite pathway enrichment tools (EnrichMet, MetaboAnalystR, FELLA) on an identical metabolite input set, quantifying EnrichMet's computational efficiency advantage (~3 s vs ~28 s vs ~14 min on 94 metabolites).

## When to use

Use when choosing among metabolomics pathway enrichment tools on runtime grounds, or when replicating the paper's runtime comparison of EnrichMet vs MetaboAnalystR vs FELLA on a fixed metabolite list.

## When NOT to use

- The goal is accuracy or biological agreement between tools rather than speed — runtime comparison alone does not validate enrichment result concordance
- The input is not a fixed metabolite identifier list shared across tools (e.g. raw metabolomics matrices or differential results only usable by one tool), which breaks the controlled runtime comparison

## Inputs

- Representative metabolite query list of 94 KEGG identifiers with summary statistics (met_id, pval/padj, log2fc) from the enrichmet package extdata
- KEGG pathway-to-metabolite map (e.g. fetch_kegg_pathway_metabolites(organism = "hsa"))
- Identical input set passed to MetaboAnalystR and FELLA for a controlled comparison

## Outputs

- Runtime comparison table with one row per tool (EnrichMet, MetaboAnalystR, FELLA) and per-tool wall-clock runtimes
- Verified ordering EnrichMet (~3 s) < MetaboAnalystR (~28 s) < FELLA (~14 min) on the 94-metabolite dataset

## How to apply

Load a single fixed metabolite identifier list — the paper's representative 94-metabolite KEGG identifier set with accompanying summary statistics from the enrichmet package extdata. Run each tool's pathway enrichment on exactly the same list, recording wall-clock runtime per tool: EnrichMet's single-call `enrichmet()` workflow (Fisher's Exact Test over-representation, fgsea-based MetSEA, STITCH/Reactome network analysis), MetaboAnalystR's pathway analysis, and FELLA's network-based pathway analysis. Assemble a runtime comparison table with one row per tool and verify the expected ordering EnrichMet < MetaboAnalystR < FELLA (approximately 3 s, 28 s, 14 min respectively). The rationale is that EnrichMet runs entirely locally with KEGG-derived annotations and no web-server round-trips, whereas FELLA's network propagation is the dominant cost.

## Related tools

- **EnrichMet** (Subject of the benchmark; performs Fisher's Exact Test pathway enrichment, MetSEA via fgsea, and network analysis in ~3 s on the 94-metabolite set) — https://github.com/biodatalab/enrichmet.git
- **MetaboAnalystR** (Baseline comparator; pathway analysis timed at approximately 28 seconds on the same metabolite list)
- **FELLA** (Network-based baseline comparator; pathway analysis timed at approximately 14 minutes on the same metabolite list)
- **R** (Execution environment in which all three tools' runtimes are recorded)

## Examples

```
library(enrichmet); PathwayVsMetabolites <- fetch_kegg_pathway_metabolites(organism="hsa"); system.time(results <- enrichmet(inputMetabolites=met_list, PathwayVsMetabolites=PathwayVsMetabolites, input_type="kegg"))
```

## Evaluation signals

- Per-tool runtime table contains exactly three rows: EnrichMet, MetaboAnalystR, FELLA, each with a recorded wall-clock time
- Runtime ordering satisfies EnrichMet < MetaboAnalystR < FELLA on the same 94-metabolite input
- EnrichMet runtime is on the order of seconds (reported benchmark ~3 s), MetaboAnalystR tens of seconds (~28 s), FELLA minutes (~14 min)
- All three tools consumed the identical metabolite identifier list, so runtime differences are attributable to the tools rather than input size

## Limitations

- Reported runtimes (~3 s vs ~28 s vs ~14 min) come from the paper's representative 94-metabolite dataset; absolute times vary with hardware and network latency, and EnrichMet fetches KEGG/Reactome annotations at runtime via APIs cached with BiocFileCache
- The comparison measures computational efficiency only; the paper does not use it to claim result-concordance differences between tools

## Evidence

- [intro] Runtimes on a fixed 94-metabolite benchmark set: "EnrichMet completed the pathway analysis for representative dataset of 94 metabolites in approximately 3 seconds, compared with approximately 28 seconds for MetaboAnalystR"
- [intro] FELLA baseline runtime: "approximately 28 seconds for MetaboAnalystR and approximately 14 minutes for FELLA"
- [abstract] EnrichMet benchmark headline claim: "Benchmark analyses show that enrichment on sample data completes in approximately 3 seconds"
- [intro] Single-call local execution rationale: [evidence span withheld — anchor status: not_found]
- [readme] Runtime annotation fetching affects timing: [evidence span withheld — anchor status: external_doc]
