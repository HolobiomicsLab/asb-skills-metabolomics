---
name: feature-list-consistency-evaluation
description: Use when MS1 feature lists from multiple LC-IM-MS replicate runs are
  available and you need to quantify whether preprocessing (e.g., PreProcessor smoothing)
  changed replicate-to-replicate detection consistency.
when_to_use_negative:
- Raw spectra have not been feature-found yet — this skill compares existing replicate
  feature lists, not MS-files (run a feature finder such as Lipid Annotator first).
- Only one replicate is available — the all-3-replicate intersection and consistency
  metric are undefined.
- The question is lipid annotation gain or mass accuracy (19.4% annotation increase;
  10.6 to 1.9 ppm after saturation repair) rather than replicate detection consistency.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3172
tools:
- name: PNNL PreProcessor
  role: Produced the preprocessed replicate MS-files (demultiplexing, multidimensional
    smoothing with 3 points in LC and 7 in IM, intensity threshold of 50 counts, 1
    adjacent point spike removal) whose downstream feature lists are compared against
    the original ones
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: Lipid Annotator
  role: Downstream feature finding and annotation of lipids used to generate the per-replicate
    lipid feature lists evaluated for consistency
- name: pandas
  role: Loads each replicate's feature list and intersects replicate-specific feature
    sets to compute consistent-detection counts and the relative percentage
- name: R
  role: Alternative scripting environment listed by the task for the replicate set-intersection
    and percentage computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:ed7779903805210a246baf34a04297c049f30be7a6debc1f09462f7d109cb635
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Lipid Annotator
  - PNNL PreProcessor
  - R
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-list-consistency-evaluation

## Summary

Quantifies whether PNNL PreProcessor preprocessing improves replicate-to-replicate consistency of LC-IM-MS MS1 feature detection by intersecting per-replicate lipid feature lists and computing the relative increase in features found across all 3 replicates. In the source study, multidimensional smoothing raised consistent detection of NIST SRM 1950 plasma lipid extract features by 70.7%.

## When to use

Use when MS1 feature lists from multiple LC-IM-MS replicate runs are available and you need to quantify whether preprocessing (e.g., PreProcessor smoothing) changed replicate-to-replicate detection consistency.

## When NOT to use

- Raw spectra have not been feature-found yet — this skill compares existing replicate feature lists, not MS-files (run a feature finder such as Lipid Annotator first).
- Only one replicate is available — the all-3-replicate intersection and consistency metric are undefined.
- The question is lipid annotation gain or mass accuracy (19.4% annotation increase; 10.6 to 1.9 ppm after saturation repair) rather than replicate detection consistency.

## Inputs

- Per-replicate original-data LC-IM-MS lipid feature lists for 3 replicates of the NIST SRM 1950 human plasma lipid extract (MassIVE MSV000087295)
- Per-replicate PreProcessor-preprocessed lipid feature lists for the same triplicate LC-DT IM-MS run (MassIVE MSV000087295)

## Outputs

- Counts of features consistently detected across all 3 replicates for each condition (original, preprocessed)
- Relative percentage increase in consistently detected features, written to replicate_feature_counts.csv

## How to apply

Download the deposited original and preprocessed lipid feature lists for the 3 LC-DT IM-MS replicates of the NIST SRM 1950 human plasma lipid extract from MassIVE (ftp://massive.ucsd.edu/MSV000087295). Load each replicate's list into pandas (or R) and key each set on the paper's feature definition — an ion with one or multiple isotopic peaks detected in the MS1 data — so isotopic peaks collapse into a single feature. For each condition (original vs preprocessed), intersect the replicate-specific feature sets to count features consistently detected across all 3 replicates; the preprocessed lists come from demultiplexing followed by smoothing (points: 3 in LC and 7 in IM), an intensity threshold of 50 counts, and 1 adjacent point spike removal. Compute the relative increase as (count_preprocessed - count_original) / count_original * 100 and write the per-condition counts and the percentage to replicate_feature_counts.csv. Accept the result when the recomputed percentage falls within ~70-71%, matching the paper's reported 70.7%, which is attributed mainly to better-defined low-abundance signals.

## Related tools

- **PNNL PreProcessor** (Produced the preprocessed replicate MS-files (demultiplexing, multidimensional smoothing with 3 points in LC and 7 in IM, intensity threshold of 50 counts, 1 adjacent point spike removal) whose downstream feature lists are compared against the original ones) — https://omics.pnl.gov/software/pnnl-preprocessor
- **Lipid Annotator** (Downstream feature finding and annotation of lipids used to generate the per-replicate lipid feature lists evaluated for consistency)
- **pandas** (Loads each replicate's feature list and intersects replicate-specific feature sets to compute consistent-detection counts and the relative percentage)
- **R** (Alternative scripting environment listed by the task for the replicate set-intersection and percentage computation)

## Examples

```
import pandas as pd; n_common = len(set.intersection(*[set(pd.read_csv(f).iloc[:, 0]) for f in rep_feature_lists]))
```

## Evaluation signals

- Recomputed relative increase in all-3-replicate consistent features falls within ~70-71%, reproducing the reported 70.7%.
- count_preprocessed > count_original in the replicate set intersections — on this dataset smoothing must increase, not reduce, consistency.
- Features are keyed per the paper's definition (ions with one or multiple isotopic peaks detected in MS1 data), with isotopic peaks collapsed before intersection.
- The intersection is computed over the same triplicate set (3 replicates of NIST SRM 1950) for both conditions so the original vs preprocessed comparison is symmetric.
- replicate_feature_counts.csv contains per-condition counts for both original and preprocessed plus the percentage, with non-negative integer counts.

## Limitations

- Consistency gains are greatest among low-abundance ions, so abundant features may show smaller improvements.
- The 70.7% benchmark is specific to triplicate LC-DT IM-MS analysis of a NIST SRM 1950 human plasma lipid extract; other matrices, replicates counts, or IM-MS instrumentation are not established.
- Consistent detection rose 70.7% but lipid annotations for features found in at least 2 replicates rose only 19.4%, so consistent detection should not be equated with annotation gain.

## Evidence

- [results] Smoothing increased 3-replicate consistent lipid feature detection by 70.7%: "the number of LC-IM-MS lipid features (ions with one or multiple isotopic peaks detected in the MS1 data) consistently detected across 3 replicates was increased by 70.7%."
- [results] Dataset: triplicate LC-DT IM-MS analysis of human plasma lipid extract: "a human plasma lipid extract was analyzed by LC-DT IM-MS in triplicates"
- [results] Preprocessing parameters used for the preprocessed condition: "Step 2 was demultiplexing using 75% pulses for artifact removal followed by smoothing (points: 3 in LC and 7 in IM), intensity threshold of 50 counts and 1 adjacent point spike removal."
- [abstract] Gains concentrate on low-abundance signals: "multidimensional smoothing improves peak shapes of poorly defined low-abundance signals"
- [intro] Preprocessing boosts reproducibility of results: "Preprocessing algorithms working at the raw data stage can boost the quality and reproducibility of results by tackling two main challenges associated with dynamic range"
- [results] Consistent detection gain exceeds annotation gain (19.4%): "The PreProcessor provided a 19.4% increase in the total number of lipid annotations for features found in at least 2 replicates."
