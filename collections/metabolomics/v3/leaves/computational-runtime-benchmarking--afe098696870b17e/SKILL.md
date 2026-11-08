---
name: computational-runtime-benchmarking
description: Use when comparing wall-clock runtime of pathway enrichment tools (e.g.,
  EnrichMet vs MetaboAnalystR vs FELLA) on the same metabolite list to quantify computational
  efficiency claims.
when_to_use_negative:
- The goal is pathway enrichment results or biological interpretation rather than
  quantifying tool runtime — use the individual enrichment skill instead.
- Inputs are not an identical metabolite list shared across all tools — runtime differences
  would confound input size rather than tool efficiency.
- Benchmarking tools outside metabolomics pathway enrichment (e.g., genomics, assembly)
  — the reported reference timings are specific to EnrichMet/MetaboAnalystR/FELLA.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: EnrichMet
  role: Primary benchmarked tool; pathway enrichment executed via a single R function
    call and timed (reported approx. 3 s on 94 metabolites)
  repo: https://github.com/biodatalab/enrichmet.git
- name: MetaboAnalystR
  role: Comparison tool; pathway analysis on the same 94-metabolite list timed (reported
    approx. 28 s)
- name: FELLA
  role: Comparison tool; network-based pathway analysis on the same 94-metabolite
    list timed (reported approx. 14 min)
- name: R
  role: Execution environment and timing harness (e.g., system.time) for recording
    per-tool wall-clock runtimes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:e719c48fd804623b4dcc3aae0dc07d73e208af9af59811adc99d5744da53083f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - FELLA
  - MetaboAnalystR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# computational-runtime-benchmarking

## Summary

Benchmark wall-clock runtime of competing pathway enrichment tools (EnrichMet, MetaboAnalystR, FELLA) on an identical metabolite input set to quantify computational efficiency differences. This skill reproduces the paper's head-to-head timing comparison on the representative 94-metabolite dataset.

## When to use

Use when comparing wall-clock runtime of pathway enrichment tools (e.g., EnrichMet vs MetaboAnalystR vs FELLA) on the same metabolite list to quantify computational efficiency claims.

## When NOT to use

- The goal is pathway enrichment results or biological interpretation rather than quantifying tool runtime — use the individual enrichment skill instead.
- Inputs are not an identical metabolite list shared across all tools — runtime differences would confound input size rather than tool efficiency.
- Benchmarking tools outside metabolomics pathway enrichment (e.g., genomics, assembly) — the reported reference timings are specific to EnrichMet/MetaboAnalystR/FELLA.

## Inputs

- character vector of KEGG compound IDs (representative 94-metabolite benchmark list from enrichmet extdata)
- accompanying summary statistics (met_id, pval/padj, log2fc) for tools requiring ranked or background input
- KEGG pathway-to-metabolite map (PathwayVsMetabolites, organism code "hsa")
- same input list passed identically to EnrichMet, MetaboAnalystR, and FELLA

## Outputs

- wall-clock runtime measurement per tool (seconds/minutes)
- runtime comparison table with one row per tool (EnrichMet, MetaboAnalystR, FELLA)
- verified ordering assertion: EnrichMet < MetaboAnalystR < FELLA (approx. 3 s vs 28 s vs 14 min)

## How to apply

Load a representative KEGG identifier list (the paper's benchmark uses 94 metabolites with accompanying summary statistics from the enrichmet package extdata directory). Execute each tool's pathway analysis on the exact same metabolite list and record wall-clock runtime for each (e.g., with R's system.time), keeping input, machine, and annotation state constant. The reported benchmark expects approximately 3 seconds for EnrichMet, approximately 28 seconds for MetaboAnalystR, and approximately 14 minutes for FELLA. Assemble a runtime comparison table with one row per tool and verify the ordering EnrichMet < MetaboAnalystR < FELLA. Note that enrichmet fetches KEGG/Reactome annotations at runtime and caches them with BiocFileCache, so time after a warm cache or report cache state separately.

## Related tools

- **EnrichMet** (Primary benchmarked tool; pathway enrichment executed via a single R function call and timed (reported approx. 3 s on 94 metabolites)) — https://github.com/biodatalab/enrichmet.git
- **MetaboAnalystR** (Comparison tool; pathway analysis on the same 94-metabolite list timed (reported approx. 28 s))
- **FELLA** (Comparison tool; network-based pathway analysis on the same 94-metabolite list timed (reported approx. 14 min))
- **R** (Execution environment and timing harness (e.g., system.time) for recording per-tool wall-clock runtimes)

## Examples

```
system.time(results <- enrichmet(inputMetabolites = input_metabolites_94, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df))
```

## Evaluation signals

- Per-tool runtimes reproduce the ordering EnrichMet < MetaboAnalystR < FELLA on the same input list.
- EnrichMet pathway analysis completes in seconds (approximately 3 s on the 94-metabolite reference dataset), not minutes.
- All three tools receive the identical 94-metabolite KEGG identifier list — verify input equivalence before comparing timings.
- Comparison table contains exactly one row per tool (EnrichMet, MetaboAnalystR, FELLA) with a recorded wall-clock value for each.
- Timing runs are repeatable within the same order of magnitude as the reported benchmarks (3 s / 28 s / 14 min).

## Limitations

- Runtimes are machine- and network-dependent: enrichmet fetches KEGG REST and Reactome annotations at runtime (cached via BiocFileCache), so cold-cache timing includes download latency; report cache state.
- The reference benchmark covers a single 94-metabolite dataset; scaling to other input sizes or species is not established by these numbers.
- No changelog is documented for the package, so version-specific runtime behavior is hard to track over time.

## Evidence

- [intro] Reported benchmark runtimes for the 94-metabolite dataset: "EnrichMet completed the pathway analysis for representative dataset of 94 metabolites in approximately 3 seconds, compared with approximately 28 seconds for MetaboAnalystR and approximately 14"
- [abstract] Benchmark completion time claim: "Benchmark analyses show that enrichment on sample data completes in approximately 3 seconds"
- [readme] Runtime annotation fetching affects timing: [evidence span withheld — anchor status: external_doc]
