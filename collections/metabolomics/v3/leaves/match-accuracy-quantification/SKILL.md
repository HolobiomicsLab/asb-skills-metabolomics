---
name: match-accuracy-quantification
description: Reference for validating feature-correspondence matching when a simulated
  (synthetic) LC–MS metabolomics dataset with known embedded ground-truth correspondences
  is available and you need a match-accuracy number.
when_to_use_negative:
- Only real datasets are available and no embedded ground-truth correspondences exist
  — use annotated-feature validation (e.g. level-2 MSI annotations) instead, which
  the paper notes is 'limited by the number of annotations'
- Evaluating matching quality between real biological cohorts (e.g. MESA vs Rotterdam)
  where accuracy cannot be computed exactly
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Runs the full three-step matching workflow (threshold matching, unique correspondence,
    poor-match deletion) whose output matches are scored against ground truth
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment for the M2S toolbox used to produce the match list being
    validated
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:f4a5391c8cceb864911c439ee09e93b6010aa030f723308f71f13382b027b9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# match-accuracy-quantification

## Summary

Quantify the fraction of M2S feature matches that are correct by comparing final matches against embedded ground-truth correspondences in a simulated LC–MS dataset, validating the correspondence workflow end-to-end.

## When to use

Reference for validating feature-correspondence matching when a simulated (synthetic) LC–MS metabolomics dataset with known embedded ground-truth correspondences is available and you need a match-accuracy number.

## When NOT to use

- Only real datasets are available and no embedded ground-truth correspondences exist — use annotated-feature validation (e.g. level-2 MSI annotations) instead, which the paper notes is 'limited by the number of annotations'
- Evaluating matching quality between real biological cohorts (e.g. MESA vs Rotterdam) where accuracy cannot be computed exactly

## Inputs

- Simulated untargeted LC–MS feature tables (reference and target) with known embedded ground-truth correspondences, e.g. Dataset 3 with 4755 features per dataset
- Ground-truth correspondence pairs embedded in the simulated data design
- Feature attributes: retention time (RT), m/z, and optionally feature intensity (FI)

## Outputs

- Match-accuracy fraction (correct matches / total detected matches), e.g. 99.6%
- Count of true (correct) matches and false matches
- Final set of M2S matches classified as correct or incorrect

## How to apply

Run the M2S correspondence workflow (Step 1 threshold matching, Step 2 unique correspondence via shift modeling of RT/MZ/FI, Step 3 poor-match deletion using nonlinear thresholds at median + 3×MAD) on both feature tables of the simulated dataset — e.g. Dataset 3 with 4755 features per dataset. Retrieve the ground-truth correspondence pairs embedded in the simulated data design. Compare the final set of M2S matches against the ground-truth pairs, counting true and false matches, then compute match accuracy as correct matches divided by total detected matches. In the paper's simulated Dataset 3 (derived from Dataset 2 with added systematic and random variability), this procedure yielded 99.6% correct matches (2682 of 2717 matches).

## Related tools

- **M2S (Match 2 Sets)** (Runs the full three-step matching workflow (threshold matching, unique correspondence, poor-match deletion) whose output matches are scored against ground truth) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment for the M2S toolbox used to produce the match list being validated) — https://github.com/rjdossan/M2S

## Examples

```
matlab -batch "run M2S on simulated Dataset 3 (4755 features each); compare final matches to embedded ground-truth pairs; report correct/total matches"
```

## Evaluation signals

- Match-accuracy fraction must equal the expected value (99.6% on simulated Dataset 3 with 4755 features/dataset)
- Number of detected matches consistent with expected count (~2717 matches, of which 2682 correct)
- Correctness verified by exact agreement of final matches with embedded ground-truth pairs
- Accuracy computed as correct matches divided by total detected matches, not by total features

## Limitations

- Simulated-data accuracy does not capture real-data failure modes, such as clusters of features at different FI (possible column bleeding) or features behaving very differently from modeled RT/MZ/FI shifts
- Performance is dependent on the simulation design (systematic and random variability added to the parent dataset)
- Ground truth only exists where correspondences were embedded; features without embedded pairs cannot be scored
- Step 3 (poor-match deletion) may inadvertently delete some correct matches, slightly biasing accuracy downward

## Evidence

- [abstract] Simulated data with 4755 features per dataset, 99.6% of matches correct: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [methods] Step 3 tightens thresholds to delete poor matches: "Step 3: Find Poor Matches (Tighten Thresholds)"
- [discussion] Validation limited by number of annotations on real data: "while being limited by the number of annotations in our datasets"
- [discussion] Step 3 may delete some correct matches: "this final/optional step may inadvertently delete some correct matches"
