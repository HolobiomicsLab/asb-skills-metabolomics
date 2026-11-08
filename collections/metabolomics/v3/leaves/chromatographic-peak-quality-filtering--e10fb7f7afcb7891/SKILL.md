---
name: chromatographic-peak-quality-filtering
description: Use when raw centroid/profile HRMS scans (.mzML) have been peak-picked
  and the feature list must be cleaned of low-quality peaks — split peaks, peaks spanning
  too few scans, and high-baseline-noise features — before annotation or statistical
  analysis.
when_to_use_negative:
- The input is already a cleaned feature table exported by another tool (e.g., MSDIAL,
  XCMS, MZmine) — quality filtering has already been applied upstream.
- The goal is merging isotope, adduct, or in-source-fragment features into individual
  chemicals — feature grouping is not yet implemented in MSS and is listed as future
  work.
- The target is identification (MS/MS fragment searching against MoNA MassBank) rather
  than removing poor-quality chromatographic peaks.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Hosts mssmain.peak_pick and its post-processing chain (peak width filter,
    replicated peak filter, regression-based peak boundary determination) plus the
    bundled random forest quality model
  repo: https://github.com/XiminHu/mass-suite
- name: PeakUtils
  role: Finds peak indices on concatenated scans inside mssmain.peak_pick, defining
    the candidate peaks the filters then prune
- name: pymzML
  role: Parses raw .mzML scans into Python-compatible metadata that feeds peak picking
    and downstream quality filtering
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:55e2ca4548c0e1f2e1a8f362790a65fcbb831d46fe2b99536f4e702aeba2acd1
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# chromatographic-peak-quality-filtering

## Summary

Post-processing filter chain applied after mssmain.peak_pick to remove poor-quality chromatographic features (split peaks, insufficient scan numbers, high baseline noise) from HRMS .mzML data, optionally augmented by a pre-trained random forest peak quality classifier.

## When to use

Use when raw centroid/profile HRMS scans (.mzML) have been peak-picked and the feature list must be cleaned of low-quality peaks — split peaks, peaks spanning too few scans, and high-baseline-noise features — before annotation or statistical analysis.

## When NOT to use

- The input is already a cleaned feature table exported by another tool (e.g., MSDIAL, XCMS, MZmine) — quality filtering has already been applied upstream.
- The goal is merging isotope, adduct, or in-source-fragment features into individual chemicals — feature grouping is not yet implemented in MSS and is listed as future work.
- The target is identification (MS/MS fragment searching against MoNA MassBank) rather than removing poor-quality chromatographic peaks.

## Inputs

- Peak-picked feature list from mssmain.peak_pick (derived from .mzML scans parsed via mssmain.get_scans/pymzML)
- Pre-trained random forest peak quality model (*.pkl shipped in the mss package, scored on 15 descriptive peak parameters)
- Known-composition standard list (e.g., 398 MS-amenable chemicals spiked across EPA ENTACT samples #505, #506, #508) for validation

## Outputs

- Filtered list of high-quality chromatographic peaks with regression-determined boundaries
- Feature-extraction accuracy metric computed against a known standard list (e.g., feature_extraction_accuracy.csv)

## How to apply

After mssmain.get_scans parses the .mzML file with pymzML into Python-compatible metadata and mssmain.peak_pick concatenates scans and finds peak indices using PeakUtils, apply the built-in post-processing: a peak width filter, a replicated peak filter, and regression-based peak boundary determination. These filters target the article's named failure modes — peak splitting, insufficient scan numbers, and high baseline noise. Optionally pass peaks through the random forest quality-assessment model (shipped as a .pkl in the mss package) that scores peaks on 15 descriptive peak parameters; treat it as an optional gate, not a requirement. The decision point is whether downstream analysis demands maximum recall (keep the unfiltered post-processed list) or maximum precision (apply the RF filter). Validate by matching the retained peaks against a known-composition standard list.

## Related tools

- **Mass-Suite (MSS)** (Hosts mssmain.peak_pick and its post-processing chain (peak width filter, replicated peak filter, regression-based peak boundary determination) plus the bundled random forest quality model) — https://github.com/XiminHu/mass-suite
- **PeakUtils** (Finds peak indices on concatenated scans inside mssmain.peak_pick, defining the candidate peaks the filters then prune)
- **pymzML** (Parses raw .mzML scans into Python-compatible metadata that feeds peak picking and downstream quality filtering) — https://github.com/pymzml/pymzML

## Examples

```
from mss import mssmain; scans = mssmain.get_scans('ENTACT_505.mzML'); peaks = mssmain.peak_pick(scans)
```

## Evaluation signals

- Against EPA ENTACT mixed-chemical standards #505/#506/#508, retained peaks should match at least 99.5% of the 398 known MS-amenable chemicals (no more than 2 unmatched).
- No surviving feature should exhibit the article's named artifact modes: split peaks, insufficient scan numbers, or high baseline noise.
- Optionally verify the random forest quality gate is functioning: peaks scored on the 15 descriptive peak parameters and low-quality peaks excluded.
- Compare retained-feature overlap with an independent tool's output (MSDIAL or XCMS) as a sanity check; the article reports ~52 ± 5–6% average overlap for such a comparison.
- Confirm peak boundaries were set by the regression-based determination step (not raw PeakUtils indices) before quantification.

## Limitations

- Even after filtering, 2 of the 398 known ENTACT peaks were not matched — expect near-but-not-perfect recall on complex known mixtures.
- The random forest quality assessment is optional; without it, poor-quality peaks may persist, and users are cautioned to understand algorithm limitations and assumptions to avoid misinterpretation.
- The MSS package has no pinned version number, no archived release (e.g., Zenodo DOI), no changelog, and no specified Python/dependency versions, so filter behavior may drift across releases; feature grouping remains unimplemented.

## Evidence

- [methods] Post-processing filters remove named artifact modes: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features (e.g., peak splitting, insufficient scan"
- [readme] Random forest optional quality gate: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
- [methods] Validated filter performance on known standards: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
