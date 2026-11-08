---
name: metabolomics-data-preprocessing
description: Use when you have raw LC-MS peak intensity / concentration values for
  hundreds of metabolites across two experimental groups (e.
when_to_use_negative:
- Your data come from a single experimental group — DNEA requires two conditions for
  joint network estimation; use CorrelationCalculator instead
- Your feature table is already collapsed/aggregated or you only need downstream steps
  (BICtune, stabilitySelection, getNetworks, filterNetworks) — the aggregation step
  is explicitly optional
- Missingness is extreme but non-uniform such that dropping metabolites missing in
  >30% of samples would remove the features of interest
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: DNEA R package
  role: Provides createDNEAobject() for object construction and aggregateFeatures()
    for correlation-based, knowledge-based, or hybrid feature aggregation into a collapsed_DNEAobj
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Runtime environment; DNEA installs via BiocManager::install("DNEA") and can
    run on cloud or HPC systems
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:9cbc154ba9f3826f7b10c9b6988a970745dca11b20a1f249712b630e6a7dd132
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-data-preprocessing

## Summary

Step 1 of the DNEA workflow: build a DNEAobject from raw metabolomics peak intensities for two experimental conditions (after removing metabolites missing in >30% of samples and median-imputing the rest), then optionally collapse highly correlated and/or same-chemical-class features into single nodes with aggregateFeatures(), yielding a collapsed_DNEAobj.

## When to use

Use when you have raw LC-MS peak intensity / concentration values for hundreds of metabolites across two experimental groups (e.g. TEDDY 134 metabolites x 322 samples, DM:control vs DM:case) and need to create a DNEAobject and/or reduce correlated or chemically related features before network estimation.

## When NOT to use

- Your data come from a single experimental group — DNEA requires two conditions for joint network estimation; use CorrelationCalculator instead
- Your feature table is already collapsed/aggregated or you only need downstream steps (BICtune, stabilitySelection, getNetworks, filterNetworks) — the aggregation step is explicitly optional
- Missingness is extreme but non-uniform such that dropping metabolites missing in >30% of samples would remove the features of interest

## Inputs

- Raw peak intensity / concentration matrix (metabolites x samples) for two experimental conditions (e.g. TEDDY metabolomics data: 134 metabolites x 322 samples)
- Group labels for the two conditions (e.g. DM:control / DM:case)
- Optional feature_groups data frame mapping metabolite names to chemical classes (e.g. 'BCAAs' for isoleucine, leucine, valine; 'fatty_acids' for metabolites containing 'acid')

## Outputs

- DNEAobject from createDNEAobject()
- collapsed_DNEAobj from aggregateFeatures() with an updated feature list and a populated feature_membership slot (component metabolite names and mean intensity per experimental group per aggregate)
- Saved object file (e.g. collapsed_DNEA.rds)

## How to apply

Filter the feature table to exclude metabolites missing in more than 30% of samples, impute remaining missing values with the per-metabolite median, and call createDNEAobject() with group labels (e.g. DM:control / DM:case). For optional Step 1.2, run aggregateFeatures() in one of three modes: correlation-based (collapse features whose absolute Pearson correlation exceeds a user-supplied correlation_threshold, e.g. 0.7), knowledge-based (aggregate features within user-defined chemical classes, e.g. grouping isoleucine/leucine/valine as 'BCAAs' via a feature_groups data frame), or hybrid (both). Internally the function re-runs createDNEAobject() and returns a collapsed_DNEAobj whose feature_membership slot records each aggregate's component metabolite names and per-group mean intensities. Rationale: collapsing reduces redundancy among correlated metabolites before GLASSO-based joint network estimation under the p >> n regime.

## Related tools

- **DNEA R package** (Provides createDNEAobject() for object construction and aggregateFeatures() for correlation-based, knowledge-based, or hybrid feature aggregation into a collapsed_DNEAobj) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Runtime environment; DNEA installs via BiocManager::install("DNEA") and can run on cloud or HPC systems) — http://www.github.com/Karnovsky-Lab/DNEA/

## Examples

```
collapsed_DNEA <- aggregateFeatures(TEDDYdat, method = "hybrid", correlation_threshold = 0.7); saveRDS(collapsed_DNEA, "collapsed_DNEA.rds")
```

## Evaluation signals

- On the TEDDY demo (134 metabolites), hybrid aggregation with correlation_threshold=0.7 reduces the feature count from 134 to 130; reproduce this reduction as a sanity check
- Every collapsed aggregate satisfies its aggregation criterion: pairwise |Pearson correlation| > correlation_threshold and/or identical entry in the feature_groups chemical-class column
- The returned object is a collapsed_DNEAobj and its feature_membership slot is fully populated with component metabolite names and per-group mean intensities for every aggregate
- Missingness QC invariants: no retained metabolite is missing in >30% of samples, and no NA values remain after median imputation
- Network-level check downstream: total edges across the TEDDY network decrease relative to the un-aggregated object, as expected when redundant features are merged

## Limitations

- Feature aggregation is optional; skipping it leaves redundant correlated features that may inflate network density
- Knowledge-based aggregation depends entirely on user-supplied chemical-class annotations; metabolites without a class default to their own singleton node
- GGM estimation is computationally heavy for large feature sets, so unnecessary features not removed during preprocessing increase compute and memory burden

## Evidence

- [methods] The data were filtered to exclude metabolites that were missing in more than 30% of the samples.: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [methods] For the remaining metabolites, all missing values were imputed with the median value of each metabolite.: "For the remaining metabolites, all missing values were imputed with the median value of each metabolite."
- [methods] For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation.: "For demonstration purposes, we are going to use the "hybrid" approach for feature aggregation."
- [intro] The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation: "The workflow can be divided into four main steps: (1) Data pre-processing and optional feature aggregation"
- [readme] The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions"
