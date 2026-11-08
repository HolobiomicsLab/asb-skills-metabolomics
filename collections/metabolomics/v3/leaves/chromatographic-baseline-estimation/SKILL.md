---
name: chromatographic-baseline-estimation
description: Use when processing acoustic ejection MS MRM data where per-ejection
  peak areas must be baseline-corrected before quantification, e.g. to compute A(m,k)
  transition areas from rtmsEcho-extracted XICs.
when_to_use_negative:
- Input is conventional LC-MS chromatographic data with real chromatographic separation
  rather than discrete ejection windows
- Only full-scan (precursor-ion or neutral-loss) AEMS acquisitions are available and
  no MRM transition XICs are present
- Baseline has already been corrected upstream and only raw integrated areas are needed
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: R package that automates baseline correction and area quantification for AEMS
    MRM data
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying rtms framework extended by rtmsEcho for loading and processing
    mass spectrometry data
- name: R
  role: Language in which the rtms/rtmsEcho pipeline is written and run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:e13baac0aaf4cbbe2ab935f69d229140a177b38f8e5c5b8870ce2a906414f535
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chromatographic-baseline-estimation

## Summary

Estimate and subtract the per-transition baseline intensity for acoustic ejection mass spectrometry (AEMS) MRM traces so that peak-area quantification reflects true signal rather than instrument noise floor, as implemented in rtmsEcho.

## When to use

Use when processing acoustic ejection MS MRM data where per-ejection peak areas must be baseline-corrected before quantification, e.g. to compute A(m,k) transition areas from rtmsEcho-extracted XICs.

## When NOT to use

- Input is conventional LC-MS chromatographic data with real chromatographic separation rather than discrete ejection windows
- Only full-scan (precursor-ion or neutral-loss) AEMS acquisitions are available and no MRM transition XICs are present
- Baseline has already been corrected upstream and only raw integrated areas are needed

## Inputs

- AEMS MRM raw data (extracted-ion chromatogram points per transition)
- Detection-window table mapping each well/ejection to its mass transition and time window

## Outputs

- Per-transition, per-ejection baseline-corrected peak area A(m,k)
- mrm_areas.csv table (transition, ejection/well, area)

## How to apply

For each MRM mass transition m and each ejection k, extract the discrete extracted-ion-chromatogram (XIC) points falling inside that ejection's detection window from the AEMS raw data, using the detection-window table that maps each well/ejection to its transition and time window. Estimate the transition's baseline intensity by averaging the transition's XIC intensity over two one-second windows: one taken immediately before the first ejection and one immediately after the last ejection of the run, scaled by 2/3. Subtract this estimated baseline from the XIC intensities within the ejection's detection window, then sum the baseline-corrected intensity over that window to yield the area A(m,k). Assemble all A(m,k) values into a transition-by-ejection table (saved as mrm_areas.csv in the reference workflow) so downstream quantification uses background-corrected areas rather than raw integrated signal.

## Related tools

- **rtmsEcho** (R package that automates baseline correction and area quantification for AEMS MRM data) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying rtms framework extended by rtmsEcho for loading and processing mass spectrometry data)
- **R** (Language in which the rtms/rtmsEcho pipeline is written and run)

## Examples

```
Rscript -e 'library(rtmsEcho); areas <- rtmsEcho::estimateMRMAreas(raw_xic, window_table); write.csv(areas, "mrm_areas.csv", row.names=FALSE)'
```

## Evaluation signals

- Every transition–ejection pair listed in the detection-window table has a nonmissing A(m,k) entry
- Areas reproduce the rtmsEcho baseline-correction and area-quantification behavior described in the paper
- Summed baseline-corrected intensities within each ejection window match expected per-transition signal-to-noise improvements
- Output table mrm_areas.csv contains one row per (transition, ejection/well) pair with finite area values

## Limitations

- Baseline is estimated from only two one-second windows (before first and after last ejection) scaled by 2/3, which may be fragile when baseline drifts within a run
- Requires knowledge of the exact detection-window boundaries; incorrect window assignments propagate directly into baseline and area estimates
- Method is specific to AEMS MRM data; behavior for PI/NL full-scan modes is not covered by this skill

## Evidence

- [methods] rtmsEcho estimates the MRM baseline intensity for each mass transition by averaging that transition's intensity over two one-second windows, one taken before the first ejection and one after the last ejection of the run.: [evidence span withheld — anchor status: not_found]
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] we developed rtmsEcho, an open-source R package that extends our previously published rtms framework: "we developed rtmsEcho, an open-source R package that extends our previously published rtms framework"
