---
name: cross-dataset-feature-matching
description: Use when two peak-picked, conventionally aligned untargeted LC-MS feature
  tables acquired under non-identical conditions (different gradients, instruments,
  or laboratories) must have their overlapping features identified and concatenated.
when_to_use_negative:
- Datasets acquired from excessively disparate chromatographic techniques (e.g., an
  RPLC dataset aligned to a HILIC dataset), since elution order is not preserved and
  HILIC sets are harder to align
- Datasets in different ionization modes (positive vs negative) or with biologically
  dissimilar samples lacking overlap in metabolomic composition
- Abundance values normalized or transformed in a way that removes ranked abundance
  order information, which the similarity scoring depends on
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metabCombiner
  role: Primary R package implementing data filtering, m/z grouping, anchor selection,
    RT spline fitting, FPA scoring (calcScores, evaluateParams), and combined table
    generation
  repo: https://github.com/hhabra/metabCombiner
- name: mgcv
  role: Underlying engine for fit_gam; generalized additive model spline fitting of
    the RT mapping, with default scat family for heavy-tailed data
- name: R
  role: Execution environment; metabCombiner is distributed as an R package on GitHub
    and Bioconductor
  repo: https://github.com/hhabra/metabCombiner
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03693
    title: '<i>metabCombiner</i>: Paired Untargeted LC-HRMS Metabolomics Feature Matching
      and Concatenation of Disparately Acquired Data Sets'
  source_content_hash: sha256:1595b494eef30246c3d1061c54646774116c23911f34c388359fc1607472977d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hhabra/metabCombiner
  repo_url: https://github.com/hhabra/metabCombiner
  tools:
  - R
  - metabCombiner
  - mgcv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-dataset-feature-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Match overlapping <m/z, retention time> features across two disparately acquired untargeted LC-HRMS metabolomics datasets using metabCombiner: m/z grouping, RT mapping spline fitting, and weighted feature pair alignment (FPA) scoring, followed by concatenation of abundances into a combined table.

## When to use

Use when two peak-picked, conventionally aligned untargeted LC-MS feature tables acquired under non-identical conditions (different gradients, instruments, or laboratories) must have their overlapping features identified and concatenated.

## When NOT to use

- Datasets acquired from excessively disparate chromatographic techniques (e.g., an RPLC dataset aligned to a HILIC dataset), since elution order is not preserved and HILIC sets are harder to align
- Datasets in different ionization modes (positive vs negative) or with biologically dissimilar samples lacking overlap in metabolomic composition
- Abundance values normalized or transformed in a way that removes ranked abundance order information, which the similarity scoring depends on

## Inputs

- Two peak-picked, conventionally aligned untargeted LC-MS metabolomics feature tables (e.g., plasma20 and plasma30 shipped with metabCombiner), same ionization mode
- Training-set table of identified compounds with m/z, retention time, and abundance values (for evaluateParams weight selection)

## Outputs

- Combined table of intersecting feature measurements with concatenated abundances from both datasets
- Scored feature pair alignments with similarity scores (calcScores output, e.g., calc_scores.csv)
- Fitted retention time mapping spline with mean absolute RT prediction error (≈0.06 min on the plasma pair)

## How to apply

Format and filter each dataset (retention time range filter between rtmin and rtmax, missingness filter below the misspc threshold, duplicate removal within close m/z and RT distances such as 0.0025 Da and 0.05 min) with the metabCombiner R package. Group features by m/z values to generate a search space of candidate feature pair alignments (FPAs), then fit a retention time mapping spline through selected anchor ordered pairs using fit_gam (GAM via mgcv, default family 'scat' for heavy-tailed data) or fit_loess. Score each candidate FPA by m/z difference, mapped RT difference, and relative abundance difference using weights A, B, C; select these weights with evaluateParams(), which sweeps a grid to maximize the positive score separation between true FPAs of shared identified training compounds and their misaligned pairings (paper optimum A=100, B=15, C=0.3; vignette region A≈50–70, B≈14–15, C≈0.3–0.4). Finally reduce the combined table to resolve one-to-one matches and concatenate abundances into the combined feature table. The approach presumes same ionization mode, biologically similar samples, and mostly preserved elution order.

## Related tools

- **metabCombiner** (Primary R package implementing data filtering, m/z grouping, anchor selection, RT spline fitting, FPA scoring (calcScores, evaluateParams), and combined table generation) — https://github.com/hhabra/metabCombiner
- **mgcv** (Underlying engine for fit_gam; generalized additive model spline fitting of the RT mapping, with default scat family for heavy-tailed data)
- **R** (Execution environment; metabCombiner is distributed as an R package on GitHub and Bioconductor) — https://github.com/hhabra/metabCombiner

## Examples

```
combined <- calcScores(object = metabCombiner(xdata = plasma20, ydata = plasma30), A = 100, B = 15, C = 0.3)
```

## Evaluation signals

- Mean absolute retention time prediction error of the fitted spline is roughly 0.06 min on the plasma20/plasma30 pair
- Weighted per-compound matching accuracy on annotated test-set compounds reaches approximately 90%
- Weighted per-compound average alignment scores for the 270 annotated test compounds fall between 0.85 and 0.9 when scored with A=100, B=15, C=0.3
- evaluateParams grid sweep identifies an optimal weight region (A≈50–70, B≈14–15, C≈0.3–0.4) and the paper's values (A=100, B=15, C=0.3) yield near-maximal true-vs-misaligned FPA score separation on training compounds
- Most test-set compounds achieve the highest alignment score for all of their variants, confirming correct match selection

## Limitations

- HILIC-acquired datasets are more difficult to align than RPLC datasets
- Preprocessing errors such as incomplete or multipeak integration, low signal-to-noise features, and missed peaks propagate into alignment; preprocessing method and parameters should be chosen carefully per dataset
- Output must be corrected for batch effects before further statistical and bioinformatics analyses
- Elution order must be mostly preserved; RPLC vs HILIC cross-technique alignment is unsuitable

## Evidence

- [abstract] metabCombiner groups features by mass-to-charge (m/z) values to generate a search space of possible feature pair alignments: "metabCombiner groups features by mass-to-charge (m/z) values to generate a search space of possible feature pair alignments"
- [abstract] ranks alignments by m/z, mapped retention time, and relative abundance similarity: "ranks alignments by m/z, mapped retention time, and relative abundance similarity"
- [abstract] achieving a mean absolute retention time prediction error of roughly 0.06 min: "achieving a mean absolute retention time prediction error of roughly 0.06 min"
- [intro] The duplicate filter detects and removes features within close m/z and RT distances: "The duplicate filter detects and removes features within close m/z and RT distances (e.g 0.0025Da & 0.05 min)."
- [intro] They have been acquired in the same ionization mode (positive or negative): "They have been acquired in the same ionization mode (positive or negative)"
- [readme] takes peak-picked and conventionally aligned untargeted LC-MS datasets: "takes peak-picked and conventionally aligned untargeted LC-MS datasets and determines the overlapping <mass-to-charge (m/z), retention time (rt)>"
