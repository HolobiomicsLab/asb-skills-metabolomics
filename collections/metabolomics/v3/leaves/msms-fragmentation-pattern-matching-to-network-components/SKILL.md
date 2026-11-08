---
name: msms-fragmentation-pattern-matching-to-network-components
description: Use when you have a mined mineMS2 pattern library plus a GNPS molecular
  network (components of similar MS/MS spectra) and need to identify which patterns
  specifically explain each network component.
when_to_use_negative:
- Input is a raw spectral library or unmined spectra — pattern mining itself (mineMS2
  mining workflow) must be run first before coupling to components
- The goal is structural annotation of individual spectra without a molecular network
  — use pattern-to-ChemOnt-concept explanation instead
- Spectra come from a probabilistic motif approach such as MS2LDA — mineMS2 patterns
  are exact graphs and this coupling workflow is defined for mineMS2 patterns
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mineMS2
  role: R package providing findPatternsExplainingComponents to rank mined fragmentation
    patterns explaining each GNPS molecular network component by recall, precision,
    and size
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components (groups of similar spectra)
    are matched against the mined patterns
- name: R
  role: Runtime environment in which the mineMS2 R package functions are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:54b1a022a403cf38ff37412b141ad7b3c25c398e99e5a7f279c98637cfd4eb97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# msms-fragmentation-pattern-matching-to-network-components

## Summary

Couples exact fragmentation patterns mined by the mineMS2 R package to GNPS molecular network components, ranking per component the patterns that best explain its spectra using recall, precision, and size metrics.

## When to use

Use when you have a mined mineMS2 pattern library plus a GNPS molecular network (components of similar MS/MS spectra) and need to identify which patterns specifically explain each network component.

## When NOT to use

- Input is a raw spectral library or unmined spectra — pattern mining itself (mineMS2 mining workflow) must be run first before coupling to components
- The goal is structural annotation of individual spectra without a molecular network — use pattern-to-ChemOnt-concept explanation instead
- Spectra come from a probabilistic motif approach such as MS2LDA — mineMS2 patterns are exact graphs and this coupling workflow is defined for mineMS2 patterns

## Inputs

- Mined fragmentation patterns from mineMS2 (ms2Lib object / pattern library, e.g., from the Penicillium nordicum dataset)
- GNPS molecular network components with detected MS/MS components for the same spectra
- MS/MS peak list (mgf) and metadata (e.g., pnordicum.mgf and pnordicum_ms2_info.tsv)

## Outputs

- Per-component table of top-5 explaining patterns with recall, precision, and size values (e.g., components_patterns_metrics.tsv)
- Identified best-explaining pattern (highest F1/precision) per network component

## How to apply

Load the mined fragmentation patterns (e.g., from a prior mineMS2 run on an ms2Lib object such as the 51-spectrum Penicillium nordicum dataset) together with the GNPS molecular network components detected in the same spectra. Call mineMS2's findPatternsExplainingComponents function with the metric argument set to c('recall','precision','size') and top set to 5 to extract, for each detected component, the top-5 explaining patterns with their recall, precision, and size values. Interpret the metrics as specificity: a pattern with recall 1 but lower precision explains the component but is less specific to it, while F1 = 1 indicates a perfect, specific match (e.g., component 8, spectra with precursor m/z 370.1283 and 404.0891, is best explained by pattern P70 with F1 = 1). Compile the per-component pattern metrics into a table (e.g., components_patterns_metrics.tsv) for downstream structural interpretation and annotation of unknown spectra.

## Related tools

- **mineMS2** (R package providing findPatternsExplainingComponents to rank mined fragmentation patterns explaining each GNPS molecular network component by recall, precision, and size) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components (groups of similar spectra) are matched against the mined patterns)
- **R** (Runtime environment in which the mineMS2 R package functions are executed)

## Examples

```
findPatternsExplainingComponents(patterns.m2l, components, metric = c('recall','precision','size'), top = 5)
```

## Evaluation signals

- Output table covers all detected network components (9 components in the P. nordicum case) with 5 patterns each
- For component 8 (precursor m/z 370.1283 and 404.0891), the best explaining pattern is P70 with F1-score 1
- All reported patterns for a component have recall 1 when they explain it; differences appear in precision/size (specificity)
- Metric columns recall, precision, and size are populated for every pattern-component pair in the output table

## Limitations

- Exact pattern mining such as mineMS2 extracts a high number of patterns; no scoring system for identifying the most informative patterns yet exists
- GNPS network parameters must be tuned to obtain a network consistent with the reference study (see Supplementary File 1, Note S2)
- Frequent patterns are only required to appear in at least two spectra, so low-precision patterns that explain a component are common and must be ranked by specificity

## Evidence

- [intro] we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra.: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components and annotate unknown spectra."
- [readme] README coupling statement: [evidence span withheld — anchor status: external_doc]
- [methods] workflow metric setting: [evidence span withheld — anchor status: not_found]
- [results] component 8 finding: [evidence span withheld — anchor status: not_found]
