---
name: metabolomics-benchmark-reproduction
description: Use when a paper reports a quantitative mapping or annotation rate for
  a metabolomics identifier-resolution tool on a named published dataset and you need
  to independently verify that number.
when_to_use_negative:
- The goal is novel mapping of your own metabolomics data rather than reproducing
  a published benchmark number on the paper's dataset
- The dataset or tool version differs from the published benchmark conditions, making
  rate comparison to the paper's figure invalid
- You need statistical significance testing of tool superiority — the paper reports
  rates graphically (Figure 1C) without quantitative performance metrics or thresholds
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaFetcheR
  role: Identifier-mapping tool whose published benchmark (non-empty-field mapping
    rate) is reproduced on the Priolo et al. 2014 dataset
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: MetaboAnalyst
  role: Comparator tool whose ~62% mapping rate on the same input identifiers serves
    as the benchmark reference
- name: R
  role: Implementation language; benchmark script runs via devtools installation and
    the resolve_metabolites call
- name: PostgreSQL
  role: Local cache database built from HMDB, ChEBI and LIPID MAPS dump files via
    bulk-insertion functions before benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:bd68e0fcac4c04fce98f1287ba4559774590028d89fa9a92790db7681ee929f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/komorowskilab/MetaFetcheR
  repo_url: https://github.com/komorowskilab/MetaFetcheR
  tools:
  - MetaFetcheR
  - MetaboAnalyst
  - PostgreSQL
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-benchmark-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a published metabolite-identifier mapping benchmark by re-running the mapping tool on the same input dataset and comparing computed mapping rates to the paper's reported figures (e.g. MetaFetcheR ~78% vs MetaboAnalyst ~62% on Priolo et al. 2014, Figure 1C).

## When to use

Use when a paper reports a quantitative mapping or annotation rate for a metabolomics identifier-resolution tool on a named published dataset and you need to independently verify that number.

## When NOT to use

- The goal is novel mapping of your own metabolomics data rather than reproducing a published benchmark number on the paper's dataset
- The dataset or tool version differs from the published benchmark conditions, making rate comparison to the paper's figure invalid
- You need statistical significance testing of tool superiority — the paper reports rates graphically (Figure 1C) without quantitative performance metrics or thresholds

## Inputs

- Benchmark dataset metabolite identifier table (Priolo et al. 2014 metabolite identifiers as a data frame)
- Database dump files for HMDB, ChEBI and LIPID MAPS
- Reference mapping rates from the paper (Figure 1C: ~78% MetaFetcheR, ~62% MetaboAnalyst)
- Local PostgreSQL database instance for the cache

## Outputs

- Computed non-empty-field mapping rate for the reproduced run
- CSV file (e.g. mapping_rate_priolo2014.csv) containing the computed rate plus the MetaFetcheR (~78%) and MetaboAnalyst (~62%) reference rates

## How to apply

Install the tool exactly as the authors specify (for MetaFetcheR: devtools::install_github("shizidushu/hfun","komorowskilab/metafetcher") since the install depends on the hfun package). Reconstruct the tool's required environment before running: build the local PostgreSQL cache database from HMDB, ChEBI and LIPID MAPS dump files via the bulk-insertion functions (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps, bulk_insert_secondary_ids), while KEGG and PubChem records are cached on the fly via http_call_api. Load the benchmark dataset's metabolite identifier table (here the Priolo et al. 2014 discovered-data frame) and run the resolver (resolve_metabolites(df.discovered)), letting the queue-based algorithm reiterate until all identifiers are retrieved or cannot be further resolved. Compute the same metric the paper used — the non-empty-field mapping rate, i.e. the fraction of resolved (non-empty) identifier fields over input entries — and compare it to the reference values from the paper's figure. Because the paper states no exact acceptance threshold, judge agreement qualitatively (the computed rate should be close to the reported ~78%), and record all three numbers (computed, tool reference, comparator reference) in an output CSV for auditability.

## Related tools

- **MetaFetcheR** (Identifier-mapping tool whose published benchmark (non-empty-field mapping rate) is reproduced on the Priolo et al. 2014 dataset) — https://github.com/komorowskilab/MetaFetcheR/
- **MetaboAnalyst** (Comparator tool whose ~62% mapping rate on the same input identifiers serves as the benchmark reference)
- **R** (Implementation language; benchmark script runs via devtools installation and the resolve_metabolites call)
- **PostgreSQL** (Local cache database built from HMDB, ChEBI and LIPID MAPS dump files via bulk-insertion functions before benchmarking)

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher"); rate <- resolve_metabolites(df.discovered); write.csv(data.frame(rate=rate, metafetcher_ref=0.78, metaboanalyst_ref=0.62), "mapping_rate_priolo2014.csv")
```

## Evaluation signals

- The R script runs to completion without errors, including the PostgreSQL bulk-insertion phase and resolve_metabolites termination
- The output CSV contains all three values: the computed mapping rate, the ~78% MetaFetcheR reference, and the ~62% MetaboAnalyst reference
- The computed MetaFetcheR rate is consistent (expert-reviewed closeness) with the approximately 78% value reported in Figure 1C, since no exact threshold is stated in the paper
- The queue-based resolution loop terminates only when all identifiers are retrieved or cannot be further resolved, not on a timeout or silent failure
- The computed rate exceeds the MetaboAnalyst ~62% comparator rate, consistent with the paper's claim of outperformance

## Limitations

- The paper reports the benchmark graphically (Figure 1C) with no exact numeric acceptance threshold, so agreement must be judged qualitatively
- No version number is stated for MetaFetcheR, and benchmark datasets are cited only by publication reference without accession or data-availability information, complicating exact reproduction
- The paper claims outperformance over MS_targeted and MetaboAnalyst without quantitative performance metrics in the discussion
- Unique MetaFetcheR identifiers are only a planned future feature, and KEGG/PubChem caching depends on live HTTP calls, which can change over time

## Evidence

- [results] On the Priolo et al. 2014 dataset, MetaFetcheR achieved a non-empty fields mapping rate of ~78%, while MetaboAnalyst achieved a ~62% mapping rate, as shown in Figure 1C.: [evidence span withheld — anchor status: not_found]
