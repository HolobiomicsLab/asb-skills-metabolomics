---
name: intensity-weighted-gaussian-mz-density-estimation
description: Use when reconstructing or tuning the mass trace detection stage of LC-MS
  feature finding from centroided mzML data, specifically when deciding which nearby
  peaks belong to a seeded trace along the m/z axis.
when_to_use_negative:
- Input data is profile-mode or already aggregated into a feature table rather than
  centroided peaks
- The analysis goal is the second stage of the algorithm (feature assembly from already-detected
  mass traces via m/z spacing, co-elution, and SVM isotope-pattern scoring), not trace
  seeding and recruitment
- Using alternative feature-detection packages such as XCMS/CAMERA where a different
  peak-picking model is already assumed
tools:
- name: FeatureFinderMetabo
  role: Hosts the mass trace detection stage in which the intensity-weighted Gaussian
    m/z density is estimated online while traces are seeded and extended
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: Framework providing the PeakMap data structure for centroided mzML input and
    the library in which the kernel-density trace extension is implemented (version
    1.10)
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:99cebe882a9daf31c2f169203f9f0fb567f79b4293675cdb8690fefead49216f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Intensity-weighted Gaussian m/z density estimation

## Summary

An online density-estimation step within the FeatureFinderMetabo mass trace detection stage that characterizes the local m/z neighborhood of a growing trace using an intensity-weighted Gaussian kernel, so peaks most likely originating from the same ion mass are recruited while co-eluting noise is rejected.

## When to use

Use when reconstructing or tuning the mass trace detection stage of LC-MS feature finding from centroided mzML data, specifically when deciding which nearby peaks belong to a seeded trace along the m/z axis.

## When NOT to use

- Input data is profile-mode or already aggregated into a feature table rather than centroided peaks
- The analysis goal is the second stage of the algorithm (feature assembly from already-detected mass traces via m/z spacing, co-elution, and SVM isotope-pattern scoring), not trace seeding and recruitment
- Using alternative feature-detection packages such as XCMS/CAMERA where a different peak-picking model is already assumed

## Inputs

- Centroided LC-MS spectra loaded into an OpenMS PeakMap (mzML input) accessible to FeatureFinderMetabo
- Peak list P′ sorted by intensity for seeding and recursive recruitment

## Outputs

- Per-trace local m/z density estimates guiding peak recruitment into mass traces
- Growing mass traces (m/z trajectories over retention time) passed to LOWESS smoothing and boundary definition

## How to apply

Sort centroided peaks by intensity and seed candidate mass traces from the highest-intensity, not-yet-assigned peaks in the PeakMap. While extending each seed along the retention time axis in both directions, estimate the local m/z density online with an intensity-weighted Gaussian kernel and recursively recruit peaks from the peak pool P′ that are close in m/z and thus most likely originate from the same ion mass. Use the estimated density to keep the trace centered on the co-eluting signal, then smooth the growing trace's m/z trajectory with a degree-2 LOWESS filter whose window is derived from the chromatographic FWHM (s = Delta_t0.5). This kernel-based density estimate is what distinguishes true trace continuation from background peaks in complex metabolomic backgrounds such as human plasma; the downstream steps (s/2 pruning from the apex, minimum-based valley splitting, Riemann-sum intensity) operate on the recruited trace.

## Related tools

- **FeatureFinderMetabo** (Hosts the mass trace detection stage in which the intensity-weighted Gaussian m/z density is estimated online while traces are seeded and extended) — https://github.com/OpenMS/OpenMS
- **OpenMS** (Framework providing the PeakMap data structure for centroided mzML input and the library in which the kernel-density trace extension is implemented (version 1.10)) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in input.mzML -out features.featureXML
```

## Evaluation signals

- Output feature map exists, parses as valid featureXML, and contains a nonzero number of mass traces whose intensities are positive
- Recruited trace peaks remain close in m/z to the seed — the kernel should exclude peaks unlikely to originate from the same ion mass
- Detected features per measurement fall in the paper's reported range (on average 2,020 ± 236 features per measurement on the metabolomics benchmark data)
- Quantification linearity of resulting feature intensities vs. spike-in concentration (Pearson R ≥ 0.98 for all standard compounds)

## Limitations

- The sensitivity/specificity claim (above 95% on typical data) lacks explicit threshold, parameter, and 'typical data' definitions in the article
- Very low-abundance analytes may escape detection: no signals could be detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection
- The density estimate is tuned to centroided LC-MS metabolomics data; applicability to other separation technologies or analyte classes (lipidomics, peptidomics) requires re-validation

## Evidence

- [methods] Mass trace detection seeds a trace on an intense peak and extends it along the retention time axis in both directions by recruiting peaks from P′ that are close in m/z: [evidence span withheld — anchor status: not_found]
- [abstract] The algorithm is based on a sensitive detection of mass traces: "The algorithm is based on a sensitive detection of mass traces"
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [readme] OpenMS is an open-source software C++ library for LC-MS data management and analyses.: [evidence span withheld — anchor status: not_found]
- [results] In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations: "In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations"
