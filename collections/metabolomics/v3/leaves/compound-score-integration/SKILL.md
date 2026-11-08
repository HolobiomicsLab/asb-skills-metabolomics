---
name: compound-score-integration
description: Use when a peak (precursor m/z, RT, MS/MS spectrum) must be scored against
  candidate metabolite annotations, e.g. reproducing reported annotation scores like
  the sinapate example at m/z 385.1118, RT 1.27 min in negative mode.
when_to_use_negative:
- The task is raw data processing (peak detection, alignment) rather than scoring
  already-listed candidate metabolite annotations against a peak.
- The input peak lacks MS/MS fragments or a retention time, so the component scores
  required by Equation (3) cannot be computed.
- The metabolite of interest is outside the tool's plant-specific metabolite database,
  so no candidate annotations can be retrieved at the 25 ppm precursor cutoff.
tools:
- name: PlantMetSuite
  role: Web-based tool whose annotation scoring module applies the 25 ppm MS1 cutoff,
    10 ppm fragment matching, RT scoring, and Equation (3) weighted integration
  repo: github.com__YaonanLab__PlantMetSuite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:28e59d8eee5ab5dc75eb63ff796d99b7af8118fb3667e66ac4479bdf5e258b6f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# compound-score-integration

## Summary

Computes PlantMetSuite's integrated metabolite annotation score (Equation (3)) by combining an MS/MS fragment-match score, a retention-time score, and additional criteria with user-defined weights into a single final score per candidate metabolite.

## When to use

Use when a peak (precursor m/z, RT, MS/MS spectrum) must be scored against candidate metabolite annotations, e.g. reproducing reported annotation scores like the sinapate example at m/z 385.1118, RT 1.27 min in negative mode.

## When NOT to use

- The task is raw data processing (peak detection, alignment) rather than scoring already-listed candidate metabolite annotations against a peak.
- The input peak lacks MS/MS fragments or a retention time, so the component scores required by Equation (3) cannot be computed.
- The metabolite of interest is outside the tool's plant-specific metabolite database, so no candidate annotations can be retrieved at the 25 ppm precursor cutoff.

## Inputs

- MS/MS spectrum and peak metadata (precursor m/z e.g. 385.1118, RT e.g. 1.27 min, ion mode e.g. negative) from PlantMetSuite example data
- MS1 precursor mass cutoff (25 ppm) and fragment mass tolerance (10 ppm)
- Candidate metabolite annotations with reference fragment lists from the plant-specific metabolite database
- User-defined component weights for the integrated score formula (Equation (3))

## Outputs

- Component scores per candidate: MS/MS fragment-match score (e.g. 0.9606) and RT score (e.g. 1)
- Integrated final annotation score per candidate (e.g. 0.9688)
- annotation_scores_sinapate.csv results table

## How to apply

Load the peak's MS/MS spectrum and metadata (e.g. m/z 385.1118, RT 1.27 min, negative mode) and run PlantMetSuite's annotation scoring with an MS1 precursor mass cutoff of 25 ppm to retrieve candidates. Match fragment peaks against each candidate's reference fragments using a 10 ppm fragment tolerance to compute the MS/MS score (expected 0.9606 for the 1-O-beta-D-glucopyranosyl sinapate example). Compute the retention-time score from the peak RT using the tool's RT scoring scheme (expected 1 for the sinapate example). Combine the MS/MS score, RT score, and additional criteria via the integrated score formula, Equation (3), using user-defined component weights to yield the final integrated score (expected 0.9688), and write results to annotation_scores_sinapate.csv.

## Related tools

- **PlantMetSuite** (Web-based tool whose annotation scoring module applies the 25 ppm MS1 cutoff, 10 ppm fragment matching, RT scoring, and Equation (3) weighted integration) — github.com__YaonanLab__PlantMetSuite

## Evaluation signals

- Reproduced MS/MS fragment-match score equals 0.9606 for the sinapate example
- Reproduced retention-time score equals 1 for the sinapate example
- Integrated final score via Equation (3) equals 0.9688 as reported in Results 2.2 of the paper
- All retrieved candidate annotations fall within the 25 ppm MS1 precursor mass cutoff of m/z 385.1118
- Fragment matches used for the MS/MS score all fall within the 10 ppm fragment tolerance and results are written to annotation_scores_sinapate.csv

## Limitations

- The integrated score depends on user-defined weights for the component scores, so results are sensitive to the chosen weighting scheme.
- Scoring requires candidate annotations from the plant-specific metabolite database; compounds absent from that database cannot be scored.
- No changelog was found for the repository, so tool version drift is not traceable.

## Evidence

- [abstract] abstract: "we developed PlantMetSuite, a web-based tool for comprehensive metabolomics analysis and visualisation"
- [results] results: [evidence span withheld — anchor status: not_found]
- [methods] methods: [evidence span withheld — anchor status: not_found]
