---
name: lc-ms-simulated-data-metrics
description: Use when evaluating a feature-matching workflow on synthetic LC-MS metabolomics
  data where the true correspondences between reference and target feature tables
  are known by construction, and a match-correctness fraction must be reported.
when_to_use_negative:
- The input is real experimental data with no embedded ground-truth correspondences
  — use validation by manually annotated features instead, which is the paper's approach
  for Datasets 1 and 2
- You need to compare matching results against another published tool such as metabCombiner
  — this skill computes accuracy against known ground truth, not method-vs-method
  comparison
- Only feature-picking or QC output quality is in question (e.g. QC dilution series
  R² < 0.7 filtering) — that precedes matching and is not what this accuracy metric
  measures
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: The matching algorithm whose detected matches are scored against the embedded
    ground truth to compute the correctness fraction on simulated data
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment in which the M2S matching workflow is run and the accuracy
    metric is computed
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:a7c0e901266522bb6e825101e82aa52fecf705c335f2423c11f149e5529376b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lc-ms-simulated-data-metrics

## Summary

Computes match-accuracy metrics for LC-MS feature correspondence by comparing detected matches against ground-truth correspondences embedded in simulated feature tables, as demonstrated on the paper's Dataset 3 with 4755 features per dataset where 99.6% of M2S matches were correct.

## When to use

Use when evaluating a feature-matching workflow on synthetic LC-MS metabolomics data where the true correspondences between reference and target feature tables are known by construction, and a match-correctness fraction must be reported.

## When NOT to use

- The input is real experimental data with no embedded ground-truth correspondences — use validation by manually annotated features instead, which is the paper's approach for Datasets 1 and 2
- You need to compare matching results against another published tool such as metabCombiner — this skill computes accuracy against known ground truth, not method-vs-method comparison
- Only feature-picking or QC output quality is in question (e.g. QC dilution series R² < 0.7 filtering) — that precedes matching and is not what this accuracy metric measures

## Inputs

- Simulated LC-MS feature table for the reference dataset (e.g. MESA-derived, 4755 features) with columns for RT, m/z, and FI
- Simulated LC-MS feature table for the target dataset (e.g. Rotterdam-derived, 4755 features) with columns for RT, m/z, and FI
- Ground-truth correspondence pairs embedded in the simulated data design (mapping of reference feature index to target feature index)

## Outputs

- Match-correctness fraction (e.g. 99.6% correct matches on Dataset 3)
- Count of true matches and count of false matches among detected matches
- Total number of detected matches between the two simulated datasets

## How to apply

Construct or load a simulated dataset pair (here derived from Dataset 2 with added systematic and random variability, each containing 4755 features) in which ground-truth correspondence pairs are embedded in the data design. Run the matching workflow — for M2S this means Step 1 threshold matching, Step 2 unique correspondence via shift modeling and penalty-score selection, and optionally Step 3 poor-match deletion using nonlinear thresholds at median + 3×MAD — in MATLAB to obtain the final match set. Retrieve the embedded ground-truth pairs and compare them against the detected matches to count true and false matches. Compute the fraction of correct matches as correct matches divided by total matches; on the paper's simulated Dataset 3 this equals 99.6% (consistent with 2682 detected matches against the embedded ground truth). Unlike real-data validation via annotated features, this metric is not limited by the number of annotations and gives a direct accuracy measure of the matching algorithm.

## Related tools

- **M2S (Match 2 Sets)** (The matching algorithm whose detected matches are scored against the embedded ground truth to compute the correctness fraction on simulated data) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment in which the M2S matching workflow is run and the accuracy metric is computed) — https://github.com/rjdossan/M2S

## Evaluation signals

- The reported match-correctness fraction must equal 99.6% when reproducing the paper's simulated Dataset 3 result (2682 detected matches vs embedded ground truth)
- Every detected match must be classified as either true or false by direct comparison with the embedded correspondence table — no matches left unclassified
- The total number of detected matches should be reported alongside the accuracy (e.g. ~2717 expected from the simulation design) so precision context is preserved
- Ground-truth pairs must originate from the simulation design, not from post-hoc manual annotation of real data
- The metric denominator is the total number of detected matches, not the total number of features or annotations

## Limitations

- Requires synthetic data with by-construction ground truth; cannot be applied to real datasets where correspondences are unknown
- Accuracy reflects the simulation design (systematic and random variability added to Dataset 2); real-data noise may differ and yield lower accuracy
- Optional Step 3 poor-match deletion may inadvertently delete some correct matches, which would deflate the computed accuracy
- The simulation was limited to 4755 features per dataset; scaling to much larger feature tables has not been validated by this metric in the paper

## Evidence

- [abstract] In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [methods] Step 3: Find Poor Matches (Tighten Thresholds): "Step
3: Find Poor Matches (Tighten Thresholds)"
- [methods] Step 2: Find unique correspondence: "Step
2: Find
Unique Correspondence"
- [discussion] while being limited by the number of annotations in our datasets: "while
being limited by the number of annotations in our datasets"
