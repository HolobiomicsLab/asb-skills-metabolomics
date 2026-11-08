---
name: mrm-transition-xic-extraction
description: Use when processing AEMS MRM raw data from 384- or 1536-well plates where
  each well/ejection maps to one or more mass transitions and you need per-transition
  peak areas, baseline correction, and shot-to-peak association in the rtmsEcho/rtms
  R framework.
when_to_use_negative:
- If the input is a full-scan (precursor ion or neutral loss mode) AEMS acquisition
  rather than an MRM transition list, use the full-scan processing path instead.
- If the data is conventional LC-MS chromatography with continuous chromatographic
  peaks rather than discrete per-well AEMS ejections, this skill is not appropriate.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: 'R package that implements the AEMS MRM processing: XIC extraction, baseline
    estimation/correction, and peak-area quantification per transition and ejection'
  repo: github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying R framework on which rtmsEcho extends to provide raw data loading
    and general MS data structures
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:bfaa63ef7fb324a3daa53352973d3f6c6feae83e798a139b6446575bdfe4df37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mrm-transition-xic-extraction

## Summary

Reconstructs rtmsEcho's MRM quantification path for acoustic ejection mass spectrometry (AEMS) data: extracts per-transition extracted-ion chromatograms (XICs), estimates the per-ejection baseline intensity, applies baseline correction, and integrates baseline-corrected intensity over each ejection's time window to yield per-transition peak areas.

## When to use

Use when processing AEMS MRM raw data from 384- or 1536-well plates where each well/ejection maps to one or more mass transitions and you need per-transition peak areas, baseline correction, and shot-to-peak association in the rtmsEcho/rtms R framework.

## When NOT to use

- If the input is a full-scan (precursor ion or neutral loss mode) AEMS acquisition rather than an MRM transition list, use the full-scan processing path instead.
- If the data is conventional LC-MS chromatography with continuous chromatographic peaks rather than discrete per-well AEMS ejections, this skill is not appropriate.

## Inputs

- AEMS MRM raw data file (acoustic ejection mass spectrometry MRM acquisition)
- Detection-window table mapping each well/ejection to its mass transition and time window

## Outputs

- Per-transition peak-area table A(m,k) for every transition–ejection pair (transition, ejection/well, area), e.g. mrm_areas.csv

## How to apply

Load the AEMS MRM raw data together with the detection-window table mapping each well/ejection to its mass transition and time window. For each transition m and ejection k, extract the discrete XIC points falling inside that ejection's detection window. Estimate the per-transition baseline for each ejection by averaging the transition's intensity over two one-second windows — one taken before the first ejection and one after the last ejection of the run — scaled by 2/3, then subtract this baseline from the XIC intensities within the ejection window. Sum the baseline-corrected intensity over each ejection's time window to compute the area A(m,k) for every transition–ejection pair. Finally, assemble the A(m,k) areas into a single table (transition, ejection/well, area) and save it as mrm_areas.csv; the rationale is that AEMS eliminates chromatographic separation, so each ejection must be treated as an independent shot with its own baseline estimate rather than a continuous chromatographic peak.

## Related tools

- **rtmsEcho** (R package that implements the AEMS MRM processing: XIC extraction, baseline estimation/correction, and peak-area quantification per transition and ejection) — github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying R framework on which rtmsEcho extends to provide raw data loading and general MS data structures)

## Examples

```
Rscript -e 'library(rtmsEcho); areas <- processMRM(raw="aems_raw", windows=windowsTable); write.csv(areas, "mrm_areas.csv", row.names=FALSE)'
```

## Evaluation signals

- Verify that every transition–ejection pair listed in the detection-window table has a nonmissing A(m,k) entry in mrm_areas.csv.
- Confirm the per-transition baseline estimate was computed from one-second windows immediately before the first and after the last ejection, scaled by 2/3.
- Check that areas reproduce the baseline-corrected sum of XIC intensity within each ejection's time window.
- Confirm the output table columns are (transition, ejection/well, area) and the file is named mrm_areas.csv.
- Cross-check that rtmsEcho's reported processing time for a full 384-well MRM plate with <5 transitions is under 2 s on the same input.

## Limitations

- Existing open-source MRM data processing platforms do not address the specialized requirements of AEMS platforms, particularly their unique data architectures and ultrahigh-throughput processing demands.
- No benchmark datasets or accessioned data resources are named in the article to support the reported 10- to 100-fold speedup over conventional methods.
- No software version number is stated for rtmsEcho.

## Evidence

- [methods] rtmsEcho estimates the MRM baseline intensity for each mass transition by averaging that transition's intensity over two one-second windows, one taken before the first ejection and one after the last ejection of the run.: [evidence span withheld — anchor status: not_found]
- [intro] AEMS eliminates chromatographic separation, requires only nanoliter-scale sample volumes, and supports hundreds to thousands of measurements per hour from 384- or 1536-well plates.: [evidence span withheld — anchor status: not_found]
- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
- [abstract] github.com__nathanieltwarog__rtmsEcho: "# github.com__nathanieltwarog__rtmsEcho"
