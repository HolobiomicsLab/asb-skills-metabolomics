---
name: r-package-extdata-data-loading
description: Use when reproducing the EnrichMet runtime benchmark or any example pathway-enrichment
  run that requires the package's built-in 94-metabolite dataset and its summary-statistics
  table.
when_to_use_negative:
- The user has their own metabolomics matrix or metabolite list and does not need
  the package's bundled benchmark dataset.
- Working with a non-KEGG identifier system (e.g. lipid LION terms) where the extdata
  KEGG IDs are not directly interpretable.
- The task requires live annotation retrieval (fetch_kegg_pathway_metabolites, fetch_kegg_compound_lookup)
  rather than just loading the example dataset.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: EnrichMet
  role: R package whose extdata directory stores the representative 94-metabolite
    example data that this loading skill packages for benchmarking and example runs
  repo: https://github.com/biodatalab/enrichmet.git
- name: MetaboAnalystR
  role: downstream benchmark tool that consumes the same loaded metabolite list to
    compare runtime
- name: FELLA
  role: network-based benchmark tool that consumes the same loaded metabolite list
    to compare runtime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:7ae6b5e5d0c3b9c2838aa0b4d22f13767695d9e88ee8b96d3a060fa7c4100538
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - FELLA
  - MetaboAnalystR
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# r-package-extdata-data-loading

## Summary

Loads bundled example metabolomics data (e.g. the representative 94-metabolite KEGG identifier list and summary statistics) from the enrichmet R package's extdata directory so downstream pathway enrichment benchmarks can be run reproducibly.

## When to use

Use when reproducing the EnrichMet runtime benchmark or any example pathway-enrichment run that requires the package's built-in 94-metabolite dataset and its summary-statistics table.

## When NOT to use

- The user has their own metabolomics matrix or metabolite list and does not need the package's bundled benchmark dataset.
- Working with a non-KEGG identifier system (e.g. lipid LION terms) where the extdata KEGG IDs are not directly interpretable.
- The task requires live annotation retrieval (fetch_kegg_pathway_metabolites, fetch_kegg_compound_lookup) rather than just loading the example dataset.

## Inputs

- enrichmet package extdata directory contents (bundled 94-metabolite KEGG identifier list)
- accompanying summary-statistics table with columns met_id, pval or padj, and log2fc
- optional fallback: Zenodo record 17819145 files summary_stat.csv / example_data.csv

## Outputs

- character vector of KEGG compound IDs suitable for enrichmet()'s inputMetabolites parameter
- data.frame of summary statistics usable for MetSEA and background extraction
- wall-clock runtime measurements for EnrichMet, MetaboAnalystR, and FELLA on the same metabolite list

## How to apply

Locate the enrichmet package installation and point file-reading code at its extdata directory, which ships the representative 94-metabolite KEGG identifier list plus accompanying summary statistics (columns met_id, pval/padj, log2fc per the README's example_data description). Read the KEGG IDs with a simple vector or data-frame load (e.g. read.csv or a package helper) so the metabolites can be passed as inputMetabolites to enrichmet(). Keep the loaded data unmodified because the benchmark comparison (EnrichMet ~3 s vs MetaboAnalystR ~28 s vs FELLA ~14 min) depends on feeding the identical 94-metabolite list to each tool. If the extdata file is missing, fall back to the Zenodo-hosted summary_stat.csv / example_data.csv referenced in the README via get_cached_file().

## Related tools

- **EnrichMet** (R package whose extdata directory stores the representative 94-metabolite example data that this loading skill packages for benchmarking and example runs) — https://github.com/biodatalab/enrichmet.git
- **MetaboAnalystR** (downstream benchmark tool that consumes the same loaded metabolite list to compare runtime)
- **FELLA** (network-based benchmark tool that consumes the same loaded metabolite list to compare runtime)

## Examples

```
library(enrichmet); example_summary <- read.csv(get_cached_file("https://zenodo.org/api/records/17819145/files/summary_stat.csv/content"), stringsAsFactors = FALSE)
```

## Evaluation signals

- The loaded metabolite list contains exactly 94 KEGG identifiers, matching the benchmark dataset size.
- Summary-statistics table includes the required columns met_id, pval (or padj), and log2fc before being passed to enrichmet().
- Identical metabolite vector is supplied to EnrichMet, MetaboAnalystR, and FELLA so runtime ordering EnrichMet < MetaboAnalystR < FELLA holds.
- EnrichMet run on the loaded data completes in roughly 3 seconds as reported in the benchmark.

## Limitations

- Annotation data (KEGG pathways, compound lookups, Reactome mappings) are NOT bundled in extdata; they must be fetched at runtime from KEGG REST/Reactome APIs and cached with BiocFileCache.
- If the extdata example files are unavailable, the README directs users to Zenodo-hosted CSVs, adding a network dependency.
- The benchmark runtime numbers are approximate (approx. 3 s, 28 s, 14 min) and will vary with hardware.

## Evidence

- [intro] Representative dataset of 94 metabolites: "EnrichMet completed the pathway analysis for representative dataset of 94 metabolites in approximately 3 seconds"
- [intro] Benchmark runtime ordering: "compared with approximately 28 seconds for MetaboAnalystR and approximately 14 minutes for FELLA"
- [readme] Summary statistics columns required for the example data: [evidence span withheld — anchor status: external_doc]
- [readme] Runtime annotation fetching rather than bundled data: [evidence span withheld — anchor status: external_doc]
- [readme] Zenodo fallback for example data files: "example_path <- get_cached_file(
  "https://zenodo.org/api/records/17819145/files/summary_stat.csv/content"
)"
