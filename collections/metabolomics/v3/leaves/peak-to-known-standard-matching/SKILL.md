---
name: peak-to-known-standard-matching
description: Use when an HRMS feature list from .mzML data of mixed chemical standards
  with known composition needs reliability validation by matching detected peaks to
  the spiked standard list and computing extraction accuracy.
when_to_use_negative:
- Input is already a curated feature table from a tool other than the extraction pipeline
  under test — cross-tool agreement should instead be measured as overlap (e.g., the
  ~52% MSS vs. MSDIAL/XCMS comparison), not standard-matching accuracy
- Samples are real environmental or unknown-composition matrices with no spiked standard
  list, so no ground-truth peak inventory exists to match against
- The goal is identification/annotation of unknown features (e.g., MS/MS fragment
  matching against MoNA MassBank), not extraction reliability assessment
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Source of mssmain.get_scans and mssmain.peak_pick used to produce the feature
    peak list that is matched against the known spiked-chemical list
  repo: https://github.com/XiminHu/mass-suite
- name: pymzml
  role: Parses raw .mzML scans into Python-compatible metadata as the first step before
    peak picking and standard matching
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Finds peak indices during mssmain.peak_pick so that every known spiked chemical
    has a chance to be detected and matched
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:0ee111fdb40f85000ef5b1ba159fcf977cea259b70c4b206834866a53f282648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - PeakUtils
  - Python
  - pymzml
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# peak-to-known-standard-matching

## Summary

Validates an HRMS feature-extraction workflow by matching every extracted chromatographic peak against a curated list of chemicals of known composition (e.g., the 398 MS-amenable chemicals spiked into EPA ENTACT mixtures #505, #506, #508) and reporting accuracy as the fraction of known peaks recovered, which for Mass-Suite (MSS) was 99.5% (2 of 398 unmatched).

## When to use

Use when an HRMS feature list from .mzML data of mixed chemical standards with known composition needs reliability validation by matching detected peaks to the spiked standard list and computing extraction accuracy.

## When NOT to use

- Input is already a curated feature table from a tool other than the extraction pipeline under test — cross-tool agreement should instead be measured as overlap (e.g., the ~52% MSS vs. MSDIAL/XCMS comparison), not standard-matching accuracy
- Samples are real environmental or unknown-composition matrices with no spiked standard list, so no ground-truth peak inventory exists to match against
- The goal is identification/annotation of unknown features (e.g., MS/MS fragment matching against MoNA MassBank), not extraction reliability assessment

## Inputs

- Archived .mzML data files for samples of known composition (e.g., EPA ENTACT samples #505, #506, #508)
- Known spiked-chemical list (398 MS-amenable chemicals present in all three mixtures)
- Extracted feature peak list from mssmain.peak_pick (default MSS settings)

## Outputs

- feature_extraction_accuracy.csv containing the feature-extraction accuracy (expected 99.5%, i.e., 2 out of 398 peaks unmatched)
- Count of matched vs. unmatched known peaks

## How to apply

Process the archived .mzML files of the standard mixtures through the full extraction pipeline first (mssmain.get_scans with pymzML, then mssmain.peak_pick with PeakUtils-based peak indexing plus post-processing: peak width filter, replicated peak filter, and regression-based peak boundary determination; optionally the pre-trained random forest peak quality assessment built from 15 descriptive peak parameters). Generate the feature peak list with default MSS settings, then match each extracted chromatographic peak against the known spiked-chemical list (398 MS-amenable chemicals present in all three mixtures #505, #506, #508). Compute feature-extraction accuracy as the fraction of known peaks matched; a correct run on the ENTACT mixtures should recover 99.5% of peaks (396/398). Rationale: benchmarking against standards of known composition grounds the accuracy number in ground truth rather than cross-tool overlap alone.

## Related tools

- **Mass-Suite (MSS)** (Source of mssmain.get_scans and mssmain.peak_pick used to produce the feature peak list that is matched against the known spiked-chemical list) — https://github.com/XiminHu/mass-suite
- **pymzml** (Parses raw .mzML scans into Python-compatible metadata as the first step before peak picking and standard matching) — https://github.com/pymzml/pymzML
- **PeakUtils** (Finds peak indices during mssmain.peak_pick so that every known spiked chemical has a chance to be detected and matched)

## Examples

```
from mss import mssmain; scans = mssmain.get_scans('ENTACT_505.mzML'); peaks = mssmain.peak_pick(scans)
```

## Evaluation signals

- Feature-extraction accuracy should equal 99.5% on the ENTACT mixtures #505, #506, #508 — exactly 2 out of 398 known peaks unmatched
- Every known spiked chemical must be checked against the extracted peak list in all three mixtures before computing the accuracy fraction
- Output file feature_extraction_accuracy.csv must be produced containing the computed accuracy
- Extraction must be run with default MSS settings (no ad hoc parameter tuning) so the 99.5% benchmark is comparable
- Optionally confirm the random forest peak quality model (trained on 15 descriptive peak parameters) was applied to exclude noisy peaks before matching

## Limitations

- Only 2 of 398 unmatched peaks are tolerable; the accuracy figure is specific to the ENTACT mixtures and may not transfer to other standard panels or matrix types
- The random forest peak quality assessment is optional; skipping it may retain noisy peaks that inflate or distort the match statistics
- MSS does not yet implement feature grouping (merging isotopes, adducts, in-source fragments as individual chemicals), which could affect one-to-one peak-to-standard matching
- No packaged version number, archived release, or dependency versions are specified, so reproducing the exact 99.5% benchmark depends on the repository state used

## Evidence

- [abstract] MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy: "MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy"
- [methods] MSS extracted 99.5% of peaks (2 peaks out of 398 didn't match) known to be present in all three mixtures [73].: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures"
- [methods] archived samples (mixed chemical standards) from the EPA ENTACT study [73] (Additional file 1: Text S2; sample numbers #505, #506 and #508; 398 MS-amenable chemicals in total): "archived samples (mixed chemical standards) from the EPA ENTACT study [73] (Additional file 1: Text S2; sample numbers #505, #506 and #508; 398 MS-amenable chemicals in total)"
- [methods] The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53]): "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package"
- [methods] raw data (.mzML format) is first imported and parsed with pymzML [52] as Python-compatible metadata with the mssmain.get_scans function.: [evidence span withheld — anchor status: not_found]
- [readme] README release note that the extraction pipeline was benchmarked against an independent tool: "Feature extraction pipeline validated comparing with msdial pipeline output"
