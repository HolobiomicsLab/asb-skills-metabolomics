---
name: peak-quality-post-processing-filtering
description: Use when a raw mzML HRMS file has been peak-picked (e.g., mssmain.peak_pick)
  and the resulting feature list contains noisy, split, or replicate peaks that must
  be filtered before background subtraction, clustering, or feature export.
when_to_use_negative:
- Input is already a curated feature table (e.g., a CSV from MSDIAL, XCMS, or a previous
  MSS run) rather than freshly peak-picked scans
- Feature grouping to merge isotopes, adducts, and in-source fragments is the goal
  — this is not yet implemented in MSS and is listed as future work
- Baseline correction of the raw signal is needed first; baseline subtraction is a
  separate optional step prior to peak picking, not part of post-processing
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Mass-Suite (MSS)
  role: Provides mssmain.peak_pick and post-processing (peak width filter, replicated
    peak filter, regression-based peak boundary determination) plus the packaged pre-trained
    random forest quality model
  repo: https://github.com/XiminHu/mass-suite
- name: PeakUtils
  role: Used by mssmain.peak_pick to find peak indices in concatenated scans prior
    to post-processing filters
- name: pymzml
  role: Parses .mzML raw data into Python-compatible scan metadata upstream of peak
    picking and filtering
  repo: https://github.com/pymzml/pymzML
- name: scikit-learn
  role: Dependency underlying the integrated random forest peak quality assessment
    model
- name: Pandas
  role: Exports the filtered feature list to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:3b0d0c7b5b637fcb2f60aa469f9f0d7915436a6e778fd4bf47c6bcf2e02a5946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Pandas
  - PeakUtils
  - Python
  - pymzml
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-quality-post-processing-filtering

## Summary

Post-process MSS peak-picked features (m/z–RT pairs) with peak width filtering, replicated peak filtering, regression-based peak boundary determination, and optional random-forest peak quality scoring to remove poor-quality features such as split peaks, insufficient scan numbers, and high baseline noise.

## When to use

Use when a raw mzML HRMS file has been peak-picked (e.g., mssmain.peak_pick) and the resulting feature list contains noisy, split, or replicate peaks that must be filtered before background subtraction, clustering, or feature export.

## When NOT to use

- Input is already a curated feature table (e.g., a CSV from MSDIAL, XCMS, or a previous MSS run) rather than freshly peak-picked scans
- Feature grouping to merge isotopes, adducts, and in-source fragments is the goal — this is not yet implemented in MSS and is listed as future work
- Baseline correction of the raw signal is needed first; baseline subtraction is a separate optional step prior to peak picking, not part of post-processing

## Inputs

- Peak-picked feature list (m/z–RT pairs) from mssmain.peak_pick on .mzML scans parsed with mssmain.get_scans/pymzml
- Optional scan-level signal intensity metadata with baseline subtraction applied

## Outputs

- Filtered feature table of m/z–RT pairs (feature_table.csv) with poor-quality peaks removed
- Optional per-peak descriptive parameters and random forest quality scores

## How to apply

After mssmain.peak_pick (which concatenates scans and locates peak indices via PeakUtils), run MSS post-processing that applies a peak width filter, a replicated peak filter, and regression-based peak boundary determination; these target poor-quality features caused by peak splitting, insufficient scan numbers, and high baseline noise. Optionally compute the 15 descriptive parameters per extracted peak and score peak quality with the packaged pre-trained random forest model (a *.pkl model integrated in mss) to exclude noisy peaks. Optionally, an integrated random forest model can serve as a peak quality assessment tool during import. Export the surviving features (m/z–RT pairs) to CSV with pandas, expecting ~99.5% extraction accuracy (398 peaks, 2 unmatched) on ENTACT mixtures with default settings.

## Related tools

- **Mass-Suite (MSS)** (Provides mssmain.peak_pick and post-processing (peak width filter, replicated peak filter, regression-based peak boundary determination) plus the packaged pre-trained random forest quality model) — https://github.com/XiminHu/mass-suite
- **PeakUtils** (Used by mssmain.peak_pick to find peak indices in concatenated scans prior to post-processing filters)
- **pymzml** (Parses .mzML raw data into Python-compatible scan metadata upstream of peak picking and filtering) — https://github.com/pymzml/pymzML
- **scikit-learn** (Dependency underlying the integrated random forest peak quality assessment model)
- **Pandas** (Exports the filtered feature list to CSV)

## Examples

```
from mss import mssmain; scans = mssmain.get_scans('sample.mzML'); peaks = mssmain.peak_pick(scans); features = mssmain.post_process(peaks); features.to_csv('feature_table.csv')
```

## Evaluation signals

- Output feature_table.csv exists and contains m/z–RT feature pairs
- On EPA ENTACT mixtures (#505, #506, #508; 398 MS-amenable chemicals) with default settings, MSS extracts ~99.5% of known peaks (2 of 398 unmatched)
- Features flagged with peak splitting, insufficient scan numbers, or high baseline noise are absent from the filtered table
- Random forest quality scores (when enabled) exclude peaks below the model's quality threshold, and only peaks passing width and replicate filters remain
- Filtered feature counts are consistent with MSDIAL/XCMS comparison (~52% feature overlap expected with those tools)

## Limitations

- Post-processing targets peak splitting, insufficient scan numbers, and high baseline noise, but feature grouping (merging isotopes, adducts, in-source fragments) is not implemented and is listed as future work
- Users must understand the algorithms' limitations and assumptions to avoid misinterpretation; no specific algorithm limitations are enumerated by the authors
- Package has no stated version number or archived release (e.g., Zenodo DOI), only a GitHub URL and pypi listing, complicating reproducibility

## Evidence

- [methods] Post-processing applies width, replicate, and boundary filters to reduce poor quality features: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features"
- [readme] Random forest model assesses peak quality during import: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
- [methods] MSS peak picking finds indices via PeakUtils: "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53])"
- [methods] MSS extracted 99.5% of known ENTACT peaks (2 of 398 unmatched): "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [methods] Raw mzML parsed with pymzml via mssmain.get_scans: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
