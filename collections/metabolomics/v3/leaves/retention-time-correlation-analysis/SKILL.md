---
name: retention-time-correlation-analysis
description: Use when an untargeted metabolomics feature table contains multiple putative
  annotations per feature (shared monoisotopic mass and adduct variants) and a single
  most-probable annotation per feature must be selected using adduct probability,
  feature-intensity correlations, and mass error.
when_to_use_negative:
- Input is a raw or unmerged annotation list that has not been through RowMerger,
  so multiple rows per feature have not yet been consolidated
- No feature-intensity table across samples is available, so the pairwise feature-intensity
  correlations required by TPMetrics cannot be computed
- The goal is manual comparison among annotations or classification of compounds rather
  than automated single-annotation-per-feature prioritization (Tagger/REname/RowMerger
  cover those tasks)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web server whose TPMetrics module computes the multi-criteria score (adduct
    probability, feature correlations, mass-error penalty) and filters to one annotation
    per feature; also usable programmatically via its RESTful API
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:c36f09fe0d12b10c5ed8ebbdf428d6e0492193dd441f07ced2ef15aac5cd970d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# retention-time-correlation-analysis

## Summary

Scores and prioritizes putative metabolite annotations by computing pairwise feature-intensity correlations among annotations sharing the same monoisotopic mass within a narrow retention-time window, combined with user-ranked adduct probabilities and a mass-error penalty, as implemented in the TPMetrics module of TurboPutative.

## When to use

Use when an untargeted metabolomics feature table contains multiple putative annotations per feature (shared monoisotopic mass and adduct variants) and a single most-probable annotation per feature must be selected using adduct probability, feature-intensity correlations, and mass error.

## When NOT to use

- Input is a raw or unmerged annotation list that has not been through RowMerger, so multiple rows per feature have not yet been consolidated
- No feature-intensity table across samples is available, so the pairwise feature-intensity correlations required by TPMetrics cannot be computed
- The goal is manual comparison among annotations or classification of compounds rather than automated single-annotation-per-feature prioritization (Tagger/REname/RowMerger cover those tasks)

## Inputs

- RowMerger merged-annotation table (e.g. demo merged-annotation CSV from the TurboPutative web server)
- Feature-intensity table (e.g. demo feature-intensity dataset supplied on the TurboPutative web server)
- User-ranked adduct probabilities defined per metabolite class
- Retention-time window parameters (narrow window for same-feature correlations; wider window, default 2 min, for same-metabolite-class correlations)

## Outputs

- Filtered annotation table with exactly one top-scored annotation per feature (e.g. tpmetrics_filtered_annotations.csv)
- Per-annotation combined scores composed of adduct-probability, correlation, and mass-error components

## How to apply

Start from a RowMerger merged-annotation table plus a feature-intensity table and, optionally, user-ranked adduct probabilities defined per metabolite class. For each putative annotation, compute an adduct-probability score from the ranked adduct list, then compute pairwise correlations of feature intensities across samples for annotations of the same feature within a narrow retention-time window, retaining the highest positive correlation coefficient; additionally compute a same-metabolite-class correlation between features within a wider retention-time window (default 2 min). Apply a mass-error penalty and combine the adduct-probability, correlation, and mass-error components into a single annotation score. Retain only the highest-scoring annotation per feature and write the filtered table (e.g. tpmetrics_filtered_annotations.csv); TurboPutative reports an 80%-90% reduction of putative annotation entries for massive datasets.

## Related tools

- **TurboPutative** (Web server whose TPMetrics module computes the multi-criteria score (adduct probability, feature correlations, mass-error penalty) and filters to one annotation per feature; also usable programmatically via its RESTful API) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Evaluation signals

- Output table contains exactly one top-scored annotation per feature
- Number of retained annotation entries is substantially reduced relative to the RowMerger input (paper reports 80%-90% reduction for massive datasets)
- Each retained annotation's score reflects the combination of adduct-probability, correlation, and mass-error components as configured
- Same-feature correlation uses the highest positive correlation coefficient among features sharing the same monoisotopic mass within the narrow timeframe
- Same-metabolite-class correlations are computed within the wider retention-time window (default 2 min)

## Limitations

- Feature-intensity correlations require a separate intensity dataset combined with the annotation table; without it, correlation scoring cannot be performed
- Adduct-probability ranking depends on user-defined per-metabolite-class priors, so scores are sensitive to those rankings
- Retention-time window choices (narrow same-feature window; 2 min default for class correlations) constrain which correlations can be detected

## Evidence

- [abstract] TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations: "TPMetrics combines different datasets with feature intensity and relevant information for the researcher and calculates a score based on adduct probability and feature correlations"
- [abstract] TurboPutative reduces putative annotations by 80%-90% in massive datasets: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] TurboPutative workflow can be automated via RESTful API: "TurboPutative comprises a rapid, automated, and customizable workflow that can also be included in programmed bioinformatics pipelines through its RESTful API services"
