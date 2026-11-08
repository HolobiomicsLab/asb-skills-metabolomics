---
name: random-forest-peak-quality-scoring
description: Use when peaks have already been detected from .mzML HRMS scans with
  mssmain.peak_pick and you need to filter out poor-quality features (peak splitting,
  insufficient scan numbers, high baseline noise) using a pre-trained ML model.
when_to_use_negative:
- Input is already a curated feature table (e.g., a CSV of m/z–RT pairs) — scoring
  operates on raw peak candidates from peak_pick, not on aligned features.
- You need feature grouping/merging of isotopes, adducts, or in-source fragments —
  the article lists this as not yet implemented (future work), so the RF scorer cannot
  substitute for it.
- Your peaks come from non-mzML vendor formats — the pipeline's import stage only
  supports .mzML in this workflow.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Mass-Suite (MSS)
  role: Hosts the mssmain.peak_pick and post-processing pipeline; ships the pre-trained
    random forest .pkl model used to score peak quality
  repo: https://github.com/XiminHu/mass-suite
- name: pymzml
  role: Parses .mzML raw data into Python-compatible scan metadata upstream of peak
    picking and scoring
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Finds peak indices on concatenated scans prior to quality scoring
- name: scikit-learn
  role: Runtime dependency for the random forest classifier (>= 0.23.0) used for peak
    quality assessment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:f59772654ee7b6e9d0535b44cde49245ca2115461d519cdf9d50f144db9f687f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# random-forest-peak-quality-scoring

## Summary

Optional peak quality assessment step in the Mass-Suite (MSS) feature extraction pipeline that scores detected chromatographic peaks with a packaged pre-trained random forest model to exclude noisy, split, or poor-quality features before exporting the m/z–RT feature table.

## When to use

Use when peaks have already been detected from .mzML HRMS scans with mssmain.peak_pick and you need to filter out poor-quality features (peak splitting, insufficient scan numbers, high baseline noise) using a pre-trained ML model.

## When NOT to use

- Input is already a curated feature table (e.g., a CSV of m/z–RT pairs) — scoring operates on raw peak candidates from peak_pick, not on aligned features.
- You need feature grouping/merging of isotopes, adducts, or in-source fragments — the article lists this as not yet implemented (future work), so the RF scorer cannot substitute for it.
- Your peaks come from non-mzML vendor formats — the pipeline's import stage only supports .mzML in this workflow.

## Inputs

- Peak indices / chromatographic peak candidates from mssmain.peak_pick on .mzML data parsed via pymzml
- 15 descriptive parameters per extracted peak (m/z, RT, peak width, area, SNR-style metrics)
- Packaged pre-trained random forest model file (*.pkl shipped in the mss module)

## Outputs

- Quality-scored peak list with low-quality features removed
- Filtered m/z–RT feature table suitable for export to feature_table.csv via pandas

## How to apply

After mssmain.peak_pick concatenates scans and finds peak indices with PeakUtils, run the MSS peak post-processing (peak width filter, replicated peak filter, regression-based peak boundary determination) to remove obviously poor features. Then apply the random forest classifier integrated into the MSS import/feature-extraction function as an optional peak quality assessment tool: for each extracted peak, compute its descriptive parameters, feed them to the packaged pre-trained model (a .pkl file shipped in the mss module), and drop peaks scored as low quality. Keep the surviving features and export the m/z–RT pairs to CSV with pandas. The decision point is whether to enable the model — it is optional by design, and users are cautioned to understand algorithm assumptions before trusting scores. Validate against the ENTACT benchmark, where default settings recovered 99.5% of 398 known peaks (2 unmatched).

## Related tools

- **Mass-Suite (MSS)** (Hosts the mssmain.peak_pick and post-processing pipeline; ships the pre-trained random forest .pkl model used to score peak quality) — https://github.com/XiminHu/mass-suite
- **pymzml** (Parses .mzML raw data into Python-compatible scan metadata upstream of peak picking and scoring) — https://github.com/pymzml/pymzML
- **PeakUtils** (Finds peak indices on concatenated scans prior to quality scoring)
- **scikit-learn** (Runtime dependency for the random forest classifier (>= 0.23.0) used for peak quality assessment)

## Examples

```
from mss.mssmain import peak_pick; peaks = peak_pick(scans, points_threshold=6, intensity_threshold=0.01, signal_noise=5)
```

## Evaluation signals

- feature_table.csv exists and is a valid CSV whose rows are m/z–RT pairs after quality filtering
- On the EPA ENTACT mixtures (#505, #506, #508), default settings recover ~99.5% of the 398 known MS-amenable chemicals (2 peaks unmatched)
- Poor-quality artifacts (peak splitting, insufficient scan numbers, high baseline noise) are visibly absent from the retained feature list
- Feature overlap with MSDIAL/XCMS outputs is in the expected ~52 ± 5–6% range when comparing pipelines
- The *.pkl model file is present in the mss module directory and loads without error when scoring is enabled

## Limitations

- Scoring is optional, and the authors caution users must develop fundamental understanding of the algorithms' limitations and assumptions to avoid misinterpretation; specific algorithm limitations are not enumerated in the paper.
- Feature grouping (merging isotopes, adducts, in-source fragments) is not implemented, so the RF filter cannot deconvolute co-eluting chemical species.
- The MSS package does not state a version number, dependency versions, or a Zenodo DOI in the article, complicating exact reproducibility of model behavior.
- Dependencies pin scikit-learn >= 0.23.0; the packaged .pkl model may be incompatible with substantially different scikit-learn versions.

## Evidence

- [methods] optional compute the 15 descriptive parameters per extracted peak and score peak quality with the packaged pre-trained random forest model to exclude noisy peaks: [evidence span withheld — anchor status: not_found]
- [methods] post-processing reduces poor quality features: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features"
- [readme] A random forest model is integrated in the function as optional assessing tool for peak quality assessment.: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
- [methods] MSS extracted 99.5% of known peaks from ENTACT mixtures: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [discussion] users must understand algorithm limitations and assumptions: "we do strongly encourage users to develop fundamental understanding about the algorithms, and their limitations and assumptions"
