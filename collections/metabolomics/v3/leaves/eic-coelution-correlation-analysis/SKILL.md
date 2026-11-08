---
name: eic-coelution-correlation-analysis
description: Use when inspecting a PFAS or other HRMS feature (m/z of interest) in
  raw ddMS2 mzML data and you need to identify coeluting ions — in-source fragments
  or adducts — by correlating EICs of all detected features in the same RT region.
when_to_use_negative:
- Only a feature table (e.g., from vendor software) is available without the corresponding
  mzML files — the README notes data evaluation only works when the mzML files are
  also given, otherwise MS2 data would be missing
- Raw data are not centroided ddMS2 mzML — PFΔScreen expects data-dependent acquisition
  with centroided spectra, ideally one collision energy per precursor
- The goal is de novo feature detection or PFAS prioritization (KMD, MD/C-m/C, suspect
  screening) rather than visualization/coelution checking of already detected features
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Hosts the RawDataVisualization tool whose EIC correlator performs the coelution
    correlation and flags correlated ion pairs
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Used to load mzML raw data and detected features (Python interface to OpenMS)
    so EICs can be extracted from MS1 data
- name: MSConvert
  role: Converts vendor raw MS data (e.g., Agilent .d) into the centroided mzML input
    required for EIC extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:80f2f06b61c451655f13be57860ff49489d78421c1336afa4e29d7e5ded68469
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# eic-coelution-correlation-analysis

## Summary

Reconstruction of PFΔScreen's EIC correlator within the RawDataVisualization tool: extract extracted-ion chromatograms (EICs) from mzML MS1 data at a 5 mDa width and correlate them over an RT window of ±25 s to flag coeluting ions (in-source fragments, adducts) related to a feature of interest.

## When to use

Use when inspecting a PFAS or other HRMS feature (m/z of interest) in raw ddMS2 mzML data and you need to identify coeluting ions — in-source fragments or adducts — by correlating EICs of all detected features in the same RT region.

## When NOT to use

- Only a feature table (e.g., from vendor software) is available without the corresponding mzML files — the README notes data evaluation only works when the mzML files are also given, otherwise MS2 data would be missing
- Raw data are not centroided ddMS2 mzML — PFΔScreen expects data-dependent acquisition with centroided spectra, ideally one collision energy per precursor
- The goal is de novo feature detection or PFAS prioritization (KMD, MD/C-m/C, suspect screening) rather than visualization/coelution checking of already detected features

## Inputs

- Sample mzML raw data file (centroided ddMS2 spectra, e.g., converted from Agilent .d via MSConvert)
- Optional external feature list (external_feature_list.xlsx / SampleFeatures.xlsx) with its corresponding mzML files
- A user-specified m/z of interest and RT range (±25 s) around the feature

## Outputs

- EIC plots for the feature of interest and all coeluting detected features (5 mDa extraction width, RT ±25 s)
- List of flagged coelution pairs (candidate in-source fragments or adducts) with correlation R² > 0.96
- Interactive HTML visualizations saved in a sample-named output folder alongside the PFΔScreen results table (Excel)

## How to apply

Load the sample mzML (centroided, data-dependent acquisition) and, optionally, a custom feature list (e.g., external_feature_list.xlsx) with Python and pyOpenMS; note the corresponding mzML files must be present so MS2 data remain accessible. Implement accurate-mass EIC extraction from MS1 data at an extraction width of 5 mDa within an RT window of ±25 s, supporting extraction of multiple homologue EICs at once by extrapolating a series from a common repeating unit such as CF2. Let the user specify an m/z of interest, then correlate the EICs of all detected features within that RT range and visualize only highly correlating ions (R² > 0.96 per the validation workflow; R² > 0.95 cited as an example). Feature pairs correlated above the R² threshold are flagged as candidate in-source fragments or adducts; the correlator sits in the RawDataVisualization tab, which becomes available once the FeatureFinding tab is completed. Validate end-to-end by confirming EIC plots are produced and coelution pairs are reported only when correlation exceeds R² > 0.96 at 5 mDa extraction width.

## Related tools

- **PFΔScreen** (Hosts the RawDataVisualization tool whose EIC correlator performs the coelution correlation and flags correlated ion pairs) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Used to load mzML raw data and detected features (Python interface to OpenMS) so EICs can be extracted from MS1 data)
- **MSConvert** (Converts vendor raw MS data (e.g., Agilent .d) into the centroided mzML input required for EIC extraction)

## Examples

```
python -c "import pyopenms; exp = pyopenms.MSExperiment(); pyopenms.MzMLFile().load('sample.mzML', exp)"
```

## Evaluation signals

- Coelution pairs are reported only when their correlation exceeds R² > 0.96 at a 5 mDa extraction width (as in the article's validation: 12 out of 368 EICs correlated with R² > 0.96)
- The script runs end-to-end on a sample mzML and both EIC and MS2 plots are produced
- EIC extraction respects the ±25 s RT window and 5 mDa width; homologue series extrapolation (e.g., CF2 repeats) yields multiple EICs n-at-once
- Correlated ions above the R² threshold are visually highlighted in the RawDataVisualization output, consistent with flagging in-source fragments/adducts
- The RawDataVisualization tab is only exercised after FeatureFinding (or ExternalFeatureTable preprocessing) is completed, per the README workflow

## Limitations

- Requires centroided ddMS2 mzML input; MS2 data are inaccessible if the corresponding mzML files are not supplied alongside custom feature lists
- The correlation threshold (R² > 0.96) and extraction width (5 mDa) are adjustable input parameters without stated defaults, and no quantitative benchmark against alternative NTS data-reduction tools is reported
- Correlation above the threshold only nominates candidate in-source fragments or adducts; results require manual verification of peak shape and blank occurrence

## Evidence

- [results] 12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa: "12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa"
- [readme] Additionally, a coelution correlation can be performed with the RawDataVisualization tool.: "Additionally, a coelution correlation can be performed with the RawDataVisualization tool."
- [readme] Data from the results table can be used to visualize EICs (and extrapolate HS with common repeating units such as CF2), MS1 and MS2 spectra.: "Data from the results table can be used to visualize EICs (and extrapolate HS with common repeating units such as CF2), MS1 and MS2 spectra."
- [readme] Whenever the FeatureFinding tab is completed, the RawDataVisualization can be used even without PFAS-specific data.: "Whenever the FeatureFinding tab is completed, the RawDataVisualization can be used even without PFAS-specific data."
