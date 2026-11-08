---
name: mzml-scan-import-parsing
description: Use when starting an HRMS non-targeted analysis from vendor-neutral raw
  files (.mzML) in Python and you need scan-level metadata (m/z–intensity scans) before
  chromatographic peak detection with mssmain.peak_pick.
when_to_use_negative:
- The input is already a processed feature table or peak list (e.g., .csv of m/z–RT
  pairs) — go straight to alignment, clustering, or statistical analysis instead of
  re-parsing raw scans.
- 'The raw data is not in .mzML format (README: ''Supported formats: .mzml'') — convert
  it first or use another tool.'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Provides mssmain.get_scans, the import/parsing function producing Python-compatible
    scan metadata, with optional baseline subtraction
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Underlying .mzML parser used by mssmain.get_scans; fast mzML parsing with
    random access in compressed files
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:308e21159b0380cb92b2b03199ed14f7c3f8319b7d033aa112a0c3e0aeb667d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Pandas
  - PeakUtils
  - Python
  - pymzML
  - pymzml
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mzml-scan-import-parsing

## Summary

Import and parse HRMS raw data in .mzML format into Python-compatible scan metadata using pymzML via the Mass-Suite (MSS) function mssmain.get_scans, with optional baseline subtraction prior to peak picking. This is the first stage of the MSS feature-extraction pipeline for non-targeted analysis.

## When to use

Use when starting an HRMS non-targeted analysis from vendor-neutral raw files (.mzML) in Python and you need scan-level metadata (m/z–intensity scans) before chromatographic peak detection with mssmain.peak_pick.

## When NOT to use

- The input is already a processed feature table or peak list (e.g., .csv of m/z–RT pairs) — go straight to alignment, clustering, or statistical analysis instead of re-parsing raw scans.
- The raw data is not in .mzML format (README: 'Supported formats: .mzml') — convert it first or use another tool.

## Inputs

- HRMS raw data in .mzML format (e.g., package example data from the MSS DEMO folder)
- Optional baseline-subtraction setting (based on signal intensities)

## Outputs

- Python-compatible scan metadata (parsed scan objects/metadata suitable for mssmain.peak_pick)
- Downstream feature list of m/z–RT pairs exportable to CSV via pandas

## How to apply

Convert or obtain the raw HRMS data in .mzML format, then call mssmain.get_scans, which parses the file with pymzML (README requires pymzML 2.4.6 or higher) into Python-compatible scan metadata. Optionally apply baseline subtraction based on signal intensities before downstream feature extraction — this is a decision point for noisy baselines. The resulting scan objects feed directly into mssmain.peak_pick, which concatenates scans and finds peak indices using PeakUtils, so keep the parsed output in the structure that function expects. For batch processing of many files, note that MSS multiprocessing reduced single-file extraction time from 201 ± 1.1 s to 58 ± 1.4 s, which is relevant when importing large datasets. MSS also supports data alignment across experiments after import, and results can be exported as .csv/.xlsx.

## Related tools

- **Mass-Suite (MSS)** (Provides mssmain.get_scans, the import/parsing function producing Python-compatible scan metadata, with optional baseline subtraction) — https://github.com/XiminHu/mass-suite
- **pymzML** (Underlying .mzML parser used by mssmain.get_scans; fast mzML parsing with random access in compressed files) — https://github.com/pymzml/pymzML

## Examples

```
from mss import mssmain; scans = mssmain.get_scans('example_data/sample.mzML', import_mode='centroid', noise_threshold=5000)
```

## Evaluation signals

- The .mzML file parses without error and returns scan objects with metadata usable by downstream MSS functions.
- Subsequent mssmain.peak_pick on the parsed scans yields chromatographic peak indices (using PeakUtils), confirming the import output is in the expected structure.
- End-to-end check: the extracted feature_table.csv exists and is a CSV containing m/z–RT feature pairs.
- On EPA ENTACT mixtures #505, #506, #508 (398 MS-amenable chemicals), the full MSS pipeline recovered 99.5% of known peaks (2 of 398 unmatched) with default settings — a benchmark for the import-to-feature workflow.
- Post-processing filters (peak width filter, replicated peak filter, regression-based peak boundary determination) can run on the parsed scans to reduce poor-quality features.

## Limitations

- Only .mzML is supported for raw data import ('Supported formats: .mzml').
- Feature grouping to merge isotopes, adducts, and in-source fragments is not yet implemented — listed as future work.
- No Python version or dependency versions specified in the article's requirements ('Other requirements: none'); the README specifies minimum versions (pymzML 2.4.6, Pandas 1.0.3, etc.), but no MSS package version or archived release (e.g., Zenodo DOI) is cited for reproducibility.
- Authors caution users must develop fundamental understanding about the algorithms, and their limitations and assumptions, to avoid misinterpretation.

## Evidence

- [methods] raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function.: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [methods] Converted data are then available for optional baseline subtraction based on signal intensities prior to subsequent feature extraction: "Converted data are then available for optional baseline subtraction based on signal intensities prior to subsequent feature extraction"
- [methods] The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53]): "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53])"
- [readme] Supported formats:
* .mzml

MS data will be imported and filtered to create a dataframe in readable format from the mzml file.: "MS data will be imported and filtered to create a dataframe in readable format from the mzml file."
- [methods] MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73].: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
