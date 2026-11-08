---
name: in-silico-annotation-cutoff-filtering
description: Use when cleaning GNPS molecular networking library annotations or SIRIUS/ISDB
  in-silico annotation results before computing annotation statistics or novelty ranks
  for an untargeted MS/MS extract set.
when_to_use_negative:
- Input is already a cleaned, filtered annotation table (re-running cutoffs could
  double-filter and under-count annotations)
- You only have raw MZmine quantitative data with no annotation source (GNPS library
  matches, SIRIUS/ISDB or timaR output) — there is nothing to filter
- You need to preserve all tentative annotations for exploratory spectral review rather
  than for computing a novelty or annotation-rate score
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Source of the library spectral match results that are cleaned with ppm_error,
    shared_peaks and cosine thresholds
- name: Inventa
  role: Hosts the annotation cleanup parameters (ppm_error, shared_peaks, cosine,
    ionisation mode) and uses the filtered annotations for the Feature Component score
  repo: github.com/luigiquiros/inventa
- name: SIRIUS
  role: Produces in-silico annotations (with Zodiac and Confidence scores) that are
    cut off at min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0
- name: timaR
  role: Provides taxonomically reponderated in-silico annotation results consumed
    as tima_results_filename
  repo: https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- name: gnps_postprocessing
  role: Repository of Python scripts to postprocess/clean GNPS annotations, used as
    a submodule of Inventa
  repo: github.com/lfnothias/gnps_postprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:c604a288b7917810654364b9f8a3c0ec78c97a8ba19b56b5d4c4461eaa78ed65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  - SIRIUS
  - gnps_postprocessing
  - timaR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# In-silico annotation cutoff filtering

## Summary

Filters library and in-silico spectral annotations (GNPS library matches, SIRIUS/ISDB, timaR) using explicit quality thresholds so that downstream novelty scoring in Inventa relies only on high-confidence annotations.

## When to use

Use when cleaning GNPS molecular networking library annotations or SIRIUS/ISDB in-silico annotation results before computing annotation statistics or novelty ranks for an untargeted MS/MS extract set.

## When NOT to use

- Input is already a cleaned, filtered annotation table (re-running cutoffs could double-filter and under-count annotations)
- You only have raw MZmine quantitative data with no annotation source (GNPS library matches, SIRIUS/ISDB or timaR output) — there is nothing to filter
- You need to preserve all tentative annotations for exploratory spectral review rather than for computing a novelty or annotation-rate score

## Inputs

- GNPS library spectral match / annotation table (from a feature-based molecular networking job)
- SIRIUS/ISDB in-silico annotation result files (project space)
- timaR reponderated annotation output (tima_results_filename)
- Canopus/Sirius NPC summary file (canopus_npc_summary_filename) for chemical class annotation
- Ionisation mode specification (e.g. 'pos')

## Outputs

- Cleaned annotation table with only threshold-passing library annotations
- Valid in-silico annotations passing Zodiac/Confidence score cut-offs
- Reduced node/feature counts (annotated vs non-annotated) feeding the Feature Component (FC) and combined annotation-rate statistics

## How to apply

Obtain the GNPS spectral match results table and run a cleanup script (from gnps_postprocessing, used as a submodule of Inventa) with thresholds defined in the Inventa parameters: ppm_error = 5 (minimum error in ppm to consider an annotation valid), shared_peaks = 10, and cosine = 0.7, restricted to the desired ionisation mode (e.g. 'pos'). For in-silico SIRIUS/ISDB results, apply cut-offs of min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0; timaR-weighted annotation files can additionally be passed via tima_results_filename. The rationale is to discard weak library matches and low-confidence in-silico predictions so that the Feature Component (FC) counts 'non-annotated' features against a trustworthy annotated baseline. In the Celastraceae proof of concept, this reduced 2,494 GNPS library-annotated nodes to 1,751 valid annotations. Verify by comparing node counts before and after filtering against reference values and confirming the filtered table retains the expected columns (row m/z, row retention time, Peak area).

## Related tools

- **GNPS** (Source of the library spectral match results that are cleaned with ppm_error, shared_peaks and cosine thresholds)
- **Inventa** (Hosts the annotation cleanup parameters (ppm_error, shared_peaks, cosine, ionisation mode) and uses the filtered annotations for the Feature Component score) — github.com/luigiquiros/inventa
- **SIRIUS** (Produces in-silico annotations (with Zodiac and Confidence scores) that are cut off at min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0)
- **timaR** (Provides taxonomically reponderated in-silico annotation results consumed as tima_results_filename) — https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- **gnps_postprocessing** (Repository of Python scripts to postprocess/clean GNPS annotations, used as a submodule of Inventa) — github.com/lfnothias/gnps_postprocessing

## Examples

```
git submodule update --remote  # pull gnps_postprocessing scripts, then run the cleanup with ppm_error=5, shared_peaks=10, cosine=0.7 on the GNPS spectral match table
```

## Evaluation signals

- Count of library-annotated nodes before cleaning (e.g. 2,494 for the Celastraceae set) matches the raw GNPS output
- Count of valid annotations after filtering (e.g. 1,751) is reproducible when re-running with ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos'
- Filtered table retains only annotations meeting or exceeding each threshold (no cosine < 0.7 or shared_peaks < 10 rows remain)
- SIRIUS/ISDB-filtered set contains no entries with Zodiac score < 0.9
- Combined annotation rate computed from the cleaned set is consistent with the reported overall rate (~68%) for the Celastraceae feature set

## Limitations

- Threshold values (cosine 0.7, shared_peaks 10, ppm 5, Zodiac 0.9) are the defaults used for the Celastraceae proof of concept; other datasets may require retuning rather than blind reuse
- Cleaning is restricted to a single ionisation mode at a time (e.g. 'pos'); negative-mode data must be processed separately
- CANOPUS/SIRIUS outputs require an extra recompute step (via canopus_treemap's Canopus class) before their chemical class annotations can be filtered and used, adding a pipeline dependency

## Evidence

- [methods] Annotation cleanup thresholds in Inventa: "ppm_error = 5                     # min error in ppm to consider an annotation valable"
- [methods] Inventa cleaning-up annotations from GNPS: "For cleaning-up annotations from GNPS"
- [methods] timaR reponderated annotation input: "`tima_results_filename`: [timaR](https://taxonomicallyinformedannotation.github.io/tima-r/index.html) reponderated output format."
- [readme] GNPS postprocessing scripts used to clean annotations: [evidence span withheld — anchor status: external_doc]
- [readme] GNPS annotation format expectation: "The standard format from GNPS is prefered:"
- [results] Celastraceae annotation counts before and after cleaning: [evidence span withheld — anchor status: not_found]
- [methods] SIRIUS/ISDB cut-offs for in-silico annotation: "Optionally, the following inputs can be added:

- Annotations files (ISDB and/or SIRIUS)"
- [readme] Feature Component definition relies on annotation status: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
