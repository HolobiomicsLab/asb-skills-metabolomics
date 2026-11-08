---
name: feature-extraction-accuracy-benchmarking
description: Use when you need to quantify how faithfully an HRMS peak-detection pipeline
  recovers known analytes, by matching extracted chromatographic peaks against a spiked-chemical
  list of known composition (e.g., 398 MS-amenable chemicals in ENTACT mixtures).
when_to_use_negative:
- Samples are of unknown composition — there is no known spiked-chemical list to match
  against, so recovered-fraction accuracy cannot be computed
- The goal is cross-tool feature agreement rather than known-analyte recovery; for
  that, MSS reports ~52% overlap with MSDIAL (52 ± 5%) and XCMS (52 ± 6%) instead
- Input is already a processed feature table — re-running get_scans/peak_pick would
  be redundant; benchmarking requires raw .mzML scans
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: HRMS data analysis package whose feature extraction (mssmain.peak_pick) is
    the pipeline being benchmarked
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Parses the raw .mzML scans into Python-compatible metadata via mssmain.get_scans
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Finds peak indices on concatenated scans during mssmain.peak_pick
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:84806f6a273f0acfe4858e66cdc5c5a35b8220e23c8de21dd6172709b84c0d19
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# feature-extraction-accuracy-benchmarking

## Summary

Benchmarks an HRMS feature-extraction workflow's reliability by running it on samples of known composition (EPA ENTACT mixed chemical standards #505, #506, #508) and computing the fraction of known peaks recovered, following the Mass-Suite (MSS) validation design that achieved 99.5% feature-extraction accuracy.

## When to use

Use when you need to quantify how faithfully an HRMS peak-detection pipeline recovers known analytes, by matching extracted chromatographic peaks against a spiked-chemical list of known composition (e.g., 398 MS-amenable chemicals in ENTACT mixtures).

## When NOT to use

- Samples are of unknown composition — there is no known spiked-chemical list to match against, so recovered-fraction accuracy cannot be computed
- The goal is cross-tool feature agreement rather than known-analyte recovery; for that, MSS reports ~52% overlap with MSDIAL (52 ± 5%) and XCMS (52 ± 6%) instead
- Input is already a processed feature table — re-running get_scans/peak_pick would be redundant; benchmarking requires raw .mzML scans

## Inputs

- Archived .mzML data files for known-composition mixed chemical standard samples (EPA ENTACT samples #505, #506, #508)
- Known spiked-chemical list (398 MS-amenable chemicals present in all three mixtures)

## Outputs

- Feature peak list extracted with default MSS settings
- Feature-extraction accuracy as the fraction of known peaks matched (e.g., feature_extraction_accuracy.csv, expected 99.5%)

## How to apply

Import archived .mzML files for the known-composition samples with mssmain.get_scans, which parses raw scans into Python-compatible metadata via pymzml. Run mssmain.peak_pick to concatenate scans, find peak indices with PeakUtils, and apply post-processing (peak width filter, replicated peak filter, regression-based peak boundary determination) that removes split peaks, features with insufficient scan numbers, and high baseline noise; optionally apply the pre-trained random forest peak quality model built from 15 descriptive peak parameters. Generate the feature peak list with default MSS settings and match each extracted chromatographic peak against the known spiked-chemical list (398 chemicals present in all three mixtures). Compute accuracy as the fraction of known peaks matched — for MSS, 2 of 398 peaks went unmatched, i.e., 99.5%.

## Related tools

- **Mass-Suite (MSS)** (HRMS data analysis package whose feature extraction (mssmain.peak_pick) is the pipeline being benchmarked) — https://github.com/XiminHu/mass-suite
- **pymzML** (Parses the raw .mzML scans into Python-compatible metadata via mssmain.get_scans) — https://github.com/pymzml/pymzML
- **PeakUtils** (Finds peak indices on concatenated scans during mssmain.peak_pick)

## Examples

```
from mss import mssmain; scans = mssmain.get_scans(path='ENTACT_505.mzML'); peaks = mssmain.peak_pick(scans)
```

## Evaluation signals

- Feature-extraction accuracy equals the fraction of the 398 known chemicals matched — expected value 99.5% (2 peaks out of 398 not matched)
- Exactly the known spiked-chemical list is used as the matching target; unmatched peaks are countable and attributable
- Poor-quality features (split peaks, insufficient scan numbers, high baseline noise) are removed by the peak width filter, replicated peak filter, and regression-based peak boundary determination
- Post-processing retains chromatographic peak boundaries determined by regression rather than raw index extents
- Result recorded as feature_extraction_accuracy.csv with the matched fraction per the known-composition mixtures

## Limitations

- No version number stated for the MSS package, and no DOI or archived release (e.g., Zenodo) cited for reproducibility — only a GitHub URL and pypi.org listing
- No Python version or dependency versions specified in requirements ('Other requirements: none.'), though the README lists minimums (e.g., pymzML 2.4.6+)
- Feature grouping to merge isotopes, adducts, and in-source fragments as individual chemicals is not yet implemented — listed as future work
- Users are cautioned to develop fundamental understanding of the algorithms, their limitations, and assumptions; no specific algorithm limitations are enumerated in the discussion section
- Accuracy is measured against the subset of 398 MS-amenable chemicals known to be present in all three mixtures, not the full sample complexity

## Evidence

- [abstract] MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy: "MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy"
- [methods] ENTACT mixtures recovery detail: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [methods] Peak picking mechanics: "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53])"
- [methods] Post-processing filters: "regression-based peak boundary determination) to reduce poor quality features (e.g., peak splitting, insufficient scan numbers, high baseline noise)"
- [methods] Raw data import step: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [readme] Random forest peak quality option: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
- [methods] ENTACT dataset definition: "archived samples (mixed chemical standards) from the EPA ENTACT study [73] (Additional file 1: Text S2; sample numbers #505, #506 and #508; 398 MS-amenable chemicals in total)"
