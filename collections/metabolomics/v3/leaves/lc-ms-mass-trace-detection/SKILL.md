---
name: lc-ms-mass-trace-detection
description: Use when the input is centroided LC-MS data (mzML) and the goal is sensitive
  detection of mass traces (co-eluting signals of the same ion) as the precursor step
  to feature assembly and label-free metabolite quantification.
when_to_use_negative:
- Input is already a feature table or featureXML — this skill operates on raw centroided
  spectra prior to feature assembly.
- Input data is profile (non-centroided) mode; mass trace seeding and m/z density
  estimation assume centroided peaks.
- The goal is linking features across multiple experiments — use FeatureLinkerUnlabeledQT
  downstream instead.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: FeatureFinderMetabo
  role: 'implements the mass trace detection stage: seeding, online m/z density estimation,
    LOWESS smoothing, peak splitting, and intensity integration'
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS 1.10
  role: framework in which FeatureFinderMetabo is implemented and integrated
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPView
  role: GUI for running/inspecting the mass trace detection and visualizing detected
    traces
  repo: https://github.com/OpenMS/OpenMS
- name: INIFileEditor
  role: tool for modifying FeatureFinderMetabo detection parameters
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:72a6e2422abfa1f20f805a06ce740fb10702f92e7986e2d1f389bc5d4ebd0c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - INIFileEditor
  - OpenMS
  - OpenMS 1.10
  - TOPPView
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# lc-ms-mass-trace-detection

## Summary

Detects chromatographic mass traces from centroided LC-MS data using OpenMS FeatureFinderMetabo: seeding on intense peaks, intensity-weighted Gaussian m/z density estimation, LOWESS smoothing, and valley-based peak splitting — the first stage of metabolite feature-intensity determination.

## When to use

Use when the input is centroided LC-MS data (mzML) and the goal is sensitive detection of mass traces (co-eluting signals of the same ion) as the precursor step to feature assembly and label-free metabolite quantification.

## When NOT to use

- Input is already a feature table or featureXML — this skill operates on raw centroided spectra prior to feature assembly.
- Input data is profile (non-centroided) mode; mass trace seeding and m/z density estimation assume centroided peaks.
- The goal is linking features across multiple experiments — use FeatureLinkerUnlabeledQT downstream instead.

## Inputs

- centroided LC-MS data in mzML format (PeakMap structure)
- FeatureFinderMetabo INI parameter file (editable via INIFileEditor)

## Outputs

- list of detected mass traces M with chromatographic apex, boundaries, and intensity (Riemann-sum feature intensity)
- feature map in featureXML format containing nonzero, positively-intensitied features

## How to apply

Load the centroided mzML input into OpenMS as a PeakMap and run FeatureFinderMetabo (OpenMS 1.10). The algorithm sorts peaks by intensity, seeds candidate mass traces from the highest-intensity not-yet-assigned peaks, and recursively recruits nearby peaks while estimating local m/z density online with an intensity-weighted Gaussian kernel. Each growing trace's m/z trajectory is smoothed with a degree-2 LOWESS filter using a window derived from the chromatographic FWHM (s = Δt_0.5), then peaks beyond half a smoothing window (s/2 rule) from the chromatographic apex are pruned to define boundaries. Traces spanning multiple chromatographic valleys are split with a minimum-based splitting rule so each trace corresponds to one chromatographic peak, and feature intensity is approximated by a Riemann sum over the bounded peak region. Parameters can be modified with the INIFileEditor; the tool can be run standalone or out of TOPPView.

## Related tools

- **FeatureFinderMetabo** (implements the mass trace detection stage: seeding, online m/z density estimation, LOWESS smoothing, peak splitting, and intensity integration) — https://github.com/OpenMS/OpenMS
- **OpenMS 1.10** (framework in which FeatureFinderMetabo is implemented and integrated) — https://github.com/OpenMS/OpenMS
- **TOPPView** (GUI for running/inspecting the mass trace detection and visualizing detected traces) — https://github.com/OpenMS/OpenMS
- **INIFileEditor** (tool for modifying FeatureFinderMetabo detection parameters)

## Examples

```
FeatureFinderMetabo -in input.mzML -out features.featureXML
```

## Evaluation signals

- Output feature map exists, parses as valid featureXML, and contains a nonzero number of mass traces with positive intensities.
- Detected traces correspond to single chromatographic peaks — traces spanning multiple valleys must have been split (no merged-peak traces remain).
- On simulated data matched to experimental conditions, downstream feature assembly achieves precision ~97% and recall ~96% (above the 95% threshold).
- In the human plasma benchmark, feature detection yields on average 2,020 ± 236 features per measurement, of which about 351 singly, 12 doubly, and 6 triply charged.
- Visual inspection of traces in TOPPView shows m/z trajectories centered on the co-eluting signal after LOWESS smoothing.

## Limitations

- No signals could be detected for the first six concentrations of the propionyl-l-carnitine-d3 standard, either by the algorithm or visual inspection — very low-abundance analytes can be missed.
- The paper does not define 'typical data' or exact cutoffs for the >95% sensitivity/specificity claim, so acceptance on novel data types must be re-validated.
- Accuracy depends on centroided input and reasonable chromatographic FWHM estimates for the smoothing window; the algorithm is designed for small molecules (metabolites, lipids, peptides) and may need re-tuning for other analyte classes.

## Evidence

- [methods] Mass trace detection seeds a trace on an intense peak and extends it along the retention time axis: [evidence span withheld — anchor status: not_found]
- [methods] estimating the local m/z density online with an intensity-weighted Gaussian kernel: [evidence span withheld — anchor status: not_found]
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [abstract] The algorithm is based on a sensitive detection of mass traces: [evidence span withheld — anchor status: not_found]
- [readme] It offers an infrastructure for rapid development of mass spectrometry-related software.: [evidence span withheld — anchor status: external_doc]
