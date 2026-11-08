---
name: isotopologue-pairing-quality-threshold-evaluation
description: Use when comparing IDSL.IPA peak lists (e.g. 20,253 peaks for MTBLS1684
  file 003.d) against MZmine 2, xcms-CAMERA or MS-DIAL output and you must attribute
  missing peaks to isotopologue pairing or peak quality thresholds.
when_to_use_negative:
- When the input is already an aligned peak intensity matrix or a feature table that
  has passed all filtering steps — this skill re-evaluates raw candidate peaks against
  thresholds, not finished data matrices.
- When the goal is molecular formula annotation or MS2-based structural identification
  (IDSL.UFA/IDSL.CSA territory) rather than explaining which peaks IDSL.IPA retained.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Supplies the 12C/13C isotopologue pairing logic and the peak quality determination
    criteria (mass correction, peak smoothing, local-noise baseline, peak quality
    determination) that baseline peaks are re-evaluated against.
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Language in which IDSL.IPA is implemented and in which the threshold-failure
    evaluation script is run.
  repo: https://cran.r-project.org/package=IDSL.IPA
- name: MZmine 2
  role: Source of a comparison peak list for the same test file (003.d) whose peaks
    are checked against IDSL.IPA thresholds.
- name: xcms-CAMERA
  role: Source of a comparison peak list (chromatographic region-of-interest detection)
    whose peaks are checked against IDSL.IPA thresholds.
- name: MS-DIAL
  role: Source of a comparison peak list (untargeted detection with default parameters)
    whose peaks are checked against IDSL.IPA thresholds.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00120
    title: IDSL.IPA Characterizes the Organic Chemical Space in Untargeted LC/HRMS
      Data Sets
  source_content_hash: sha256:35a72224c8ba30ad81e6d0b3ca8e391bb53c075114176f92638c646ba984035c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - MS-DIAL
  - MZmine
  - MZmine 2
  - R
  - enviPat
  - xcms-CAMERA
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isotopologue-pairing-quality-threshold-evaluation

## Summary

Evaluate baseline-tool peaks that are absent from an IDSL.IPA peaklist against the pipeline's 12C/13C isotopologue pairing and peak quality thresholds, so that peak-count differences between IDSL.IPA, MZmine 2, xcms-CAMERA and MS-DIAL can be explained as threshold-failure rates rather than raw counts.

## When to use

Use when comparing IDSL.IPA peak lists (e.g. 20,253 peaks for MTBLS1684 file 003.d) against MZmine 2, xcms-CAMERA or MS-DIAL output and you must attribute missing peaks to isotopologue pairing or peak quality thresholds.

## When NOT to use

- When the input is already an aligned peak intensity matrix or a feature table that has passed all filtering steps — this skill re-evaluates raw candidate peaks against thresholds, not finished data matrices.
- When the goal is molecular formula annotation or MS2-based structural identification (IDSL.UFA/IDSL.CSA territory) rather than explaining which peaks IDSL.IPA retained.

## Inputs

- IDSL.IPA peaklist (12C monoisotopic isotopologue peaks) for a raw LC/HRMS file such as 003.d from MTBLS1684
- Baseline peaklists from MZmine 2, xcms-CAMERA and MS-DIAL for the same raw file
- Raw LC/HRMS file (mzXML/mzML/netCDF) with MS1 full-scan spectra for re-evaluating EICs against IDSL.IPA quality thresholds

## Outputs

- Table of baseline peaks absent from IDSL.IPA and whether each fails the isotopologue pairing or peak quality thresholds (e.g. peak_count_comparison.csv with threshold-failure rates)
- Aggregate threshold-failure rate quantifying why baseline peaks were not retained by IDSL.IPA

## How to apply

After generating an IDSL.IPA peaklist on a raw LC/HRMS file (e.g. mzXML/mzML for 003.d from MTBLS1684) and baseline peaklists from MZmine 2, xcms-CAMERA and MS-DIAL, identify peaks present in a baseline list but absent from the IDSL.IPA list. For each such peak, test it against the IDSL.IPA isotopologue pairing logic (12C/13C ion pairs in individual mass spectra, with only the m/z values corresponding to the 12C isotopologue used for peak detection, each 12C m/z used in only one m/z cluster) and against the peak quality determination criteria (mass correction, peak smoothing, baseline development for local noise measurement, peak quality determination, S/N ratio). Record how many of these absent peaks fail to pass the thresholds; the article reports a failure rate for such peaks and notes that a majority of rejected EICs were low abundance but passed the S/N ratio threshold, suggesting additional auto-filtering rules are needed.

## Related tools

- **IDSL.IPA** (Supplies the 12C/13C isotopologue pairing logic and the peak quality determination criteria (mass correction, peak smoothing, local-noise baseline, peak quality determination) that baseline peaks are re-evaluated against.) — https://github.com/idslme/IDSL.IPA
- **R** (Language in which IDSL.IPA is implemented and in which the threshold-failure evaluation script is run.) — https://cran.r-project.org/package=IDSL.IPA
- **MZmine 2** (Source of a comparison peak list for the same test file (003.d) whose peaks are checked against IDSL.IPA thresholds.)
- **xcms-CAMERA** (Source of a comparison peak list (chromatographic region-of-interest detection) whose peaks are checked against IDSL.IPA thresholds.)
- **MS-DIAL** (Source of a comparison peak list (untargeted detection with default parameters) whose peaks are checked against IDSL.IPA thresholds.)

## Examples

```
library(IDSL.IPA); IPA_workflow("IPA_parameters.xlsx")
```

## Evaluation signals

- Every baseline peak marked as failing must be re-checked against the documented IDSL.IPA criteria: isotopologue pairing (12C m/z used in only one cluster) AND peak quality (mass correction, smoothing, local-noise baseline, S/N).
- Threshold-failure rate should be computed as a fraction of the total baseline peaks absent from the IDSL.IPA list, not of all detected peaks.
- Consistency check: peaks that fail the isotopologue pairing step should correspond to non-monoisotopic 12C/13C cluster members, which IDSL.IPA by design excludes because it 'detects only the most abundant monoisotopic isotopologues'.
- Cross-check the reported total peak count for the test file (20,253 peaks for 003.d from MTBLS1684) and the fold ratios 1.6x (MZmine 2), 5.4x (xcms-CAMERA) and 2.2x (MS-DIAL) before running threshold evaluation.

## Limitations

- A majority of peaks rejected by IDSL.IPA's peak quality determination were low abundance yet passed the S/N ratio threshold, so a failure rate alone cannot distinguish genuine noise from real signal without additional auto-filtering rules that do not yet exist.
- IDSL.IPA aligned data matrices need further processing (adducts and fragments, statistics filtering, molecular formula annotation) before they can be compared feature-by-feature with other pipelines.
- The threshold evaluation is specific to the 12C/13C pairing design — peaks from non-carbon-containing compounds or peaks whose monoisotopic 12C partner is absent will be excluded by construction.

## Evidence

- [results] Majority of rejected EICs low abundance but passed S/N: [evidence span withheld — anchor status: not_found]
- [readme] README claim of superior sensitivity/specificity: [evidence span withheld — anchor status: external_doc]
