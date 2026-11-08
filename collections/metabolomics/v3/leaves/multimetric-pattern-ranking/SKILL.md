---
name: multimetric-pattern-ranking
description: Use when you have mined fragmentation patterns from an MS/MS collection
  and a GNPS molecular network, and need to select which patterns explain each detected
  component.
when_to_use_negative:
- When you only need pattern frequency/support across the whole spectral collection
  rather than component-specific explanation (rank by frequency instead)
- When no molecular network components exist (e.g. singletons or an unconnected set
  of spectra) — findPatternsExplainingComponents requires components to score against
- When a scoring system or prior annotation (e.g. MS2LDA motifs from MotifDB) is available,
  since mineMS2 itself currently lacks a pattern scoring system per the discussion
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: mineMS2
  role: R package providing findPatternsExplainingComponents to score patterns against
    network components by recall, precision, and size
  repo: https://github.com/odisce/mineMS2
- name: GNPS
  role: Source of the molecular network whose components are being explained by the
    ranked patterns
- name: R
  role: Runtime environment in which the mineMS2 ranking functions are executed
  repo: https://github.com/odisce/mineMS2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:847bfedaf820464c9fc3d712e3324964445e9b7d82edb11182ebfd472e995cc2
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multimetric-pattern-ranking

## Summary

Ranks mineMS2 fragmentation patterns by how specifically they explain each GNPS molecular network component, using recall, precision, and size (F1-score) metrics via findPatternsExplainingComponents. It identifies, per component, the pattern(s) that are both sensitive and specific to that subset of spectra.

## When to use

Use when you have mined fragmentation patterns from an MS/MS collection and a GNPS molecular network, and need to select which patterns best explain each detected component.

## When NOT to use

- When you only need pattern frequency/support across the whole spectral collection rather than component-specific explanation (rank by frequency instead)
- When no molecular network components exist (e.g. singletons or an unconnected set of spectra) — findPatternsExplainingComponents requires components to score against
- When a scoring system or prior annotation (e.g. MS2LDA motifs from MotifDB) is available, since mineMS2 itself currently lacks a pattern scoring system per the discussion

## Inputs

- Mined fragmentation patterns from mineMS2 (e.g. from a previous mining task on the spectral library)
- GNPS molecular network components with detected MS/MS components (spectra grouped by pairwise modified cosine similarity)
- MS/MS spectra with precursor metadata (e.g. pnordicum.mgf / pnordicum_ms2_info.tsv from the mineMS2 package)

## Outputs

- Per-component table of top-5 explaining patterns with recall, precision, and size metrics (e.g. components_patterns_metrics.tsv)
- Best-explaining pattern per component identified by F1-score (e.g. P70 with F1=1 for component 8)

## How to apply

Load the mined fragmentation patterns (from the frequent-subgraph mining step) and the GNPS molecular network components with their detected MS/MS components, then run the mineMS2 function findPatternsExplainingComponents with metric set to c('recall','precision','size') and top set to 5. For each component (e.g. the 9 components detected in the P. nordicum dataset), extract the top-5 explaining patterns with their recall, precision, and size values and compile them into a table such as components_patterns_metrics.tsv. Prefer patterns with F1-score 1 (recall and precision both 1) as the best explanation: e.g. for component 8 (spectra with precursor m/z 370.1283 and 404.0891), pattern P70 scored F1=1 while other patterns had recall 1 but lower precision, meaning they are less specific to that pair of spectra. When filtering by specificity, the article's convention is to keep the deepest, most specific concept when supports are identical (F1-score >0.6 was used for ChemOnt concept selection).

## Related tools

- **mineMS2** (R package providing findPatternsExplainingComponents to score patterns against network components by recall, precision, and size) — https://github.com/odisce/mineMS2
- **GNPS** (Source of the molecular network whose components are being explained by the ranked patterns)
- **R** (Runtime environment in which the mineMS2 ranking functions are executed) — https://github.com/odisce/mineMS2

## Examples

```
findPatternsExplainingComponents(minedPatterns, gnpsComponents, metric = c('recall','precision','size'), top = 5)
```

## Evaluation signals

- Output table covers all expected components (9 for the P. nordicum dataset) with 5 patterns each
- Component 8 (precursor m/z 370.1283 and 404.0891) has P70 as best pattern with F1-score of 1
- Patterns other than the best one have recall 1 but precision < 1 for component 8, confirming lower specificity
- Each reported pattern carries all three metric values (recall, precision, size), not just a single score
- Selected patterns for downstream use satisfy explicit thresholds (e.g. F1-score >0.6, taxonomy depth >=3 when applied to ChemOnt concepts)

## Limitations

- Exact pattern mining extracts a high number of patterns, so component-specific ranking is needed to prune them
- No general scoring system for patterns yet exists; a pattern-scoring approach is proposed as future work
- A dedicated database of patterns of interest with a query metric does not yet exist and is planned future work

## Evidence

- [methods] Run the mineMS2 function findPatternsExplainingComponents with metric set to c('recall','precision','size') and top set to 5.: [evidence span withheld — anchor status: not_found]
- [readme] README coupling quote: [evidence span withheld — anchor status: not_found]
- [results] Component 8 best pattern: [evidence span withheld — anchor status: not_found]
- [results] ChemOnt selection thresholds: [evidence span withheld — anchor status: not_found]
