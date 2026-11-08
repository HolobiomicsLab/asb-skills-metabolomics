---
name: mass-error-penalty-scoring
description: Decision support for untargeted metabolomics annotation tables (e.g.,
  RowMerger output) with multiple putative identifications per feature and accompanying
  feature-intensity data, when putative entries must be prioritized and reduced.
when_to_use_negative:
- Input is already a reduced annotation table with a single annotation per feature
  — no prioritization is needed
- No feature-intensity dataset is available, since TPMetrics combines the annotation
  table with feature intensity to compute the correlation-based score
- The goal is compound classification rather than annotation prioritization — use
  TurboPutative's Tagger module instead
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TurboPutative
  role: Web platform whose TPMetrics module implements the adduct-probability, feature-correlation,
    and mass-error scoring and emits the filtered annotation table
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:a94f61960133fb1e3dca7da679ffe9f97e53cd629591ef923593cd98b73f6b10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-error-penalty-scoring

## Summary

A multi-criteria scoring procedure from TurboPutative's TPMetrics module that combines adduct-probability, feature-intensity correlation, and a mass-error penalty into a single annotation score, retaining only the top-scoring putative identification per feature.

## When to use

Decision support for untargeted metabolomics annotation tables (e.g., RowMerger output) with multiple putative identifications per feature and accompanying feature-intensity data, when putative entries must be prioritized and reduced.

## When NOT to use

- Input is already a reduced annotation table with a single annotation per feature — no prioritization is needed
- No feature-intensity dataset is available, since TPMetrics combines the annotation table with feature intensity to compute the correlation-based score
- The goal is compound classification rather than annotation prioritization — use TurboPutative's Tagger module instead

## Inputs

- RowMerger merged-annotation table (e.g., demo merged-annotation table from the TurboPutative web server)
- Feature-intensity table (e.g., demo feature-intensity table supplied as an example dataset on the TurboPutative web server)
- User-ranked adduct probabilities defined per metabolite class
- User-defined retention-time window (default 2 min for the same-metabolite-class correlation)

## Outputs

- tpmetrics_filtered_annotations.csv containing exactly one top-scored annotation per feature
- Per-annotation scores combining adduct-probability, feature-correlation, and mass-error components

## How to apply

Load the RowMerger merged-annotation table together with the feature-intensity table into TPMetrics. For each putative annotation, compute the adduct-probability score using user-ranked adduct probabilities defined per metabolite class. Compute the pairwise feature-intensity correlation score for putative annotations of the same feature within a narrow retention-time window, and a same-metabolite-class correlation score between features within the wider default 2 min retention-time window; the correlation component considers the highest positive correlation coefficient among features sharing the same monoisotopic mass. Apply the mass-error penalty and combine the adduct-probability, correlation, and mass-error components into a single annotation score. Retain only the highest-scoring annotation per feature and write the filtered table to tpmetrics_filtered_annotations.csv; the expected outcome on massive datasets is an 80%-90% reduction of putative annotation entries.

## Related tools

- **TurboPutative** (Web platform whose TPMetrics module implements the adduct-probability, feature-correlation, and mass-error scoring and emits the filtered annotation table) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- tpmetrics_filtered_annotations.csv contains exactly one top-scored annotation per feature
- Number of retained entries is substantially reduced relative to the RowMerger input — the paper reports an 80%-90% reduction of putative annotation entries for massive datasets
- Each retained annotation carries a combined score whose components (adduct probability, correlation, mass-error penalty) were all computed, not the correlation coefficient alone
- Correlation computations respect the retention-time windows: narrow per-feature window for intensity correlation and the wider (default 2 min) window for same-metabolite-class correlation between features

## Limitations

- Requires feature-intensity data and user-defined adduct-probability rankings per metabolite class; without them the multi-criteria score cannot be computed
- The same-metabolite-class correlation depends on a user-defined retention-time window (default 2 min), so window choice affects which features are compared
- Only the highest-scoring annotation per feature is retained, so lower-ranked putative identifications are discarded from the filtered output

## Evidence

- [abstract] TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] 80%-90% reduction of putative entries: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] Four-module web platform including TPMetrics: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
