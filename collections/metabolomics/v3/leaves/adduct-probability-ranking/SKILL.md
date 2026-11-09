---
name: adduct-probability-ranking
description: Use when a RowMerger-merged annotation table contains multiple putative
  annotations per feature and a feature-intensity table is available, and the goal
  is to prioritize the most probable annotation per feature.
when_to_use_negative:
- The dataset has only one putative annotation per feature, so there is nothing to
  prioritize or reduce
- No feature-intensity table is available, since TPMetrics requires feature intensity
  to compute the correlation score
- The goal is compound classification or biological-system insight rather than annotation
  prioritization (use the Tagger module instead)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web platform whose TPMetrics module computes the multi-criteria annotation
    score and filters to the top-scoring annotation per feature
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:fb3fb69e61b59c25582e7429b0b1f1e91169e7f50fca59163f7ac1e92a76c946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# adduct-probability-ranking

## Summary

Ranking and filtering of multiple putative metabolite annotations per feature using TPMetrics from TurboPutative, which scores each annotation by user-ranked adduct probability, feature-intensity correlations, and a mass-error penalty to retain only the most probable annotation.

## When to use

Use when a RowMerger-merged annotation table contains multiple putative annotations per feature and a feature-intensity table is available, and the goal is to prioritize the most probable annotation per feature.

## When NOT to use

- The dataset has only one putative annotation per feature, so there is nothing to prioritize or reduce
- No feature-intensity table is available, since TPMetrics requires feature intensity to compute the correlation score
- The goal is compound classification or biological-system insight rather than annotation prioritization (use the Tagger module instead)

## Inputs

- RowMerger merged-annotation table (multiple putative annotations per feature)
- Feature-intensity table
- User-ranked adduct probabilities defined per metabolite class
- Retention-time window setting (default 2 min for the wider same-class correlation window)

## Outputs

- tpmetrics_filtered_annotations.csv with exactly one top-scored annotation per feature
- Per-annotation scores combining adduct probability, feature-intensity correlation, and mass-error penalty

## How to apply

Load the RowMerger merged-annotation table together with the feature-intensity table (e.g., the demo datasets on the TurboPutative web server at https://proteomics.cnic.es/TurboPutative/). For each putative annotation, compute the adduct-probability score from user-ranked adduct probabilities defined per metabolite class. Compute a pairwise feature-intensity correlation score for putative annotations sharing the same feature within a narrow user-defined retention-time window, considering the highest positive correlation coefficient, plus a same-metabolite-class correlation score between features within the wider window (default 2 min). Apply the mass-error penalty and combine adduct-probability, correlation, and mass-error components into a single annotation score. Retain only the highest-scoring annotation per feature and write the result to tpmetrics_filtered_annotations.csv; the paper reports an 80%-90% reduction of putative annotation entries for massive datasets.

## Related tools

- **TurboPutative** (Web platform whose TPMetrics module computes the multi-criteria annotation score and filters to the top-scoring annotation per feature) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- The output file tpmetrics_filtered_annotations.csv contains exactly one top-scored annotation per feature
- The number of retained annotation entries is substantially reduced relative to the RowMerger input, matching the reported 80%-90% reduction for massive datasets
- Each retained annotation's score reflects the highest positive correlation coefficient among features sharing the same monoisotopic mass within the retention-time window
- The mass-error penalty was applied so that annotations with larger mass error score lower when adduct-probability and correlation components are comparable

## Limitations

- Correlation scoring depends on the user-defined retention-time windows and the default wider window of 2 min; inappropriate windows can misgroup features
- The pairwise correlation component considers only the highest positive correlation coefficient among features, so weaker corroborating correlations do not contribute
- Requires user-ranked adduct probabilities per metabolite class; mis-ranked adduct priors directly bias the final annotation score

## Evidence

- [abstract] TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] reduction of entries: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [results] pairwise correlation definition: [evidence span withheld — anchor status: not_found]
