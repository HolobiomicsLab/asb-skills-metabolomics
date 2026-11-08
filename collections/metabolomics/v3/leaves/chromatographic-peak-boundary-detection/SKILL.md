---
name: chromatographic-peak-boundary-detection
description: Use when reconstructing chromatographic peak boundaries from centroided
  LC-MS data — e.g. when a growing mass trace spans several valleys and must be split
  so each trace corresponds to one chromatographic peak, or when apex-based boundary
  definition is needed.
when_to_use_negative:
- Input is profile-mode (non-centroided) LC-MS data — the seeding and recruitment
  step operates on centroided peaks from P′.
- Input is already a feature table or assembled features — this skill only defines
  chromatographic boundaries of mass traces, before feature assembly by m/z spacing,
  co-elution, and SVM isotope-pattern scoring.
- Data are single-scan or direct-infusion MS with no chromatographic (retention-time)
  structure — no RT axis exists on which to detect an apex or split valleys.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FeatureFinderMetabo
  role: implements mass trace detection, LOWESS smoothing, apex-based boundary definition
    (s/2 rule), and valley splitting inside OpenMS
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: open-source framework providing the PeakMap data structure and the TOPP tool
    integration for mass trace boundary detection
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPView
  role: graphical interface for visually inspecting detected mass-trace boundaries
    and split peaks
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:7b79374349e1ec15d68f15bc8b0b28775be40f038c56010abb50c3c5e1a19203
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  - TOPPView
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-peak-boundary-detection

## Summary

Defines the retention-time boundaries of chromatographic peaks within mass traces during FeatureFinderMetabo's mass trace detection: after LOWESS smoothing, the trace apex is located, peaks beyond half a smoothing window (s/2 rule) are pruned, and traces spanning multiple chromatographic valleys are split by a minimum-based rule.

## When to use

Use when reconstructing chromatographic peak boundaries from centroided LC-MS data — e.g. when a growing mass trace spans several valleys and must be split so each trace corresponds to one chromatographic peak, or when apex-based boundary definition is needed.

## When NOT to use

- Input is profile-mode (non-centroided) LC-MS data — the seeding and recruitment step operates on centroided peaks from P′.
- Input is already a feature table or assembled features — this skill only defines chromatographic boundaries of mass traces, before feature assembly by m/z spacing, co-elution, and SVM isotope-pattern scoring.
- Data are single-scan or direct-infusion MS with no chromatographic (retention-time) structure — no RT axis exists on which to detect an apex or split valleys.

## Inputs

- centroided LC-MS data as mzML, loaded into an OpenMS PeakMap accessible to FeatureFinderMetabo
- sorted peak intensities used to seed candidate mass traces from the highest-intensity unassigned peaks
- FeatureFinderMetabo INI parameters (e.g. smoothing window / FWHM-derived s) editable via INIFileEditor

## Outputs

- list of bounded mass traces M, one per chromatographic peak, each with an Riemann-sum feature intensity
- featureXML feature map containing a nonzero number of mass traces with positive intensities

## How to apply

Seed a mass trace on an intense, unassigned peak and extend it along RT by recruiting nearby peaks from the centroided peak map (P′) using an online intensity-weighted Gaussian kernel estimate of local m/z density. Smooth each growing trace's m/z trajectory with a degree-2 LOWESS filter whose window is derived from the chromatographic FWHM (s = Δt_0.5) so the trace stays centered on the co-eluting signal. Locate the trace's chromatographic maximum and prune peaks farther than half a smoothing window (s/2 rule) from the apex to define trace boundaries. Split traces that span multiple chromatographic valleys with a minimum-based splitting rule so each trace corresponds to exactly one chromatographic peak. Approximate the feature intensity by a Riemann sum over the bounded peak region and emit traces as the mass-trace list M, validated as a nonempty featureXML with positive intensities.

## Related tools

- **FeatureFinderMetabo** (implements mass trace detection, LOWESS smoothing, apex-based boundary definition (s/2 rule), and valley splitting inside OpenMS) — https://github.com/OpenMS/OpenMS
- **OpenMS** (open-source framework providing the PeakMap data structure and the TOPP tool integration for mass trace boundary detection) — https://github.com/OpenMS/OpenMS
- **TOPPView** (graphical interface for visually inspecting detected mass-trace boundaries and split peaks) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in input.mzML -out features.featureXML
```

## Evaluation signals

- Output feature map exists and parses as valid featureXML.
- Feature map contains a nonzero number of mass traces with positive intensities.
- Every emitted trace corresponds to exactly one chromatographic peak — traces spanning multiple valleys have been split by the minimum-based rule.
- No retained peak lies farther than half a smoothing window (s/2) from the trace's chromatographic apex.
- Qualitative acceptance: the trace set is sensitive enough to serve downstream feature assembly (the paper's criterion for suitable mass trace detection).

## Limitations

- The paper reports no explicit numeric threshold or parameter values for sensitivity/specificity (claimed above 95% on 'typical data'), leaving the smoothing-window and valley-splitting parameters to the tool defaults or INIFileEditor tuning.
- Very low-abundance signals can yield no traces at all: no signals were detected for the first six concentrations of the propionyl-l-carnitine-d3 standard, either by the algorithm or by visual inspection.
- The s/2 boundary rule and LOWESS window depend on the chromatographic FWHM (Δt_0.5), so peak widths differing substantially from the assumed FWHM may bias boundaries.

## Evidence

- [methods] Smooth each growing trace's m/z trajectory with a degree-2 LOWESS filter using a window derived from the chromatographic FWHM (s = Delta_t0.5): [evidence span withheld — anchor status: not_found]
- [methods] Detect the chromatographic maximum of each trace and prune peaks beyond half a smoothing window (s/2 rule) from the apex to define trace boundaries.: [evidence span withheld — anchor status: not_found]
- [methods] Split traces that span multiple chromatographic valleys using a minimum-based splitting rule so each trace corresponds to one chromatographic peak.: [evidence span withheld — anchor status: not_found]
- [readme] It comes with a vast variety of pre-built and ready-to-use tools for proteomics and metabolomics data analysis (TOPPTools): [evidence span withheld — anchor status: external_doc]
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
