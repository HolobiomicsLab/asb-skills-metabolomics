---
name: xcms-feature-detection-comparison
description: Use when raw LC/MS mzML files are available, an XCMS peak list exists
  as the baseline, and the goal is to measure which true peaks another detector recovers
  that XCMS missed after removing blank/degenerate signals.
when_to_use_negative:
- You already have a final curated feature table and only need peak curation/scoring
  — use PeakDetective curatePeaks instead of a recovery comparison against XCMS
- Only an XCMS peak list exists and there is no second detector or raw mzML data to
  generate an alternative peak set
- The goal is classifier training/active learning rather than quantifying detection
  differences between XCMS and another detector
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: Baseline peak detector whose peak list is intersected with the alternative
    detector's output to quantify recovered peaks
  repo: https://github.com/pattilab/PeakDetective
- name: PeakDetective
  role: De novo peak detection producing peak_scores_pd.csv/peak_intensities_pd.csv
    that are compared against XCMS
  repo: https://github.com/pattilab/PeakDetective
- name: detection_helper.PeakList
  role: Wraps XCMS run (runXCMS), reads XCMS peak lists, and invokes mz.unity-based
    removeRedundancy and backgroundSubtract
  repo: https://github.com/pattilab/PeakDetective
- name: mz.unity
  role: Annotates degenerate signals (isotopes/adducts/fragments) among recovered
    peaks via removeRedundancy
- name: pyteomics
  role: Required Python package for reading raw LC/MS data in PeakDetective
  repo: https://github.com/pattilab/PeakDetective
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:3cc415c6641464f4c8d4cd10e9f4651fa4033cb1c2c6d49debbef23a460b652b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - XCMS
  - detection_helper.PeakList
  - mz.unity
  - numpy
  - pyteomics
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# xcms-feature-detection-comparison

## Summary

Run XCMS as the baseline feature/peak detector on raw mzML files and quantitatively compare its peak list against an alternative detector (e.g., PeakDetective de novo detection) to recover peaks XCMS missed, then annotate degeneracies with blank subtraction and mz.unity to count unique recovered signals.

## When to use

Use when raw LC/MS mzML files are available, an XCMS peak list exists as the baseline, and the goal is to measure which true peaks another detector recovers that XCMS missed after removing blank/degenerate signals.

## When NOT to use

- You already have a final curated feature table and only need peak curation/scoring — use PeakDetective curatePeaks instead of a recovery comparison against XCMS
- Only an XCMS peak list exists and there is no second detector or raw mzML data to generate an alternative peak set
- The goal is classifier training/active learning rather than quantifying detection differences between XCMS and another detector

## Inputs

- Raw LC/MS mzML files (SARS-CoV-2 hamster plasma dataset)
- XCMS-generated peak list (peaks.csv) with mz and rt columns
- Blank samples keyed as 'blank' / 'NIST'-equivalent blanks
- PeakDetective trained model weights (PeakDetectiveObject directory) and score cutoff (0.8)

## Outputs

- peakdetective_xcms_peak_recovery_comparison.csv comparing XCMS vs PeakDetective peak recovery
- peak_scores_pd.csv and peak_intensities_pd.csv from de novo detection
- Counts of XCMS-missed peaks recovered (2,414) and unique recovered signals after degeneracy annotation (623)

## How to apply

Run XCMS on the mzML files with the paper's parameter set: peakWidth=(13.8, 114.6) s, s2n=13.6, noise=1, mzDiff=0.0144, prefilter=5, polarity='negative', and 25.5 ppm MS1 tolerance (detection_helper.PeakList().runXCMS). In parallel, run PeakDetective de novo detection via integ.detectPeaks with cutoff=0.8, intensityCutoff (noise)=1000, numDataPoints=3 consecutive scans above noise per ROI, window=0.1 min, detectFrac=0.5, and RT alignment enabled, writing peak_scores_pd.csv and peak_intensities_pd.csv. After imputeRowMin, apply backgroundSubtract with blankKey 'blank', sampleKey for 'NIST'-equivalent blanks, factor 2, then removeRedundancy with corrThresh=0.9, rtThresh=0.5 min, polarity=-1, and 25.5 ppm tolerance to annotate isotopes/adducts/fragments (mz.unity degeneracies). Intersect the recovered peak set with the XCMS peak list and count (a) true peaks scored above cutoff that XCMS missed and (b) unique recovered signals surviving blank subtraction and degeneracy annotation. On the SARS-CoV-2 hamster plasma dataset this reproduces 2,414 XCMS-missed peaks recovered and 623 unique recovered signals, with >70% of recovered peaks being degenerate signals.

## Related tools

- **XCMS** (Baseline peak detector whose peak list is intersected with the alternative detector's output to quantify recovered peaks) — https://github.com/pattilab/PeakDetective
- **PeakDetective** (De novo peak detection producing peak_scores_pd.csv/peak_intensities_pd.csv that are compared against XCMS) — https://github.com/pattilab/PeakDetective
- **detection_helper.PeakList** (Wraps XCMS run (runXCMS), reads XCMS peak lists, and invokes mz.unity-based removeRedundancy and backgroundSubtract) — https://github.com/pattilab/PeakDetective
- **mz.unity** (Annotates degenerate signals (isotopes/adducts/fragments) among recovered peaks via removeRedundancy)
- **pyteomics** (Required Python package for reading raw LC/MS data in PeakDetective) — https://github.com/pattilab/PeakDetective

## Examples

```
det = detection_helper.PeakList(); det.runXCMS(mzmlFolder, "peaks.csv", "negative", 25.5, (13.8,114.6), s2n=13.6, noise=1, mzDiff=0.0144, prefilter=5)
```

## Evaluation signals

- Recovered-peak count from the intersection should reproduce 2,414 peaks XCMS missed on the SARS-CoV-2 hamster dataset
- Unique recovered signal count after blank subtraction and mz.unity degeneracy annotation should reproduce 623 peaks
- PeakDetective score distribution should be bimodal at the chosen cutoff with the reported TPR/FDR estimates from validation data
- Blank-subtracted peaks should show sample/blank intensity ratio above the backgroundSubtract factor (2) after imputeRowMin
- Degeneracy fraction among recovered peaks should be >70%, consistent with mz.unity annotations

## Limitations

- XCMS R packages must be installed (R >v4.0 on the path) before running XCMS or mz.unity degeneracy removal; installation takes >20 min
- Most recovered peaks (>70%) are degenerate signals, so raw recovery counts must not be interpreted as unique biological features
- Recovery counts depend on exact XCMS parameters (s2n=13.6, prefilter=5, 25.5 ppm) and PeakDetective cutoff (0.8); deviating from these changes the intersection

## Evidence

- [results] In de novo peak detection on the SARS-CoV-2 hamster plasma dataset, PeakDetective recovered 2,414 peaks that XCMS missed; blank subtraction and mz.unity annotation revealed that >70% of these recovered peaks were degenerate signals.: [evidence span withheld — anchor status: not_found]
