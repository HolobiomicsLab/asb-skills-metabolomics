---
name: mass-spectrometry-benchmark-claim-extraction
description: Use when a metabolomics/mass-spectrometry paper or abstract states numeric
  performance claims (runtimes, speedup factors, latency, accuracy, memory overhead)
  that must be tabulated and traced verbatim to source text.
when_to_use_negative:
- The source text contains no quantitative benchmark claims (e.g. purely qualitative
  method descriptions), so there is nothing to parse into metric/condition/value rows
- Claims appear only in the discussion or supplementary text rather than the abstract,
  since the extraction here is grounded in verbatim abstract statements
- You need to reproduce the benchmark experiments themselves rather than extract the
  paper's reported claim values
tools:
- name: Python
  role: Used to parse each quantitative claim into metric/condition/value fields and
    write benchmark_claims.csv via Python's csv/pandas facilities
- name: EntropySearch
  role: The flash entropy search software whose reported performance claims (speedup,
    query latency, accuracy) are the subject of the extracted benchmark table
  repo: https://github.com/YuanyueLi/EntropySearch/releases
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-023-02012-9
    title: Flash entropy search to query all mass spectral libraries in real time
  source_content_hash: sha256:41b2403b2b5c07820042e047f571d3b0b9bc98915640a987bb59ebf90578aa87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EntropySearch
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# mass-spectrometry-benchmark-claim-extraction

## Summary

Extracts every quantitative performance claim about spectral similarity search from a mass-spectrometry methods paper (here, flash entropy search vs. full dot product/entropy similarity on ~1-billion-spectra libraries) and structures them into a machine-readable benchmark_claims.csv with metric, experimental condition, and reported value.

## When to use

Use when a metabolomics/mass-spectrometry paper or abstract states numeric performance claims (runtimes, speedup factors, latency, accuracy, memory overhead) that must be tabulated and traced verbatim to source text.

## When NOT to use

- The source text contains no quantitative benchmark claims (e.g. purely qualitative method descriptions), so there is nothing to parse into metric/condition/value rows
- Claims appear only in the discussion or supplementary text rather than the abstract, since the extraction here is grounded in verbatim abstract statements
- You need to reproduce the benchmark experiments themselves rather than extract the paper's reported claim values

## Inputs

- Article abstract text (DOI 10.1038/s41592-023-02012-9) containing quantitative performance claims
- List of claim-bearing sentences to parse (verbatim abstract statements)

## Outputs

- benchmark_claims.csv with metric, experimental_condition, and reported_value columns, keyed by metric and condition
- Traceability mapping from each CSV row to a verbatim abstract quote

## How to apply

Read the article abstract (DOI 10.1038/s41592-023-02012-9) and identify every quantitative claim about spectral similarity search, such as the >8 h baseline for full dot product or entropy similarity comparison of a single MS/MS spectrum against 1-billion-entry libraries. Parse each claim with Python into three fields: metric (runtime, speedup factor, query latency, accuracy, memory overhead), experimental condition (baseline vs. flash entropy search; library size of 1 billion spectra), and reported value. Write the parsed claims to benchmark_claims.csv using Python's csv/pandas facilities, keyed by metric and experimental condition. Validate that the CSV exists, parses as valid CSV with the metric/condition/value fields, and that every row's value is traceable to a verbatim abstract statement (e.g. the >8 h baseline and the <2 s / >10,000x flash entropy search result).

## Related tools

- **Python** (Used to parse each quantitative claim into metric/condition/value fields and write benchmark_claims.csv via Python's csv/pandas facilities)
- **EntropySearch** (The flash entropy search software whose reported performance claims (speedup, query latency, accuracy) are the subject of the extracted benchmark table) — https://github.com/YuanyueLi/EntropySearch/releases

## Examples

```
import pandas as pd; rows = [('runtime','baseline full dot product/entropy similarity, 1B spectra','>8 hours'), ('query latency','flash entropy search, 1B spectra','<2 seconds'), ('speedup','flash entropy search vs baseline','>10,000x'), ('accuracy','flash entropy search vs baseline','no loss')]; pd.DataFrame(rows, columns=['metric','condition','value']).to_csv('benchmark_claims.csv', index=False)
```

## Evaluation signals

- benchmark_claims.csv exists and parses as valid CSV
- CSV contains metric, experimental_condition, and reported_value fields
- Every row's reported value is traceable to a verbatim abstract statement (e.g. '>8 h' baseline, '<2 s' and '>10,000x' flash entropy search)
- Each metric row carries the correct experimental condition (baseline full dot product/entropy similarity vs. flash entropy search, 1 billion spectra library size)
- Accuracy-related claims are captured alongside speed claims, reflecting the 'without loss in accuracy' caveat

## Limitations

- Claims are limited to what the abstract states verbatim; discussion-section content for this article was not available for extraction ('_No changelog found._')
- Reported values are paper claims, not independently reproduced measurements; speedup figures depend on the 1-billion-entry library setting and on multiple threads and GPU calculations
- The preprint citation in the README is the Research Square version (10.21203/rs.3.rs-2693233/v1), which may differ from the final Nature Methods text

## Evidence

- [abstract] Flash entropy search accelerates querying to <2 s: "Flash entropy search speeds up calculations more than 10,000 times to query 1 billion spectra in less than 2 seconds, without loss in accuracy."
- [abstract] Baseline comparison takes >8 hours: "Full dot product or entropy similarity comparisons of a single MS/MS spectrum take more than 8 hours."
- [abstract] Library size context for claims: "Public repositories of metabolomics mass spectra encompass more than 1 billion entries."
- [abstract] Hardware-dependence caveat: "It benefits from using multiple threads and GPU calculations."
- [readme] EntropySearch is the software being benchmarked: "Search spectral library with entropy similarity. Download the software from here:"
