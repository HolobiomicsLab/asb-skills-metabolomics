---
name: feature-intensity-correlation-filtering
description: Use when a RowMerger-merged annotation table contains multiple putative
  annotations per m/z feature and a feature-intensity table is available, so annotations
  can be ranked by adduct probability, feature correlation, and mass error.
when_to_use_negative:
- No feature-intensity table is available — the correlation component of the TPMetrics
  score cannot be computed without feature intensity data
- The annotation table already contains a single annotation per feature — there is
  nothing to prioritize and filtering would be a no-op
- The goal is recognizing isomers/equivalent compounds or removing redundant annotations
  — that is the REname module's task, not TPMetrics scoring
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TurboPutative
  role: Web server (and RESTful API) whose TPMetrics module computes the adduct-probability,
    feature-correlation, and mass-error score and writes the filtered annotation table
  repo: https://proteomics.cnic.es/TurboPutative/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:3ab4f36607fa71b8e8b72b188d229b7e99d32e147f90b218354aad8d48e9c4f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# feature-intensity-correlation-filtering

## Summary

Scores and filters putative metabolite annotations using the TPMetrics module of TurboPutative, which combines user-ranked adduct probabilities, pairwise feature-intensity correlations, and a mass-error penalty into a single annotation score to retain the most probable annotation per feature.

## When to use

Use when a RowMerger-merged annotation table contains multiple putative annotations per m/z feature and a feature-intensity table is available, so annotations can be ranked by adduct probability, feature correlation, and mass error.

## When NOT to use

- No feature-intensity table is available — the correlation component of the TPMetrics score cannot be computed without feature intensity data
- The annotation table already contains a single annotation per feature — there is nothing to prioritize and filtering would be a no-op
- The goal is recognizing isomers/equivalent compounds or removing redundant annotations — that is the REname module's task, not TPMetrics scoring

## Inputs

- RowMerger merged-annotation table (CSV, one row per feature with multiple putative annotations)
- Feature-intensity table (feature m/z / retention time vs. sample intensities)
- User-ranked adduct probabilities per metabolite class
- Retention-time window parameters (narrow user-defined timeframe for annotation-annotation correlation; default 2 min for same-metabolite-class correlation between features)

## Outputs

- tpmetrics_filtered_annotations.csv containing exactly one top-scored annotation per feature
- Reduced annotation dataset (reported 80%-90% reduction of putative annotation entries for massive datasets)

## How to apply

Load the RowMerger merged-annotation table and the corresponding feature-intensity table into the TPMetrics module of TurboPutative (web server or RESTful API). For each putative annotation, compute the adduct-probability score from the user-ranked adduct probabilities defined per metabolite class, and compute pairwise feature-intensity correlations among annotations sharing the same monoisotopic mass within a narrow user-defined retention-time timeframe, using the highest positive correlation coefficient. Additionally compute the same-metabolite-class correlation between features within the wider retention-time window (default 2 min), and apply the mass-error penalty. Combine the adduct-probability, correlation, and mass-error components into a single annotation score and retain only the highest-scoring annotation for each feature. This rationale follows the paper's framing that TPMetrics 'calculates a score based on adduct probability and feature correlations' after combining datasets with feature intensity.

## Related tools

- **TurboPutative** (Web server (and RESTful API) whose TPMetrics module computes the adduct-probability, feature-correlation, and mass-error score and writes the filtered annotation table) — https://proteomics.cnic.es/TurboPutative/

## Evaluation signals

- Output tpmetrics_filtered_annotations.csv contains exactly one top-scored annotation per feature
- Number of retained annotation entries is substantially reduced relative to the RowMerger input (paper reports 80%-90% reduction for massive datasets)
- Each retained annotation carries the highest combined score (adduct probability + correlation + mass-error penalty) among its putative alternatives
- Pairwise correlations were computed only among annotations sharing the same monoisotopic mass within the narrow user-defined timeframe
- Same-metabolite-class correlation between features used the wider retention-time window (default 2 min)

## Limitations

- The score depends on user-defined adduct probability rankings per metabolite class, so results vary with the user's prior adduct choices
- The narrow retention-time timeframe is user-defined, making correlation scores sensitive to this parameter
- The 80%-90% entry-reduction figure is reported for massive datasets; smaller datasets may see different reduction levels
- Requires a feature-intensity table as a separate input dataset to be combined with the annotations

## Evidence

- [abstract] TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] 80-90% entry reduction: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [other] pairwise correlation of annotations sharing monoisotopic mass: [evidence span withheld — anchor status: not_found]
