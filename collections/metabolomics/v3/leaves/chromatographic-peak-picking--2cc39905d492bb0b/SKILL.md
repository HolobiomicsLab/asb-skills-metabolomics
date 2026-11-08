---
name: chromatographic-peak-picking
description: Use when starting from raw HRMS .mzML files and you need to extract chromatographic
  features (m/z–RT pairs) for non-targeted analysis, e.g. ENTACT-style mixtures or
  environmental water samples.
when_to_use_negative:
- Input is already a feature table — use MSS alignment, clustering (dm.ms_cluster),
  or source-tracking modules instead of peak picking
- The goal is MS/MS fragment searching or formula prediction (mss frag module against
  MoNA MassBank), not chromatographic feature extraction
- Features must be grouped into chemicals (merging isotopes, adducts, in-source fragments)
  — MSS explicitly lists feature grouping as future work, so peak picking alone will
  not deliver it
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3521
tools:
- name: Mass-Suite (MSS)
  role: Provides mssmain.peak_pick (peak detection) and mssmain.get_scans, plus post-processing
    filters and the pre-trained random forest peak-quality model
  repo: https://github.com/XiminHu/mass-suite
- name: pymzml
  role: Parses .mzML raw data into Python-compatible scan metadata feeding peak_pick
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Underlying library used by mssmain.peak_pick to find peak indices in concatenated
    scans
- name: Pandas
  role: Exports the extracted feature list (m/z–RT pairs) to CSV
- name: scikit-learn
  role: Dependency backing the integrated random forest peak-quality assessment model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:f6312eadf9f208ee4bf90a29fab7573514482c946f424679d5bbe5e17c6b95a0
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# chromatographic-peak-picking

## Summary

Detects chromatographic peaks from HRMS scan data to build an m/z–RT feature table using the Mass-Suite (MSS) function mssmain.peak_pick, which concatenates scans and locates peak indices with PeakUtils, followed by post-processing filters to remove poor-quality features.

## When to use

Use when starting from raw HRMS .mzML files and you need to extract chromatographic features (m/z–RT pairs) for non-targeted analysis, e.g. ENTACT-style mixtures or environmental water samples.

## When NOT to use

- Input is already a feature table — use MSS alignment, clustering (dm.ms_cluster), or source-tracking modules instead of peak picking
- The goal is MS/MS fragment searching or formula prediction (mss frag module against MoNA MassBank), not chromatographic feature extraction
- Features must be grouped into chemicals (merging isotopes, adducts, in-source fragments) — MSS explicitly lists feature grouping as future work, so peak picking alone will not deliver it

## Inputs

- Raw HRMS data in .mzML format
- Parsed scan metadata from mssmain.get_scans (Python-compatible dataframe)
- Optional blank/experimental samples for background subtraction downstream

## Outputs

- Feature table of m/z–RT pairs (e.g. feature_table.csv exported via pandas)
- Per-peak descriptive parameters (15 parameters) and optional random-forest quality scores

## How to apply

First import and parse the .mzML raw data with mssmain.get_scans, which reads via pymzml into Python-compatible scan metadata; optionally apply baseline subtraction based on signal intensities before extraction. Then call mssmain.peak_pick, which concatenates scans and finds peak indices using the PeakUtils package. Apply MSS post-processing — the peak width filter, replicated peak filter, and regression-based peak boundary determination — to reduce poor-quality features such as peak splitting, insufficient scan numbers, and high baseline noise. Optionally compute the 15 descriptive parameters per peak and score peak quality with the packaged pre-trained random forest model to exclude noisy peaks. Finally export the feature list (one m/z–RT pair per feature) to CSV with pandas. With default settings this pipeline achieved 99.5% peak extraction accuracy on EPA ENTACT mixtures (#505, #506, #508; 398 MS-amenable chemicals), missing only 2 peaks.

## Related tools

- **Mass-Suite (MSS)** (Provides mssmain.peak_pick (peak detection) and mssmain.get_scans, plus post-processing filters and the pre-trained random forest peak-quality model) — https://github.com/XiminHu/mass-suite
- **pymzml** (Parses .mzML raw data into Python-compatible scan metadata feeding peak_pick) — https://github.com/pymzml/pymzML
- **PeakUtils** (Underlying library used by mssmain.peak_pick to find peak indices in concatenated scans)
- **Pandas** (Exports the extracted feature list (m/z–RT pairs) to CSV)
- **scikit-learn** (Dependency backing the integrated random forest peak-quality assessment model)

## Examples

```
pip install mass-suite; import mss; python -m mss_processor.py
```

## Evaluation signals

- feature_table.csv exists and is a CSV whose rows are m/z–RT feature pairs
- On the EPA ENTACT benchmark (#505/#506/#508, 398 known chemicals), default settings recover 99.5% of peaks (2 of 398 unmatched) — expect roughly 396 matched features
- Cross-tool sanity check: MSS feature overlap is ~52 ± 5% vs MSDIAL and ~52 ± 6% vs XCMS, so a feature count wildly above or below that overlap range warrants review
- Post-processing removed poor-quality features: verify peak width, replicated-peak, and regression-based boundary filters ran (no peak splitting or insufficient-scan features remain)
- If the random forest quality scorer was enabled, noisy/noisy-scored peaks are excluded from the exported table

## Limitations

- Parameter settings must be tuned per dataset — the paper's DEMO parameter settings 'would typically be different for each example'
- Feature grouping (merging isotopes, adducts and in-source fragments as individual chemicals) is not yet implemented; it is listed as future work
- The automated script workflow (mss_processor.py) only supports .csv export
- The paper gives no pinned MSS version, DOI/archived release, or dependency versions ('Other requirements: none'), so reproducibility depends on the actively maintained GitHub repo

## Evidence

- [methods] peak_pick concatenates scans and uses PeakUtils: "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53])"
- [methods] mzML parsed via pymzml with get_scans: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [methods] post-processing filters reduce poor-quality features: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features"
- [methods] 99.5% extraction benchmark on ENTACT: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [readme] random forest peak quality assessment in import function: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
