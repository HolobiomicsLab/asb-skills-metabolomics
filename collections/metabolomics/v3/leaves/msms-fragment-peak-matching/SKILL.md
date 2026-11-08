---
name: msms-fragment-peak-matching
description: Use when annotating an LC-MS/MS peak (e.g. negative-mode peak M386T77,
  m/z 385.1118, RT 1.27 min) by matching its fragment peaks to a candidate metabolite's
  reference fragments within a 10 ppm tolerance to obtain an MS/MS score.
when_to_use_negative:
- The input is already a quantified feature table or downstream statistical result
  — this skill only scores spectral annotation evidence for raw peaks
- No reference fragment spectra or candidate annotation database is available for
  the metabolite, so fragment matching cannot be performed
- The analysis is not plant metabolomics and requires a non-plant metabolite reference
  space beyond what PlantMetSuite's plant-specific database covers
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PlantMetSuite
  role: Web-based tool whose annotation scoring module performs the fragment peak
    matching (10 ppm tolerance), RT scoring, and integrated score computation (Equation
    (3))
  repo: github.com__YaonanLab__PlantMetSuite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:9d53372fac716c5aa47d0a2ffba997ed400d0f86c14ea376d0116435e636d965
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# MS/MS fragment peak matching

## Summary

Matches acquired MS/MS fragment peaks against a candidate metabolite's reference fragments to compute a fragment-match score that feeds PlantMetSuite's integrated metabolite annotation scoring (Equation (3)). Rationale: reproducible fragment-level matching with explicit ppm tolerances is the core evidence component for assigning a metabolite identity to an LC-MS/MS peak.

## When to use

Use when annotating an LC-MS/MS peak (e.g. negative-mode peak M386T77, m/z 385.1118, RT 1.27 min) by matching its fragment peaks to a candidate metabolite's reference fragments within a 10 ppm tolerance to obtain an MS/MS score.

## When NOT to use

- The input is already a quantified feature table or downstream statistical result — this skill only scores spectral annotation evidence for raw peaks
- No reference fragment spectra or candidate annotation database is available for the metabolite, so fragment matching cannot be performed
- The analysis is not plant metabolomics and requires a non-plant metabolite reference space beyond what PlantMetSuite's plant-specific database covers

## Inputs

- MS/MS spectrum of the query peak with fragment peak m/z and intensity values
- Peak metadata: precursor m/z (e.g. 385.1118), retention time (e.g. 1.27 min), ionization mode (negative)
- Candidate metabolite reference fragment list (retrieved via MS1 precursor cutoff of 25 ppm)
- User-defined component score weights for the integrated score formula

## Outputs

- MS/MS fragment-match score for each candidate (e.g. 0.9606 for 1-O-beta-D-glucopyranosyl sinapate)
- Retention-time score (e.g. 1 for the sinapate example)
- Integrated final annotation score per candidate (e.g. 0.9688)
- CSV of annotation scores, e.g. annotation_scores_sinapate.csv

## How to apply

Load the peak's MS/MS spectrum and metadata (precursor m/z, retention time, ionization mode) — e.g. the sinapate example at m/z 385.1118, RT 1.27 min, negative mode. First restrict candidates with an MS1 precursor mass cutoff of 25 ppm so only plausible molecular formulas are matched. Then match each observed fragment peak against the candidate's reference fragments using a 10 ppm fragment tolerance to compute the MS/MS score (0.9606 for the sinapate example). Separately compute the retention-time score from the peak's RT using the tool's RT scoring scheme (score of 1 for the sinapate example). Finally combine the MS/MS score, RT score, and additional criteria with user-defined weights per the integrated score formula (Equation (3)) — the sinapate example yields 0.9688 — and write the per-candidate scores to a CSV such as annotation_scores_sinapate.csv. Validate by checking the reproduced MS/MS, RT, and integrated scores against the values reported in Results 2.2 of the paper.

## Related tools

- **PlantMetSuite** (Web-based tool whose annotation scoring module performs the fragment peak matching (10 ppm tolerance), RT scoring, and integrated score computation (Equation (3))) — github.com__YaonanLab__PlantMetSuite

## Evaluation signals

- Reproduced MS/MS fragment-match score equals the reported 0.9606 for 1-O-beta-D-glucopyranosyl sinapate
- Retention-time score equals 1 for the sinapate example (RT 1.27 min)
- Integrated final score equals the reported 0.9688 when component scores are combined with the same user-defined weights per Equation (3)
- Candidate list retrieved with the 25 ppm MS1 precursor cutoff contains the true annotation (m/z 385.1118)
- Output CSV (annotation_scores_sinapate.csv) contains per-component scores and the integrated score for each candidate

## Limitations

- Scores depend on user-defined weights in the integrated formula, so different weight choices change the final integrated score
- No changelog found for the repository, so tool behavior across versions may not be traceable
- The tool is browser-accessible and plant-focused; non-plant metabolites may lack reference fragments in the plant-specific database

## Evidence

- [methods] fragment tolerance rationale: [evidence span withheld — anchor status: not_found]
- [results] integrated score validation: [evidence span withheld — anchor status: not_found]
- [abstract] tool scope: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis"
- [abstract] tool accessibility: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
