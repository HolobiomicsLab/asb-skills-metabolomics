---
name: metabolite-annotation-scoring
description: 'Use when a plant metabolomics LC-MS/MS peak needs a reproducible annotation
  score: e.g. a peak defined by m/z (e.g. 385.1118 in negative mode) plus RT (e.g.
  1.27 min) whose MS/MS spectrum should be scored against a plant-specific metabolite
  database.'
when_to_use_negative:
- The input is a feature/intensity table or statistical analysis task rather than
  an MS/MS spectrum needing annotation scoring.
- No MS/MS fragment data are available, so only an MS1 precursor match at the 25 ppm
  cutoff can be computed and the integrated score cannot be reproduced.
- The metabolite of interest is unlikely to be in the plant-specific metabolite database
  (e.g. non-plant or uncharacterized compounds).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: PlantMetSuite
  role: Web-based tool whose annotation scoring module computes MS1 (25 ppm), MS/MS
    (10 ppm fragment tolerance), and RT component scores and combines them into the
    integrated final score via Equation (3).
  repo: github.com__YaonanLab__PlantMetSuite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:4504097e1fe82ebf5625216e3c93710e1540c6ec0d49c203f4c2e7b14ad91b6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  - TurboPutative
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolite-annotation-scoring

## Summary

Reproduce metabolite annotation confidence scores with PlantMetSuite by combining MS1 precursor mass match, MS/MS fragment match (10 ppm tolerance), and retention-time scores into a single weighted integrated score (Equation (3)), as validated on the 1-O-beta-D-glucopyranosyl sinapate example (final score 0.9688).

## When to use

Use when a plant metabolomics LC-MS/MS peak needs a reproducible annotation score: e.g. a peak defined by m/z (e.g. 385.1118 in negative mode) plus RT (e.g. 1.27 min) whose MS/MS spectrum should be scored against a plant-specific metabolite database.

## When NOT to use

- The input is a feature/intensity table or statistical analysis task rather than an MS/MS spectrum needing annotation scoring.
- No MS/MS fragment data are available, so only an MS1 precursor match at the 25 ppm cutoff can be computed and the integrated score cannot be reproduced.
- The metabolite of interest is unlikely to be in the plant-specific metabolite database (e.g. non-plant or uncharacterized compounds).

## Inputs

- MS/MS peak list for the query metabolite (e.g. sinapate example spectrum, negative mode)
- Precursor m/z and retention time metadata (e.g. m/z 385.1118, RT 1.27 min)
- PlantMetSuite plant-specific metabolite database candidates
- User-defined component score weights for the integrated score formula

## Outputs

- annotation_scores CSV (e.g. annotation_scores_sinapate.csv) with per-candidate MS/MS score, RT score, and integrated final score
- Integrated annotation score per candidate metabolite (e.g. 0.9688 for 1-O-beta-D-glucopyranosyl sinapate)

## How to apply

Load the peak's MS/MS spectrum and metadata from the example data in the YaonanLab/PlantMetSuite repository. Run the PlantMetSuite annotation scoring module with an MS1 precursor mass cutoff of 25 ppm to retrieve candidate annotations matching the precursor m/z. Match fragment peaks against each candidate's reference fragments at a 10 ppm fragment tolerance to obtain the MS/MS score, and compute the retention-time score from the peak's RT using the tool's RT scoring scheme. Combine the MS/MS score, RT score, and additional criteria with user-defined weights via the integrated score formula, Equation (3). Validate the output against known reported values, e.g. MS/MS score 0.9606, RT score 1, and integrated score 0.9688 for 1-O-beta-D-glucopyranosyl sinapate, written to annotation_scores_sinapate.csv.

## Related tools

- **PlantMetSuite** (Web-based tool whose annotation scoring module computes MS1 (25 ppm), MS/MS (10 ppm fragment tolerance), and RT component scores and combines them into the integrated final score via Equation (3).) — github.com__YaonanLab__PlantMetSuite

## Evaluation signals

- Reproduced MS/MS score for the sinapate example equals 0.9606.
- Reproduced RT score for the 1.27 min retention time equals 1.
- Integrated final score computed via Equation (3) equals 0.9688 as reported in Results 2.2 of the paper.
- Candidate retrieval is performed with the stated MS1 precursor mass cutoff of 25 ppm on m/z 385.1118.
- Fragment matching uses the 10 ppm fragment tolerance, and results are written to annotation_scores_sinapate.csv.

## Limitations

- Validation shown in the source card covers a single example metabolite (1-O-beta-D-glucopyranosyl sinapate); reproducibility for other peaks is not demonstrated in the provided context.
- The integrated score depends on user-defined weights, so scores are not comparable across different weight configurations.
- The repository shows no changelog found (missing signal in the discussion section), so version drift of scoring behavior cannot be tracked from context.

## Evidence

- [results] Integrated score reproduction validated against reported values: [evidence span withheld — anchor status: not_found]
- [methods] Workflow steps with thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Fragment tolerance parameter: [evidence span withheld — anchor status: not_found]
- [abstract] Tool scope from abstract: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics"
