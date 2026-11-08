---
name: npeaks-multiple-match-cluster-evaluation
description: Use when an M2S-style matching of two untargeted LC-MS metabolomics feature
  lists (matched on RT, m/z, FI) leaves clusters of multiple candidate matches, and
  you need to gauge match quality via the unique-to-multiple-match ratio and npeaks
  differences.
when_to_use_negative:
- Match tables in which every reference feature has exactly one target match (no multiple-match
  clusters), leaving nothing for the npeaks cluster evaluation to score
- Feature lists lacking sample-level detection counts (npeaks), e.g., merged peak
  tables that only retain RT/m/z/FI medians
- Matching run only up to step 1 (initial thresholds) before step 2 unique-correspondence
  resolution, when clusters of candidate matches are not yet defined
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox that produces the match tables and unique/multiple-match cluster
    structure being evaluated by this npeaks-based validation strategy
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment in which the M2S toolbox runs and in which the npeaks
    cluster statistics are computed and tabulated
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Peak-picking tool that generated the feature tables whose per-feature sample-detection
    counts (npeaks) feed the cluster comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:7f19627c2083ca86feb62d3412c8b142e94227a16d90986566aa4a95248feff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - MESA
  - Rotterdam
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# npeaks-multiple-match-cluster-evaluation

## Summary

One of six orthogonal matching-quality validation strategies from Climaco Pinto et al. (Anal. Chem. 2022): quantify how many matched feature pairs remain ambiguous (multiple-match clusters) versus unique, and use the npeaks count (number of samples in which a feature was detected) to score whether the selected match in each cluster is the more reliable one.

## When to use

Use when an M2S-style matching of two untargeted LC-MS metabolomics feature lists (matched on RT, m/z, FI) leaves clusters of multiple candidate matches, and you need to gauge match quality via the unique-to-multiple-match ratio and npeaks differences.

## When NOT to use

- Match tables in which every reference feature has exactly one target match (no multiple-match clusters), leaving nothing for the npeaks cluster evaluation to score
- Feature lists lacking sample-level detection counts (npeaks), e.g., merged peak tables that only retain RT/m/z/FI medians
- Matching run only up to step 1 (initial thresholds) before step 2 unique-correspondence resolution, when clusters of candidate matches are not yet defined

## Inputs

- Final match table for Dataset 1 (RP UPLC MS ESI+, MESA reference vs Rotterdam target) after step 3 poor-match removal
- Final match table for Dataset 2 (RP UPLC MS ESI-) after step 3 poor-match removal
- XCMS peak-picked feature tables per dataset, including npeaks (number of samples in which each feature was detected)
- Sample intensity matrices for matched features in reference and target datasets
- Cluster assignments from M2S step 2 (unique correspondence) identifying multiple-match clusters

## Outputs

- Ratio of unique matches to multiple-match clusters per dataset (e.g., Dataset 1: 61 clusters among 5426 initial matches)
- Per-cluster npeaks difference between selected and discarded matches
- Proportion of multiple-match clusters with a positive npeaks difference
- Rows in the per-dataset summary metrics table (CSV/TSV) covering this validation strategy

## How to apply

Load the final match tables produced after M2S steps 1–3 (initial threshold matching, unique-correspondence resolution, and optional tightened-threshold poor-match removal) for both the reference dataset (e.g., MESA, Dataset 1: RP UPLC MS ESI+) and the target dataset (e.g., Rotterdam). Count how many matches are unique and how many fall into multiple-match clusters; report the ratio of unique matches to multiple-match clusters (Dataset 1: 61 clusters among 5426 initial matches). For each multiple-match cluster, pull the npeaks value (number of samples across the cohort in which each feature was detected) from the XCMS feature table and compute the npeaks difference between the selected match and each discarded candidate; report the proportion of clusters in which this difference is positive, indicating the retained match is the more frequently detected feature. Assemble these per-cluster statistics with the other five validation metrics (annotation agreement, log10FI correlation, covariate-association agreement, sample-detection counts, common highly correlated features) into a summary metrics table per dataset.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox that produces the match tables and unique/multiple-match cluster structure being evaluated by this npeaks-based validation strategy) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment in which the M2S toolbox runs and in which the npeaks cluster statistics are computed and tabulated) — https://github.com/rjdossan/M2S
- **XCMS** (Peak-picking tool that generated the feature tables whose per-feature sample-detection counts (npeaks) feed the cluster comparison)

## Evaluation signals

- Summary table contains the unique-match vs multiple-match-cluster ratio for each dataset, matching the reported scale (Dataset 1: 61 clusters among 5426 initial matches)
- Every multiple-match cluster has a computed npeaks difference between the selected and each discarded candidate match, with sign preserved
- Reported proportion of positive npeaks differences lies in [0, 1] and is consistent with the annotation-agreement rates reported for the same tables (Dataset 1: 585/604 = 96.8% correct; Dataset 2: 79/87 = 90.8% correct)
- All six validation metric categories are present in the summary table, with this npeaks/cluster metric as one of them
- Cluster counts derived from step 2 unique-correspondence output reconcile with the count of initial matches entering step 3

## Limitations

- The optional step 3 (tightened thresholds for poor-match removal) may inadvertently delete some correct matches, altering the cluster composition before npeaks scoring
- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- This metric only characterizes ambiguous clusters; features matched uniquely are not scored by the npeaks difference and must be validated by the other orthogonal strategies (annotation agreement, FI correlation, covariate-association agreement, sample-detection counts, common correlated features)

## Evidence

- [abstract] Six orthogonal validation strategies including unique/multiple-match cluster ratio and npeaks: "using six orthogonal validation strategies to gauge the matching quality"
- [methods] Unique-correspondence step that forms the clusters being evaluated: "Step 2: Find
Unique Correspondence"
- [methods] Optional poor-match removal that changes cluster membership before scoring: "Step
3: Find Poor Matches (Tighten Thresholds)"
- [abstract] Anchor numbers for the unique-match validation on Dataset 1: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [readme] M2S toolbox environment for computing the cluster statistics: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
