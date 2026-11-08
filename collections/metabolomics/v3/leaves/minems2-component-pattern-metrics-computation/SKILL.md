---
name: minems2-component-pattern-metrics-computation
description: Use when you have a GNPS molecular network (components of similar MS/MS
  spectra) plus a set of mineMS2-mined fragmentation patterns, and need to identify
  which patterns explain each network component using recall/precision/size metrics.
when_to_use_negative:
- You only need to mine patterns de novo from spectra — use the mineMS2 pattern-mining
  workflow instead; this skill assumes patterns and network components already exist
- You want to annotate patterns with chemical concepts (ChemOnt terms) rather than
  rank patterns against network components — that is a different mineMS2 use case
- The spectra have not been organized into a molecular network (no GNPS components),
  since there are no components to explain
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: mineMS2
  role: R package providing findPatternsExplainingComponents, which ranks mined fragmentation
    patterns against GNPS network components by recall, precision, and size metrics
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components (groups of similar MS/MS
    spectra) are the targets being explained
- name: R
  role: Runtime environment for executing the mineMS2 R package functions and exporting
    the metrics table as TSV
  repo: https://github.com/odisce/mineMS2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:5f8d8e5669f71311cb4e5ebc98a3e18a7bac105980764dc3eeb88ade41e8d1a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mineMS2-component-pattern-metrics-computation

## Summary

Ranks mined mineMS2 fragmentation patterns by how specifically they explain each component of a GNPS molecular network, computing recall, precision, and size metrics per pattern-component pair. This reproduces the paper's analysis of the P. nordicum dataset where, e.g., pattern P70 uniquely explains component 8 with F1-score 1.

## When to use

Use when you have a GNPS molecular network (components of similar MS/MS spectra) plus a set of mineMS2-mined fragmentation patterns, and need to identify which patterns best explain each network component using recall/precision/size metrics.

## When NOT to use

- You only need to mine patterns de novo from spectra — use the mineMS2 pattern-mining workflow instead; this skill assumes patterns and network components already exist
- You want to annotate patterns with chemical concepts (ChemOnt terms) rather than rank patterns against network components — that is a different mineMS2 use case
- The spectra have not been organized into a molecular network (no GNPS components), since there are no components to explain

## Inputs

- mineMS2 fragmentation patterns previously mined from the MS/MS spectra collection (e.g. from a findPatterns run on the Penicillium nordicum ms2Lib object)
- GNPS molecular network components with detected MS/MS components (spectra grouped by modified cosine similarity)
- ms2Lib object built from pnordicum.mgf with supplementary metadata pnordicum_ms2_info.tsv (system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"))

## Outputs

- components_patterns_metrics.tsv — per-component table of top-5 explaining patterns with recall, precision, and size values for each of the 9 detected GNPS network components
- Per-component best-explaining pattern identification (e.g. P70 for component 8 with F1-score 1)

## How to apply

Load the mineMS2 fragmentation patterns (previously mined from the MS/MS collection, e.g. the 51-spectrum Penicillium nordicum ms2Lib built with pnordicum.mgf and pnordicum_ms2_info.tsv metadata) and the GNPS molecular network components with their detected MS/MS components. Call the mineMS2 function findPatternsExplainingComponents with the metric argument set to c('recall','precision','size') and top set to 5, so that for each detected component the top-5 explaining patterns are returned with their recall, precision, and size values. Compile the per-component pattern metrics into a table (components_patterns_metrics.tsv). Interpretation follows the paper's convention: a pattern with recall 1 explains all spectra of the component, while a high precision (yielding F1 = 1) indicates the pattern is specific to that component; e.g., in component 8 (precursor m/z 370.1283 and 404.0891), P70 achieves F1-score 1 whereas competing patterns have recall 1 but lower precision. Validate that the table covers all detected components (9 in the paper's P. nordicum network) with 5 patterns each.

## Related tools

- **mineMS2** (R package providing findPatternsExplainingComponents, which ranks mined fragmentation patterns against GNPS network components by recall, precision, and size metrics) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components (groups of similar MS/MS spectra) are the targets being explained)
- **R** (Runtime environment for executing the mineMS2 R package functions and exporting the metrics table as TSV) — https://github.com/odisce/mineMS2

## Examples

```
findPatternsExplainingComponents(ms2l, network, metric = c("recall", "precision", "size"), top = 5)
```

## Evaluation signals

- The output table covers all detected network components (9 in the paper's P. nordicum analysis) with 5 patterns per component
- Component 8 (spectra with precursor m/z 370.1283 and 404.0891) has P70 as its best explaining pattern with an F1-score of 1
- Non-best patterns for component 8 show recall of 1 but lower precision, confirming they are less specific to that pair of spectra
- All recall/precision values fall in [0, 1], with precision <= recall when the pattern is contained in the component
- Metrics are reproducible from the vignette workflow (mineMS2_coupling-to-gnps.html) when run on the bundled P. nordicum dataset

## Limitations

- Exact pattern mining extracts a high number of patterns, so ranking by component-explanation metrics is needed to prioritize them
- A scoring system for patterns does not yet exist; the proposed approach of building a database of patterns of interest with a dedicated query metric is future work
- Peak subset selection due to combinatorial complexity is currently based solely on peak intensities, which may lose chemically relevant information

## Evidence

- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology: [evidence span withheld — anchor status: external_doc]
- [readme] Vignette covers focusing on patterns that best explain components of the molecular network: [evidence span withheld — anchor status: external_doc]
- [intro] Objective: coupling mineMS2 to GNPS to interpret detected components: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [results] Pattern selection thresholds from the paper's component/concept explanation evaluation: [evidence span withheld — anchor status: not_found]
