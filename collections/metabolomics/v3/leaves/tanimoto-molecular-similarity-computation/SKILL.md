---
name: tanimoto-molecular-similarity-computation
description: Use when a task requires quantifying structural relatedness between small
  molecules (e.g., ECFP fingerprint pairs) to group validation-set molecules by whether
  they have a training-set counterpart above a Tanimoto threshold such as 90% or 95%.
when_to_use_negative:
- When the task concerns the DLM architecture or training itself (Keras layers, L2
  regularization 0.0001, adam lr 0.01) rather than similarity-based grouping of molecules.
- When molecules lack computable ECFP fingerprints or the dataset is not the METLIN
  SMRT/METLIN library RT dataset, since the thresholds reported (e.g., n=5607 at 90%)
  are specific to that dataset split.
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Computes ECFP fingerprints for each molecule and the Tanimoto similarity between
    validation- and training-set molecules used for threshold-based grouping
- name: pandas
  role: Manages the per-molecule similarity groupings and threshold assignments written
    to similarity_threshold_statistical_tests.csv
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:8575373183f654ccea2f4cfd83d20ba0b62a13b16b5b5517abec952c9febeab3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - SciPy
  - Tanimoto similarity
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# tanimoto-molecular-similarity-computation

## Summary

Computes Tanimoto similarity on ECFP fingerprints between validation and training molecules from the METLIN SMRT dataset to stratify molecules by structural similarity at 95/90/80/70/50% thresholds, enabling k-NN baselines and error-stratified significance testing for RT prediction.

## When to use

Use when a task requires quantifying structural relatedness between small molecules (e.g., ECFP fingerprint pairs) to group validation-set molecules by whether they have a training-set counterpart above a Tanimoto threshold such as 90% or 95%.

## When NOT to use

- When the task concerns the DLM architecture or training itself (Keras layers, L2 regularization 0.0001, adam lr 0.01) rather than similarity-based grouping of molecules.
- When molecules lack computable ECFP fingerprints or the dataset is not the METLIN SMRT/METLIN library RT dataset, since the thresholds reported (e.g., n=5607 at 90%) are specific to that dataset split.

## Inputs

- METLIN SMRT dataset records (structures + RT in seconds) with ECFP fingerprints computed per molecule
- 75% training / 25% validation split of the 80,038-molecule METLIN library
- Similarity threshold set (95, 90, 80, 70, 50%)

## Outputs

- Per-molecule Tanimoto similarity scores between each validation molecule and all training molecules
- Threshold-specific grouping of validation molecules into 'with similar counterpart' vs 'without counterpart' sets
- k=3 nearest-neighbor candidate sets with median experimental RT for the naive baseline

## How to apply

Represent each molecule as an ECFP fingerprint using RDKit, then compute the Tanimoto similarity between each validation-set molecule and all training-set molecules from the METLIN SMRT dataset (RT in seconds plus ECFP fingerprints). Assign each validation molecule to a 'with similar counterpart' or 'without counterpart' group at each threshold of 95, 90, 80, 70 and 50%. For the k-NN comparison, form threshold-specific groups of validation molecules having at least one training-set counterpart above the threshold, and for the k=3 baseline retrieve the 3 most similar training molecules and take the median of their experimental RT. The 95% threshold behaves differently from the others in this paper — significance at 95% was not observed — so thresholds must be evaluated individually rather than pooled. The resulting grouping feeds downstream paired Wilcoxon rank tests on relative RT errors.

## Related tools

- **RDKit** (Computes ECFP fingerprints for each molecule and the Tanimoto similarity between validation- and training-set molecules used for threshold-based grouping)
- **pandas** (Manages the per-molecule similarity groupings and threshold assignments written to similarity_threshold_statistical_tests.csv)

## Examples

```
from rdkit import DataStructs; sim = DataStructs.TanimotoSimilarity(fp_validation, fp_train); group = sim >= 0.90
```

## Evaluation signals

- Reproduced grouping must yield the reported effective sample sizes at each threshold: n = 5607 (90%), 5425 (80%), 2718 (70%), 177 (50%).
- Downstream Wilcoxon pattern must match: significant error reduction at 90/80/70/50% thresholds and no significant reduction at the 95% threshold.
- Every validation molecule must be assigned to a counterpart group at each of the five thresholds; group sizes must sum to the validation set (25% of 80,038 molecules).
- k=3 baseline per molecule must use the median (not mean) of the 3 most similar training molecules' experimental RT.

## Limitations

- At the 95% Tanimoto threshold the expected significance pattern breaks: neither the DLM-vs-kNN comparison nor the with/without-counterpart comparison showed significance, so similarity grouping at 95% does not discriminate error.
- The similarity computation requires ECFP fingerprints; molecules that cannot be fingerprinted consistently with the SMRT dataset representation cannot be grouped.
- The 50% threshold group is very small (n=177), so statistical power is limited at low similarity thresholds.

## Evidence

- [abstract] an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [methods] Compute Tanimoto similarity between each validation molecule and all training molecules, and for each similarity threshold (95, 90, 80, 70, 50%): [evidence span withheld — anchor status: not_found]
- [methods] Compute Tanimoto similarity on ECFP fingerprints between each validation-set molecule and all training-set molecules: [evidence span withheld — anchor status: not_found]
- [results] When a validation-set molecule had at least one similar molecule (≥90% similarity) in the training set, its prediction error was statistically significantly smaller: [evidence span withheld — anchor status: not_found]
