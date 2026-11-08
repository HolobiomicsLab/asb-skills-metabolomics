---
name: pattern-component-overlap-analysis
description: Use when you have mined mineMS2 fragmentation patterns and a GNPS molecular
  network whose detected components (groups of similar MS/MS spectra) need interpretation,
  and you want to know which pattern explains each component.
when_to_use_negative:
- You only have raw spectra and no GNPS network components yet — mine patterns and
  build/obtain the molecular network first.
- You want de novo pattern extraction from spectra rather than ranking already-mined
  patterns against network components — use the mineMS2 mining pipeline instead.
- Your goal is to propagate annotations across the network (GNPS-style visualization)
  rather than identify patterns specific to individual components.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: R package providing findPatternsExplainingComponents to score mined patterns
    against GNPS network components with recall/precision/size metrics
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components (spectrum groups linked by
    modified cosine score edges) are the targets being explained
- name: R
  role: Runtime environment in which the mineMS2 R package is called
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:42a76970270205c066bde31b717354ac14e5ab689a22aa18baba55153adb6149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pattern-component-overlap-analysis

## Summary

Couples mineMS2-mined exact fragmentation patterns to GNPS molecular network components and, for each component, ranks which patterns best explain it using recall, precision, and size (F1) metrics via the mineMS2 function findPatternsExplainingComponents.

## When to use

Use when you have mined mineMS2 fragmentation patterns and a GNPS molecular network whose detected components (groups of similar MS/MS spectra) need interpretation, and you want to know which pattern best explains each component.

## When NOT to use

- You only have raw spectra and no GNPS network components yet — mine patterns and build/obtain the molecular network first.
- You want de novo pattern extraction from spectra rather than ranking already-mined patterns against network components — use the mineMS2 mining pipeline instead.
- Your goal is to propagate annotations across the network (GNPS-style visualization) rather than identify patterns specific to individual components.

## Inputs

- Mined mineMS2 fragmentation patterns (ms2Lib pattern objects from a frequent-subgraph mining run)
- GNPS molecular network components with detected MS/MS components (spectra grouped by modified cosine similarity, e.g. from the P. nordicum mgf + pnordicum_ms2_info.tsv metadata)

## Outputs

- Per-component table of top-5 explaining patterns with recall, precision, and size values (e.g. components_patterns_metrics.tsv)
- Best-explaining pattern per component with its F1-score (e.g. P70 for component 8)

## How to apply

Load the mined fragmentation patterns (from the frequent-subgraph mining run) and the GNPS molecular network components with their detected MS/MS components into R using the mineMS2 package. Call findPatternsExplainingComponents with metric set to c('recall','precision','size') and top set to 5, so each of the 9 detected components (e.g. in the P. nordicum 51-spectrum dataset) gets its top-5 explaining patterns. The rationale is that a pattern specific to a component has both recall 1 (present in all component spectra) and high precision (absent outside it), whereas less specific patterns achieve recall 1 with lower precision. Compile the per-component pattern metrics into a table (e.g. components_patterns_metrics.tsv) and inspect the best pattern per component; for component 8 (precursor m/z 370.1283 and 404.0891), the expected winner is pattern P70 with F1-score 1.

## Related tools

- **mineMS2** (R package providing findPatternsExplainingComponents to score mined patterns against GNPS network components with recall/precision/size metrics) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components (spectrum groups linked by modified cosine score edges) are the targets being explained)
- **R** (Runtime environment in which the mineMS2 R package is called)

## Examples

```
findPatternsExplainingComponents(patterns, components, metric = c("recall","precision","size"), top = 5)
```

## Evaluation signals

- The output table covers all detected components (9 in the P. nordicum case) with 5 top patterns each.
- Component 8 (precursor m/z 370.1283 and 404.0891) is best explained by pattern P70 with F1-score 1.
- Non-optimal patterns for component 8 show recall of 1 but lower precision, confirming they are less specific to that pair of spectra.
- Each pattern row carries recall, precision, and size values so specificity versus coverage can be inspected per component.

## Limitations

- mineMS2 exact pattern mining extracts a high number of patterns, so top-k filtering (here top = 5) is needed to focus on relevant ones.
- No scoring system for patterns exists yet; identifying the most informative patterns relies on the recall/precision/size metrics.
- Selection of a subset of spectrum peaks (currently based solely on peak intensities) may result in loss of chemically relevant information before mining.

## Evidence

- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology (Watrous et al., 2012) to focus on patterns that best explain components of the network.: [evidence span withheld — anchor status: not_found]
- [intro] we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [readme] the dataset used in the examples and vignettes consists of 51 spectra from the untargeted study of the secondary metabolism of Penicillium nordicum: [evidence span withheld — anchor status: external_doc]
- [supplementary] the P. nordicum metadata file is shipped with the package and used to build the ms2Lib object: "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
