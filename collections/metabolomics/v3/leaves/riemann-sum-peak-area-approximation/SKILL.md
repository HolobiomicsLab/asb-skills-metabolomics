---
name: riemann-sum-peak-area-approximation
description: Use when a detected mass trace in centroided LC-MS data has been bounded
  around its chromatographic apex and you need a robust feature-level intensity from
  the summed peak signals.
when_to_use_negative:
- When the input is already an assembled feature map (e.g., featureXML) and intensities
  are final — no further peak integration is needed.
- When using profile-mode data where peaks are not centroided; the Riemann sum assumes
  discrete centroided peaks.
- When peak boundaries are not yet established (e.g., before LOWESS smoothing and
  the s/2 pruning rule), since the sum requires a bounded peak region.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: OpenMS
  role: provides the FeatureFinderMetabo implementation that performs trace detection,
    LOWESS smoothing, apex localization, and Riemann-sum intensity approximation
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: the OpenMS/TOPP tool whose internal stage approximates each trace's feature
    intensity by a Riemann sum over the bounded peak region
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:56fe115e75bf18b09b24fbdcc2ae1b83ce2d246bebbca21429333b692a109e68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# riemann-sum-peak-area-approximation

## Summary

Approximate a mass trace's chromatographic peak intensity by a Riemann sum over its bounded peak region, as done in OpenMS FeatureFinderMetabo for metabolite quantification in LC-MS data.

## When to use

Use when a detected mass trace in centroided LC-MS data has been bounded around its chromatographic apex and you need a robust feature-level intensity from the summed peak signals.

## When NOT to use

- When the input is already an assembled feature map (e.g., featureXML) and intensities are final — no further peak integration is needed.
- When using profile-mode data where peaks are not centroided; the Riemann sum assumes discrete centroided peaks.
- When peak boundaries are not yet established (e.g., before LOWESS smoothing and the s/2 pruning rule), since the sum requires a bounded peak region.

## Inputs

- centroided LC-MS peak map (e.g., mzML-derived PeakMap) accessible to FeatureFinderMetabo
- a mass trace (list of peaks along retention time) with chromatographic apex and boundaries determined by LOWESS smoothing and the s/2 pruning rule

## Outputs

- feature-level intensity estimate (Riemann sum over the bounded peak region)
- mass trace intensity that feeds feature assembly and downstream quantification (e.g., the 2,020 ± 236 features per measurement in the paper's plasma experiments)

## How to apply

After FeatureFinderMetabo has detected a mass trace and pruned peaks beyond the s/2 rule from the chromatographic apex, integrate the signal intensity over the trace's bounded peak region by summing the intensities of the retained centroided peaks (a Riemann sum over the peak area). The apex is located as the chromatographic maximum of the trace's LOWESS-smoothed m/z trajectory, and the boundaries come from either the s/2 pruning rule or a minimum-based splitting rule when a trace spans multiple valleys. This approximation yields the per-feature intensity that downstream steps (e.g., the SVM-based isotope-pattern assembly step and quantification) operate on. Rationale: summing discrete peak intensities avoids fitting a parametric peak shape and is robust for centroided data where peaks are sparse point samples along RT.

## Related tools

- **OpenMS** (provides the FeatureFinderMetabo implementation that performs trace detection, LOWESS smoothing, apex localization, and Riemann-sum intensity approximation) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (the OpenMS/TOPP tool whose internal stage approximates each trace's feature intensity by a Riemann sum over the bounded peak region) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in input.mzML -out features.featureXML
```

## Evaluation signals

- Output feature map parses as valid featureXML and contains a nonzero number of mass traces with positive intensities.
- Feature intensities should scale with compound concentration — in the paper's spike-in experiments Pearson correlation R ≥ 0.98 was obtained.
- On the human plasma benchmark, expect on average 2,020 ± 236 features per measurement; a large deviation may indicate integration or boundary errors.
- Sums must not include peaks pruned by the s/2 rule beyond the chromatographic apex.
- Reproducibility check: about 1,168 features should be reproducible in at least 50% of repeated measurements (53% of the average feature count).

## Limitations

- FeatureFinderMetabo's sensitivity/specificity claim (above 95%) was reported for 'typical data' without explicit thresholds or definitions, so boundary choices may need re-tuning on non-typical data.
- The method applies to centroided data and does not describe handling of profile-mode spectra.
- Peaks not assigned to a trace (e.g., very low-intensity signals like the first six concentrations of propionyl-l-carnitine-d3 in the paper) yield no feature and thus no intensity.
- The parameter set (e.g., smoothing window s derived from Δt0.5, FWHM) must be adapted to the chromatographic system; defaults are not transferable across instruments.

## Evidence

- [results] Approximate each trace's feature intensity by a Riemann sum over the bounded peak region and emit all traces as the output mass-trace list M.: [evidence span withheld — anchor status: not_found]
- [methods] LOESS smoothing window and s/2 rule govern the bounds.: [evidence span withheld — anchor status: not_found]
- [abstract] SVM classifier of isotope patterns uses these intensities downstream.: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [readme] Validation signals include featureXML schema compliance and positive intensities.: [evidence span withheld — anchor status: not_found]
