---
name: aems-shot-window-quantification
description: Use when analyzing AEMS MRM raw data (XIC per transition across multiple
  ejections) where per-well peak areas must be computed by aligning the ejection detection-window
  table, subtracting the per-transition baseline, and summing baseline-corrected intensities
  inside each ejection's window.
when_to_use_negative:
- Data is full-scan AEMS acquisitions (precursor ion or neutral loss modes) rather
  than targeted MRM transitions
- Data is conventional LC-MS chromatography where retention-time peak picking, not
  fixed ejection-window integration, is required
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rtmsEcho
  role: Extends the rtms framework to load AEMS MRM raw data, align ejection detection
    windows to mass transitions, estimate the per-transition baseline, and sum baseline-corrected
    XIC intensities to produce per-well quantification areas.
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework used by rtmsEcho for reading and manipulating raw mass-spectrometry
    XIC data during baseline estimation and window summation.
- name: R
  role: Language environment in which rtmsEcho runs the MRM shot-window quantification
    pipeline and exports mrm_areas.csv.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:189fd7ee1c2289236d3fa4bdc228a200c6a67d9bdf5131f5d267121dc95906bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# aems-shot-window-quantification

## Summary

Quantifies the integrated peak area for each mass transition at each ejection (shot) in acoustic ejection mass spectrometry (AEMS) MRM data by baseline-correcting the XIC within the shot's detection window and summing the residual intensity, using the rtmsEcho/rtms R framework.

## When to use

Use when analyzing AEMS MRM raw data (XIC per transition across multiple ejections) where per-well peak areas must be computed by aligning the ejection detection-window table, subtracting the per-transition baseline, and summing baseline-corrected intensities inside each ejection's window.

## When NOT to use

- Data is full-scan AEMS acquisitions (precursor ion or neutral loss modes) rather than targeted MRM transitions
- Data is conventional LC-MS chromatography where retention-time peak picking, not fixed ejection-window integration, is required

## Inputs

- AEMS MRM raw data (XIC per mass transition across multiple ejections)
- Detection-window table mapping each well/ejection to its mass transition and time window
- Transition list (precursor/product m/z pairs) as used by rtmsEcho

## Outputs

- Per-transition, per-ejection peak area table A(m,k) (transition, ejection/well, area)
- mrm_areas.csv quantification output

## How to apply

Load the AEMS MRM raw data and the detection-window table mapping each well/ejection to its mass transition and time window, using the rtmsEcho/rtms R framework. For each transition m and ejection k, extract the discrete XIC points falling inside that ejection's detection window. Estimate the per-transition baseline from the intensity averaged over two one-second windows — one taken before the first ejection and one after the last ejection of the run — scaled by 2/3, then subtract this baseline from the XIC intensities within the ejection window. Sum the baseline-corrected intensity over each ejection's time window to compute the area A(m,k) for every transition-ejection pair. Assemble the A(m,k) areas into a single table (transition, ejection/well, area) and save it as mrm_areas.csv. The rationale is that AEMS eliminates chromatographic separation, so peak integration reduces to baseline-corrected summation inside the fixed ejection window rather than LC peak-picking, matching rtmsEcho's automated peak alignment, baseline correction, and area quantification.

## Related tools

- **rtmsEcho** (Extends the rtms framework to load AEMS MRM raw data, align ejection detection windows to mass transitions, estimate the per-transition baseline, and sum baseline-corrected XIC intensities to produce per-well quantification areas.) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework used by rtmsEcho for reading and manipulating raw mass-spectrometry XIC data during baseline estimation and window summation.)
- **R** (Language environment in which rtmsEcho runs the MRM shot-window quantification pipeline and exports mrm_areas.csv.)

## Examples

```
Rscript rtmsEcho_mrm_quantify.R --input raw_mrm.csv --windows detection_windows.tsv --output mrm_areas.csv
```

## Evaluation signals

- Every transition-ejection pair listed in the detection-window table has a nonmissing A(m,k) entry computed from the baseline-corrected sum
- Areas reproduce the rtmsEcho baseline-correction and area-quantification behavior described in the paper (peak alignment, baseline correction, and area quantification)
- Output mrm_areas.csv contains rows for all (transition, ejection/well) combinations with area values
- Estimated baseline corresponds to the intensity averaged over two one-second windows, one before the first ejection and one after the last ejection, scaled by 2/3
- Peak areas are computed only from XIC points inside each ejection's detection window, not across the full run

## Limitations

- Baseline estimation relies on the assumption that intensity before the first and after the last ejection reflects the per-transition baseline; unusual carryover or gradients could bias it
- Quantification is defined only for MRM runs; rtmsEcho's full-scan (PI/NL) modes use a different analysis path
- Performance depends on the accuracy of the detection-window table mapping each ejection to its transition and time window

## Evidence

- [methods] rtmsEcho estimates the MRM baseline intensity for each mass transition by averaging that transition's intensity over two one-second windows: [evidence span withheld — anchor status: not_found]
- [results] Sum the baseline-corrected intensity over each ejection's time window: [evidence span withheld — anchor status: not_found]
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] direct access to AEMS data with customizable processing of MRM and full-scan acquisitions: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
