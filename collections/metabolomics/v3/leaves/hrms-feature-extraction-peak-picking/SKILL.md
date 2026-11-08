---
name: hrms-feature-extraction-peak-picking
description: Use when raw LC-HRMS data in .mzML format must be converted into a curated
  chromatographic peak/feature list, e.g. for ENTACT-style mixtures of known MS-amenable
  chemicals or non-targeted analysis workflows.
when_to_use_negative:
- Input is already a feature table or peak list rather than raw .mzML scans (MSS peak
  picking starts from raw .mzML files)
- Feature grouping/merging of isotopes, adducts and in-source fragments into individual
  chemicals is required — the article lists this as future work, not implemented
- The needed export format is not .csv when using the automated script workflow (script/mss_processor.py
  currently only supports .csv export)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Core package; mssmain.get_scans parses .mzML into Python metadata and mssmain.peak_pick
    performs concatenated-scan peak picking with post-processing filters
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Dependency used by mssmain.get_scans to parse raw .mzML files as Python-compatible
    metadata
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Package used inside mssmain.peak_pick to find peak indices from concatenated
    scans
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:8d315f8f4130bf4e20382f8d70eeff42af8d69b8e64f6644f2ecc5a0866f61fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - PeakUtils
  - Python
  - pymzML
  - pymzml
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hrms-feature-extraction-peak-picking

## Summary

Extract chromatographic peaks/features from high-resolution mass spectrometry (HRMS) non-targeted analysis data using the Mass-Suite (MSS) peak-picking pipeline, validated at 99.5% feature-extraction accuracy on mixed chemical standards of known composition.

## When to use

Use when raw LC-HRMS data in .mzML format must be converted into a curated chromatographic peak/feature list, e.g. for ENTACT-style mixtures of known MS-amenable chemicals or non-targeted analysis workflows.

## When NOT to use

- Input is already a feature table or peak list rather than raw .mzML scans (MSS peak picking starts from raw .mzML files)
- Feature grouping/merging of isotopes, adducts and in-source fragments into individual chemicals is required — the article lists this as future work, not implemented
- The needed export format is not .csv when using the automated script workflow (script/mss_processor.py currently only supports .csv export)

## Inputs

- Raw HRMS data files in .mzML format (e.g. EPA ENTACT study samples #505, #506, #508)
- Optional blank/control samples for background feature subtraction and replicate samples for replicate evaluation
- A list of known spiked chemicals (e.g. 398 MS-amenable chemicals present in all three ENTACT mixtures) for accuracy validation

## Outputs

- Feature peak list (extracted chromatographic peaks) per sample, exportable as .csv via the automated workflow (script/mss_processor.py)
- Feature-extraction accuracy metric, e.g. fraction of known spiked peaks matched (99.5% = 396 of 398 matched on ENTACT samples)
- Python-compatible parsed scan metadata (pandas dataframes) from mssmain.get_scans

## How to apply

Import and parse archived .mzML files with mssmain.get_scans, which uses pymzML to convert raw scans into Python-compatible metadata (optional baseline subtraction is available on the converted data). Then run mssmain.peak_pick, which concatenates scans and finds peak indices via PeakUtils, followed by post-processing: a peak width filter, a replicated peak filter, and regression-based peak boundary determination, all aimed at removing poor-quality features such as split peaks, peaks with insufficient scan numbers, and high baseline noise. Optionally apply the pre-trained random forest peak-quality model (built from 15 descriptive peak parameters, packaged as a .pkl file) to further exclude noisy peaks. Multiprocessing can be enabled to cut single-file extraction from 201 ± 1.1 s to 58 ± 1.4 s and batch extraction from 897 ± 9.1 s to 350 ± 37 s; the automated script/mss_processor.py variant only supports .csv export.

## Related tools

- **Mass-Suite (MSS)** (Core package; mssmain.get_scans parses .mzML into Python metadata and mssmain.peak_pick performs concatenated-scan peak picking with post-processing filters) — https://github.com/XiminHu/mass-suite
- **pymzML** (Dependency used by mssmain.get_scans to parse raw .mzML files as Python-compatible metadata) — https://github.com/pymzml/pymzML
- **PeakUtils** (Package used inside mssmain.peak_pick to find peak indices from concatenated scans)

## Examples

```
pip install mass-suite; from mss import mssmain; scans = mssmain.get_scans(path='sample.mzML'); peaks = mssmain.peak_pick(scans)
```

## Evaluation signals

- Match extracted features against the known list of 398 MS-amenable chemicals spiked into all three ENTACT mixtures (#505, #506, #508); expected accuracy is 99.5% (2 peaks out of 398 unmatched)
- Cross-check feature overlap against MSDIAL and XCMS; MSS showed on average 52 ± 5% (MSDIAL) and 52 ± 6% (XCMS) overlap, so full agreement is not expected
- Confirm post-processing removed poor-quality features: no split peaks, no peaks with insufficient scan numbers, no high baseline-noise features
- Verify peak boundaries were determined by regression and that the peak width filter and replicated peak filter were applied before finalizing the peak list
- If multiprocessing is enabled, check processing-time improvement (58 ± 1.4 s vs 201 ± 1.1 s for a single file; 350 ± 37 s vs 897 ± 9.1 s for batch)

## Limitations

- Feature grouping to merge MS features (isotopes, adducts, in-source fragments) as individual chemicals is not yet implemented and is listed as future work
- The random forest peak quality assessment is optional and must be explicitly enabled; without it, noisy peaks rely only on rule-based post-processing filters
- No MSS version number, changelog, or archived release (e.g. Zenodo DOI) is cited; the article states 'Other requirements: none' without Python/dependency version pinning, so reproducibility depends on the environment.yml in the repo
- Authors caution users to develop fundamental understanding of the algorithms and their limitations and assumptions to avoid misinterpretation

## Evidence

- [methods] raw data import and parsing with mssmain.get_scans: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [methods] peak picking with post-processing filters: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features"
- [abstract] 99.5% feature extraction accuracy finding: "MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy"
- [methods] ENTACT benchmark result: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [readme] random forest peak quality assessment: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
