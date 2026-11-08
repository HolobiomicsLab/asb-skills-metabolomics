---
name: lipid-structural-feature-encoding
description: Use when lipid species must be turned into the specialized 26-feature
  structural vector (lipid class, fatty acid modifier, sum composition) before fitting
  or applying LiPydomics CCS/HILIC retention time prediction models.
when_to_use_negative:
- Input is already a numeric feature table or descriptors have been computed elsewhere
  — re-encoding would duplicate or conflict with the 26-feature scheme
- The task is lipid identification or statistical/plotting analysis of IM-MS data
  rather than feature generation for the prediction models
- Lipid species fall outside the 45 lipid classes (or the 23 classes with HILIC retention
  times) covered by the assembled experimental database
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: LiPydomics
  role: Supplies the lipid database in the `data` module from which lipid identities
    are drawn and encoded into the specialized feature sets
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Consumes the 26-feature encoded matrix as the regression input for CCS and
    HILIC retention time prediction models
- name: Python
  role: Implementation language in which the encoding and downstream regression are
    scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:6acc98ed6a94d63c71131e57a61201829b1948c7a3f031ed37b6ead25aa549d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-structural-feature-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes lipid species identities into a fixed 26-feature numeric representation covering lipid class, fatty acid modifier, and sum composition, as used to train LiPydomics machine-learning models for CCS and HILIC retention time prediction.

## When to use

Use when lipid species must be turned into the specialized 26-feature structural vector (lipid class, fatty acid modifier, sum composition) before fitting or applying LiPydomics CCS/HILIC retention time prediction models.

## When NOT to use

- Input is already a numeric feature table or descriptors have been computed elsewhere — re-encoding would duplicate or conflict with the 26-feature scheme
- The task is lipid identification or statistical/plotting analysis of IM-MS data rather than feature generation for the prediction models
- Lipid species fall outside the 45 lipid classes (or the 23 classes with HILIC retention times) covered by the assembled experimental database

## Inputs

- Lipid identities (lipid class, fatty acid modifier, sum composition) from the lipydomics lipid database in the `data` module
- 596-lipid HILIC retention time training subset with measured retention times in minutes

## Outputs

- Numeric feature matrix of 26 structural features per lipid species
- Train/test partition consistent with the published subsets
- Encoded features suitable for scikit-learn multivariate linear regression fitting

## How to apply

Load the lipid identities and measured values (e.g., the 596-lipid HILIC retention time training subset) from the lipid database stored in the lipydomics `data` module. Map each lipid string into the 26-feature representation, one-hot/indicator style, spanning lipid class, fatty acid modifier, and sum composition. Partition the encoded matrix into the documented training and test subsets rather than resampling freely, so metrics remain comparable to the published benchmarks. Feed the encoded features to a scikit-learn multivariate linear regression for retention time (in minutes) or CCS prediction. Validate by scoring the held-out test set and comparing MAE/MDAE/RMSE against the reported values; the same encoding must be reused, unchanged, whenever the pickled model is later applied to new lipids.

## Related tools

- **LiPydomics** (Supplies the lipid database in the `data` module from which lipid identities are drawn and encoded into the specialized feature sets) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Consumes the 26-feature encoded matrix as the regression input for CCS and HILIC retention time prediction models)
- **Python** (Implementation language in which the encoding and downstream regression are scripted)

## Evaluation signals

- Feature matrix has exactly 26 columns per lipid row covering class, fatty acid modifier, and sum composition
- Downstream test-set regression metrics land near the reported MAE/MDAE/RMSE of 0.11/0.08/0.15 min
- The same encoding reproduces train/test partitioning consistent with the documented subsets (e.g., the 596-lipid HILIC subset)
- Encoded features plus scikit-learn regression yield a model that can be pickled and re-applied to new lipid strings without re-deriving features

## Limitations

- Models were trained for only 23 lipid classes with HILIC retention times (22 for CCS), out of 45 classes in the experimental database, so encoding features for classes outside this scope have no trained model
- The feature sets are simple but specialized to the paper's lipid representation; they are not a general-purpose chemical descriptor scheme
- Coverage of the underlying experimental collections is incomplete, and ML prediction is the paper's proposed remedy for these gaps, not a guarantee of accuracy across all lipid chemical space

## Evidence
