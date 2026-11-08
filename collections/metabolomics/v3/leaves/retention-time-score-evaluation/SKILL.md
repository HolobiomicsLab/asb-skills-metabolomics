---
name: retention-time-score-evaluation
description: Use when scoring metabolite annotations from MS/MS spectra in negative
  mode where retention time is available, e.g. verifying that a candidate like 1-O-beta-D-glucopyranosyl
  sinapate at RT 1.27 min receives the expected RT score of 1.
when_to_use_negative:
- When retention time is unavailable or unreliable for the query peak — the RT component
  cannot be scored and the integrated score reduces to other components
- When the goal is database-wide candidate retrieval rather than scoring a candidate
  annotation against a known RT (candidate retrieval is handled by the 25 ppm MS1
  cutoff step, not the RT score)
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: PlantMetSuite
  role: Provides the RT scoring scheme and the integrated score formula (Equation
    (3)) that combines the RT score with the MS/MS score and additional criteria into
    the final annotation score
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:6a67b3f0a0244605e6c5557697238d685910e526b5720ede95889a3b0206e403
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# retention-time-score-evaluation

## Summary

Computes the retention-time component score within PlantMetSuite's metabolite annotation scoring, where an RT score of 1 denotes a perfect retention-time match, and combines it with the MS/MS score and additional criteria via the integrated score formula (Equation (3)) to produce a final annotation score.

## When to use

Use when scoring metabolite annotations from MS/MS spectra in negative mode where retention time is available, e.g. verifying that a candidate like 1-O-beta-D-glucopyranosyl sinapate at RT 1.27 min receives the expected RT score of 1.

## When NOT to use

- When retention time is unavailable or unreliable for the query peak — the RT component cannot be scored and the integrated score reduces to other components
- When the goal is database-wide candidate retrieval rather than scoring a candidate annotation against a known RT (candidate retrieval is handled by the 25 ppm MS1 cutoff step, not the RT score)

## Inputs

- MS/MS spectrum and metadata for the query peak (m/z 385.1118, RT 1.27 min, negative mode) from PlantMetSuite example data
- Candidate metabolite reference fragments (used for 10 ppm fragment matching)
- User-defined weights for the component scores in the integrated score formula (Equation (3))

## Outputs

- RT component score (expected 1 for the sinapate example)
- Integrated final annotation score (expected 0.9688)
- annotation_scores_sinapate.csv containing per-component and integrated scores

## How to apply

Load the candidate metabolite's retention time (e.g. 1.27 min for the sinapate example) alongside the query peak's RT, and evaluate it with PlantMetSuite's RT scoring scheme, which yields a normalized RT score (1 indicates a perfect match). Separately, match fragment peaks against the candidate's reference fragments at a 10 ppm fragment tolerance, with candidate retrieval filtered at an MS1 precursor mass cutoff of 25 ppm around m/z 385.1118. Then combine the MS/MS score (0.9606 for the sinapate example), the RT score, and additional criteria using the user-defined weights in the integrated score formula (Equation (3)) to obtain the final score (0.9688). The decision point is whether the integrated score meets the reporting threshold; results are written to annotation_scores_sinapate.csv.

## Related tools

- **PlantMetSuite** (Provides the RT scoring scheme and the integrated score formula (Equation (3)) that combines the RT score with the MS/MS score and additional criteria into the final annotation score) — https://github.com/YaonanLab/PlantMetSuite

## Evaluation signals

- Reproduced MS/MS score equals the reported 0.9606 for the sinapate example
- Reproduced RT score equals 1 (perfect RT match at 1.27 min)
- Integrated final score computed via Equation (3) equals the reported 0.9688
- Fragment matching used the 10 ppm fragment tolerance and candidate retrieval used the 25 ppm MS1 precursor mass cutoff
- Output is written to annotation_scores_sinapate.csv with per-component and final scores

## Limitations

- Scores are validated only against the reported sinapate example; other metabolites/modes were not verified in this reproduction
- The integrated score depends on user-defined weights, so reproductions must use the same weighting scheme as the paper's Results 2.2
- No changelog found for the PlantMetSuite repository, so scoring-scheme changes between versions cannot be tracked from the repo

## Evidence

- [results] integrated score formula and final value: [evidence span withheld — anchor status: not_found]
- [methods] workflow parameters for scoring: [evidence span withheld — anchor status: not_found]
- [abstract] tool accessibility statement: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis"
- [discussion] repository maintenance caveat: "Source: github:github.com__YaonanLab__PlantMetSuite"
