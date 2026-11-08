---
name: mass-spectral-library-matching-cleanup
description: Use when raw GNPS FBMN spectral match annotation tables (from MZmine2/MZmine3
  feature tables of LC-HRMS/MS runs) need cleaning against defined match-quality thresholds
  before computing annotation rates or feeding downstream prioritization.
when_to_use_negative:
- The input is already a cleaned/filtered annotation set — re-applying thresholds
  will double-filter and understate annotation rates.
- The primary results are SIRIUS/ISDB in-silico annotations rather than GNPS spectral
  library matches — those require min_ZodiacScore/min_ConfidenceScore cut-offs instead
  of cosine/shared-peaks thresholds.
- Data were acquired in negative ionisation mode and the ionisation_mode parameter
  cannot be adjusted from 'pos'.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Source of the feature-based molecular networking spectral library match results
    that are cleaned
- name: Inventa
  role: Hosts the annotation-cleanup step (ppm_error, shared_peaks, cosine, ionisation_mode
    parameters) within its workflow
  repo: https://github.com/luigiquiros/inventa
- name: gnps_postprocessing
  role: Python scripts to postprocess GNPS results, used as a submodule for annotation
    cleanup
  repo: https://github.com/lfnothias/gnps_postprocessing
- name: MZmine2
  role: Produces the quantitative feature table that contextualizes which annotated
    nodes correspond to features
- name: SIRIUS
  role: Optional complementary in-silico annotations (min_ZodiacScore = 0.9, min_ConfidenceScore
    = 0.0) merged after library-match cleanup
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:b145ea6d0a83e90fde1ede6872bbff69a9a6cf1bc2f326c397c7875a51219f6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  - SIRIUS
  - gnps_postprocessing
  - timaR
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Mass spectral library matching cleanup

## Summary

Post-processes GNPS feature-based molecular networking spectral library matches by applying stringency thresholds (ppm error, shared peaks, cosine, ionisation mode) to retain only valid annotations, as done for the Celastraceae UHPLC-HRMS/MS dataset where 2,494 library-annotated nodes were reduced to 1,751 valid annotations.

## When to use

Use when raw GNPS FBMN spectral match annotation tables (from MZmine2/MZmine3 feature tables of LC-HRMS/MS runs) need cleaning against defined match-quality thresholds before computing annotation rates or feeding downstream prioritization.

## When NOT to use

- The input is already a cleaned/filtered annotation set — re-applying thresholds will double-filter and understate annotation rates.
- The primary results are SIRIUS/ISDB in-silico annotations rather than GNPS spectral library matches — those require min_ZodiacScore/min_ConfidenceScore cut-offs instead of cosine/shared-peaks thresholds.
- Data were acquired in negative ionisation mode and the ionisation_mode parameter cannot be adjusted from 'pos'.

## Inputs

- GNPS FBMN spectral match / library annotation results table
- MZmine2 or MZmine3 feature quantitative table (Peak area, row m/z, row retention time columns)
- GNPS-format metadata table
- GNPS job identifier
- Ionisation mode specification ('pos')

## Outputs

- Cleaned list of valid library annotations (nodes passing ppm_error=5, shared_peaks=10, cosine=0.7)
- Annotation statistics (annotated nodes before and after cleaning)
- Combined annotation rate summary written to a TSV file

## How to apply

Obtain the GNPS FBMN job outputs (metadata table in GNPS format, MZmine quantitative table with Peak area/row m/z/row retention time columns, and the GNPS job identifier), then run the annotation cleanup on the spectral match results with the Inventa/gnps_postprocessing thresholds: ppm_error = 5 (minimum error in ppm to consider an annotation valable), shared_peaks = 10, cosine = 0.7, and ionisation_mode = 'pos'. The rationale is that library hits failing these match-quality and mass-accuracy criteria are unreliable and inflate apparent annotation rates. The cleanup reduces raw library-annotated nodes to a 'valid annotation' set (in the reference Celastraceae job: 2,494 library-annotated nodes → 1,751 valid annotations). Optionally complement the cleaned library matches with in-silico results (SIRIUS/ISDB with min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0) to reach a combined annotation rate (ca. 68% for the Celastraceae set). Write summary statistics (feature count, node count, singletons, edges, annotated counts before/after cleaning, annotation rate) to a TSV file.

## Related tools

- **GNPS** (Source of the feature-based molecular networking spectral library match results that are cleaned)
- **Inventa** (Hosts the annotation-cleanup step (ppm_error, shared_peaks, cosine, ionisation_mode parameters) within its workflow) — https://github.com/luigiquiros/inventa
- **gnps_postprocessing** (Python scripts to postprocess GNPS results, used as a submodule for annotation cleanup) — https://github.com/lfnothias/gnps_postprocessing
- **MZmine2** (Produces the quantitative feature table that contextualizes which annotated nodes correspond to features)
- **SIRIUS** (Optional complementary in-silico annotations (min_ZodiacScore = 0.9, min_ConfidenceScore = 0.0) merged after library-match cleanup) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
ppm_error = 5; shared_peaks = 10; cosine = 0.7; ionisation_mode = 'pos'  # annotation cleanup parameters in src/inventa.py
```

## Evaluation signals

- Reproduced cleaned annotation count matches the reference: 2,494 library-annotated nodes reduced to 1,751 valid annotations for the Celastraceae GNPS FBMN job.
- Computed combined annotation rate falls near the reported ~68% across the feature set.
- Network context is consistent with the reference job: 16,139 nodes (5,922 singletons) and 22,656 edges.
- All cleaned annotations satisfy every threshold: ppm error ≤ 5, shared peaks ≥ 10, cosine ≥ 0.7, ionisation mode 'pos'.
- Summary statistics written to the output TSV include feature count, IIN group count, node count, singleton count, edge count, and annotation counts before/after cleaning.

## Limitations

- Thresholds were tuned for the Celastraceae positive-mode dataset; the ionisation_mode parameter is hard-set to 'pos' in the reference configuration and must be changed for other acquisition modes.
- Cleanup discards library matches that are legitimate but borderline (e.g., cosine between 0 and 0.7 or > 5 ppm error), so cleaned rates are conservative and job-dependent.
- Requires the deposited GNPS FBMN job input files (GNPS-format metadata, MZmine quantitative table, job identifier); it cannot run from raw vendor files alone.

## Evidence

- [methods] Annotation cleanup thresholds: "ppm_error = 5                     # min error in ppm to consider an annotation valable"
- [methods] Cleanup step: [evidence span withheld — anchor status: not_found]
- [full_text] Reference statistics from workflow reproduction: [evidence span withheld — anchor status: not_found]
- [readme] GNPS input format: "The standard format from GNPS is prefered:"
