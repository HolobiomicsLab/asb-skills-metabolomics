---
name: gnps-molecular-network-interpretation
description: Use when you have a GNPS molecular network with detected components (e.g.
  pairs or clusters of similar MS/MS spectra) and a library of mineMS2 fragmentation
  patterns, and you want the top patterns explaining each component.
when_to_use_negative:
- Input is only a raw GNPS network without previously mined mineMS2 fragmentation
  patterns — run the pattern mining step first.
- Goal is annotating individual spectra against spectral libraries rather than interpreting
  whole molecular network components.
- Components are formed by methods other than GNPS modified-cosine molecular networking
  without adaptation of the component-matching logic.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mineMS2
  role: R package providing findPatternsExplainingComponents to rank patterns explaining
    GNPS network components by recall/precision/size
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components are interpreted; each vertex
    is a spectrum, edges indicate high pairwise modified cosine similarity
- name: R
  role: Runtime environment for the mineMS2 R package workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:dda0fcdf3dbe05bdf88616c2fad8e3fbcc01ac386f56d87f99583667127b86ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gnps-molecular-network-interpretation

## Summary

Couple mineMS2-mined exact fragmentation patterns to a GNPS molecular network to identify, per network component, the patterns that best explain the spectra, scored by recall, precision, and size metrics.

## When to use

Use when you have a GNPS molecular network with detected components (e.g. pairs or clusters of similar MS/MS spectra) and a library of mineMS2 fragmentation patterns, and you want the top patterns explaining each component.

## When NOT to use

- Input is only a raw GNPS network without previously mined mineMS2 fragmentation patterns — run the pattern mining step first.
- Goal is annotating individual spectra against spectral libraries rather than interpreting whole molecular network components.
- Components are formed by methods other than GNPS modified-cosine molecular networking without adaptation of the component-matching logic.

## Inputs

- Mined mineMS2 fragmentation patterns (output of pattern mining on the MS/MS collection)
- GNPS molecular network components with their detected MS/MS components (spectra grouped by modified cosine similarity edges)

## Outputs

- Per-component table of top-5 explaining patterns with recall, precision, and size values (components_patterns_metrics.tsv)

## How to apply

Load the mined fragmentation patterns (from a prior mineMS2 pattern-mining step) and the GNPS molecular network components with their detected MS/MS components. In R, call the mineMS2 function findPatternsExplainingComponents with metric set to c('recall','precision','size') and top set to 5 to rank, for each component, the patterns whose m/z-difference graphs are shared by the component's spectra. For each of the detected components (e.g. 9 components in the Penicillium nordicum dataset), extract the top-5 explaining patterns with their recall, precision, and size values and compile them into a table such as components_patterns_metrics.tsv. Recall indicates the fraction of component spectra containing the pattern, precision measures specificity of the pattern to that component, and size reports the pattern's graph size; specificity matters because patterns can have recall of 1 yet lower precision. Validate by checking coverage of all components and comparing best patterns against published results (e.g. component 8, precursor pair m/z 370.1283 and 404.0891, best explained by pattern P70 with F1-score 1).

## Related tools

- **mineMS2** (R package providing findPatternsExplainingComponents to rank patterns explaining GNPS network components by recall/precision/size) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components are interpreted; each vertex is a spectrum, edges indicate high pairwise modified cosine similarity)
- **R** (Runtime environment for the mineMS2 R package workflow)

## Examples

```
findPatternsExplainingComponents(patterns, components, metric = c('recall','precision','size'), top = 5)
```

## Evaluation signals

- The metrics table covers all detected components (e.g. 9 components in the P. nordicum dataset) with 5 patterns each.
- Component 8 (precursor m/z 370.1283 and 404.0891) is best explained by pattern P70 with F1-score 1, matching the published result.
- Other patterns for component 8 have recall of 1 but lower precision, confirming they are less specific to that pair of spectra.
- Each reported pattern carries concrete recall, precision, and size values rather than only a ranking.

## Limitations

- Exact pattern mining such as mineMS2 extracts a high number of patterns, so selection of explaining patterns is needed; no scoring system for patterns yet exists.
- Pattern mining requires selecting a subset of spectrum peaks (currently based solely on peak intensities), which may lose chemically relevant information.
- Runtime scales with collection size: mineMS2 can process up to 3,000 spectra in a few hours on a standard laptop.

## Evidence

- [results] For component 8 (the pair of spectra with precursor m/z 370.1283 and 404.0891), the best explaining pattern is P70 with an F1-score of 1: [evidence span withheld — anchor status: not_found]
- [readme] *mineMS2* can be further **coupled to the GNPS MS/MS molecular networking** methodology: [evidence span withheld — anchor status: external_doc]
- [intro] each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score"
- [intro] coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
