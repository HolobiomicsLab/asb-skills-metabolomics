---
name: precursor-mass-tolerance-filtering
description: Use when annotating LC–MS/MS metabolite peaks in plant metabolomics data
  and you need candidate metabolites filtered by MS1 precursor m/z tolerance (e.g.,
  25 ppm) prior to MS/MS and RT scoring.
when_to_use_negative:
- Input is a feature abundance table or differential-abundance analysis without precursor
  m/z and MS/MS fragment peaks — precursor tolerance filtering requires spectral metadata
- Target metabolites are not plant-specific and require a general metabolite reference
  database rather than PlantMetSuite's plant-tailored database
- The goal is downstream multi-omics or visualisation work in PlantMetSuite rather
  than metabolite annotation scoring
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0091
tools:
- name: PlantMetSuite
  role: Web-based tool whose annotation scoring module applies the MS1 precursor mass
    cutoff, matches fragments at 10 ppm tolerance, computes RT and integrated scores
    (Equation (3)), and writes annotation_scores_sinapate.csv
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:48c0c0802c69fed96b6720e0c8bd919d4516f1eb90cb372e4d69744d4da31948
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# precursor-mass-tolerance-filtering

## Summary

A metabolite-annotation step in PlantMetSuite that retrieves candidate metabolites within a user-defined MS1 precursor mass cutoff (25 ppm in the validated sinapate example) before scoring them by MS/MS fragment matching (10 ppm tolerance), retention time, and additional criteria combined into an integrated score via Equation (3).

## When to use

Use when annotating LC–MS/MS metabolite peaks in plant metabolomics data and you need candidate metabolites filtered by MS1 precursor m/z tolerance (e.g., 25 ppm) prior to MS/MS and RT scoring.

## When NOT to use

- Input is a feature abundance table or differential-abundance analysis without precursor m/z and MS/MS fragment peaks — precursor tolerance filtering requires spectral metadata
- Target metabolites are not plant-specific and require a general metabolite reference database rather than PlantMetSuite's plant-tailored database
- The goal is downstream multi-omics or visualisation work in PlantMetSuite rather than metabolite annotation scoring

## Inputs

- Peak MS/MS spectrum plus metadata (precursor m/z, retention time, ion mode) — e.g., m/z 385.1118, RT 1.27 min, negative mode for peak M386T77 (1-O-beta-D-glucopyranosyl sinapate) from the PlantMetSuite example data
- MS1 precursor mass cutoff parameter (25 ppm in the validated example)
- MS/MS fragment matching tolerance parameter (10 ppm in the validated example)
- User-defined component weights for the integrated score formula (Equation (3))

## Outputs

- Candidate metabolite annotations matching the precursor m/z within the 25 ppm MS1 cutoff
- Per-candidate MS/MS match score (0.9606 for the sinapate reference)
- Retention-time score (1 for the sinapate reference)
- Integrated final annotation score (0.9688 for the sinapate reference)
- Results table written to annotation_scores_sinapate.csv

## How to apply

Load the peak's MS/MS spectrum and metadata — for the reference case, peak M386T77 (1-O-beta-D-glucopyranosyl sinapate) at m/z 385.1118, RT 1.27 min in negative mode from the PlantMetSuite example data in the YaonanLab/PlantMetSuite repository. Run the PlantMetSuite annotation scoring module with an MS1 precursor mass cutoff of 25 ppm to retrieve candidate annotations matching the precursor m/z. For each candidate, match observed fragment peaks against the candidate's reference fragments using a 10 ppm fragment tolerance to compute the MS/MS score, and compute the retention-time score from the observed RT (1.27 min yields an RT score of 1). Combine the MS/MS score (expected 0.9606 for sinapate), RT score, and additional criteria using the integrated score formula of Equation (3) with user-defined weights, giving 0.9688 for the sinapate example. The precursor mass cutoff is the primary filter step that controls the candidate list size; tightening it below 25 ppm risks losing the true annotation, loosening it inflates the candidate set that downstream fragment matching must discriminate.

## Related tools

- **PlantMetSuite** (Web-based tool whose annotation scoring module applies the MS1 precursor mass cutoff, matches fragments at 10 ppm tolerance, computes RT and integrated scores (Equation (3)), and writes annotation_scores_sinapate.csv) — https://github.com/YaonanLab/PlantMetSuite

## Evaluation signals

- Reproduced MS/MS score equals 0.9606 for peak M386T77 (1-O-beta-D-glucopyranosyl sinapate)
- Reproduced RT score equals 1 for the 1.27 min retention time
- Integrated final score computed via Equation (3) equals 0.9688, matching the value reported in Results 2.2 of the paper
- Candidate list contains annotations matching m/z 385.1118 within the 25 ppm MS1 precursor cutoff
- Fragment matches are computed with the 10 ppm fragment tolerance, and results are exported to annotation_scores_sinapate.csv

## Limitations

- No changelog found for the YaonanLab/PlantMetSuite repository, so version drift in the scoring module cannot be tracked from repo metadata
- PlantMetSuite is tailored to plant metabolomics; its plant-specific database may not cover non-plant metabolites
- The skill's validated thresholds (25 ppm precursor cutoff, 10 ppm fragment tolerance) were demonstrated only on the sinapate example; other datasets may require retuning

## Evidence

- [abstract] PlantMetSuite is a web-based plant metabolomics tool with interactive bioinformatics tools and databases: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data"
- [abstract] Browser access without installation or programming: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
- [methods] Precursor cutoff and fragment tolerance in the sinapate workflow: [evidence span withheld — anchor status: not_found]
- [results] Integrated score reproduction for the sinapate example: [evidence span withheld — anchor status: not_found]
