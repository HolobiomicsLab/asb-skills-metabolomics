---
name: mzml-spectra-parsing
description: 'Use when raw HRMS non-targeted analysis data in .mzML format must be
  converted into Python-readable scan metadata prior to feature extraction (e.g.,
  mssmain.peak_pick) or when validating MSS on samples like EPA ENTACT mixtures #505,
  #506, #508.'
when_to_use_negative:
- Input is already a processed feature table (e.g., a .csv/.xlsx feature list from
  MSDIAL, XCMS, or a prior MSS run) — parsing applies only to raw .mzML files
- Data are in vendor-native raw formats (e.g., Thermo/Fishaw .raw) that have not been
  converted to .mzML — the tool supports only .mzML
- The goal is only MS/MS fragment annotation against MoNA MassBank without importing
  new raw scans (use the frag module path instead)
edam_operation: http://edamontology.org/operation_3222
tools:
- name: Mass-Suite (MSS)
  role: Provides the mssmain.get_scans function that imports and parses .mzML files
    into Python-compatible metadata as the first pipeline step
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Underlying mzML parser (cElementTree-based) called by mssmain.get_scans; requires
    Python 3.7+ and version 2.4.6 or higher per the MSS README
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:9733c4e115a27e2b9644cef69ca649353541c2f40cf037d7e166cf60430a04c4
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mzml-spectra-parsing

## Summary

Import and parse high-resolution mass spectrometry (HRMS) data in .mzML format into Python-compatible scan metadata using the Mass-Suite (MSS) function mssmain.get_scans, which wraps the pymzML parser. This is the mandatory first step of the MSS non-targeted analysis (NTA) pipeline before peak picking and feature extraction.

## When to use

Use when raw HRMS non-targeted analysis data in .mzML format must be converted into Python-readable scan metadata prior to feature extraction (e.g., mssmain.peak_pick) or when validating MSS on samples like EPA ENTACT mixtures #505, #506, #508.

## When NOT to use

- Input is already a processed feature table (e.g., a .csv/.xlsx feature list from MSDIAL, XCMS, or a prior MSS run) — parsing applies only to raw .mzML files
- Data are in vendor-native raw formats (e.g., Thermo/Fishaw .raw) that have not been converted to .mzML — the tool supports only .mzML
- The goal is only MS/MS fragment annotation against MoNA MassBank without importing new raw scans (use the frag module path instead)

## Inputs

- Raw HRMS data files in .mzML format (e.g., archived EPA ENTACT study samples #505, #506, #508)
- Environment definition file environment.yml for the MSS conda environment

## Outputs

- Python-compatible scan metadata (readable dataframe) per mzML file, held in memory for downstream mssmain.peak_pick processing
- Optionally, baseline-subtracted scan data ready for feature extraction

## How to apply

Call the Mass-Suite function mssmain.get_scans on each .mzML file; under the hood it uses pymzML (version 2.4.6 or higher per the README) to parse raw scans and convert them into Python-compatible metadata (a dataframe in readable format). Set up the MSS environment first, e.g. `conda env create -f environment.yml; conda activate mss`, which pins pandas 1.0.3+, numpy 1.16.5+, scipy 1.4.1+, and pymzML 2.4.6+. After parsing, optionally apply baseline subtraction based on signal intensities — the parsed scans are then ready for feature extraction via mssmain.peak_pick, which uses PeakUtils to find peak indices and applies post-processing (peak width filter, replicated peak filter, regression-based peak boundary determination). The article validated this parsing-to-extraction chain by matching extracted peaks against a list of 398 MS-amenable chemicals known to be present in all three ENTACT mixtures, achieving 99.5% feature-extraction accuracy (2 peaks unmatched).

## Related tools

- **Mass-Suite (MSS)** (Provides the mssmain.get_scans function that imports and parses .mzML files into Python-compatible metadata as the first pipeline step) — https://github.com/XiminHu/mass-suite
- **pymzML** (Underlying mzML parser (cElementTree-based) called by mssmain.get_scans; requires Python 3.7+ and version 2.4.6 or higher per the MSS README) — https://github.com/pymzml/pymzML

## Examples

```
import mss; from mss import mssmain; scans = mssmain.get_scans('ENTACT_505.mzML')
```

## Evaluation signals

- Parsed output is Python-compatible scan metadata (a dataframe) for every input .mzML file, not raw vendor bytes
- Downstream feature extraction on ENTACT mixtures #505/506/508 should match 99.5% of the 398 known spiked chemicals (only ~2 peaks unmatched)
- Environment satisfies pymzML >= 2.4.6 and Python >= 3.7 as required by the MSS README and pymzML README
- Optional baseline subtraction on signal intensities, if applied, leaves scan structure intact so mssmain.peak_pick can concatenate scans and find peak indices with PeakUtils
- pymzML itself passes its own test suite (`tox`) confirming correct installation of the parsing layer

## Limitations

- Only .mzML is supported by the MSS import module — other formats must be converted beforehand
- No MSS package version number, changelog, or archived release (e.g., Zenodo DOI) is cited, only the GitHub URL and pypi.org listing, so the exact parser behavior may drift between releases
- The paper specifies no Python or dependency versions in its own requirements ('Other requirements: none'); version constraints come only from the repository README
- mssmain.get_scans itself does not fix bad data quality: downstream filtering (peak width, replicated peak, baseline noise) is a separate peak_pick post-processing step

## Evidence

- [methods] raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function.: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [methods] Converted data are then available for optional baseline subtraction based on signal intensities prior to subsequent feature extraction: "Converted data are then available for optional baseline subtraction based on signal intensities prior to subsequent feature extraction"
- [readme] b) a very fast parser for mzML data, the standard mass spectrometry data format: [evidence span withheld — anchor status: external_doc]
- [readme] Supported formats: * .mzml — MS data will be imported and filtered to create a dataframe in readable format from the mzml file.: "MS data will be imported and filtered to create a dataframe in readable format from the mzml file."
- [readme] pymzML requires Python3.7+.: [evidence span withheld — anchor status: external_doc]
