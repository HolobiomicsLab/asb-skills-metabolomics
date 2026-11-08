---
name: lcms-feature-rescue-from-dda-in-dia
description: Use when DIA (SWATH) feature tables of individual biological samples
  lack metabolic features that were confidently detected in the pooled-QC DDA run,
  and their intensities must be recovered from the DIA raw data.
when_to_use_negative:
- The feature is already detected with an intensity in the DIA feature table of the
  individual sample — no rescue is needed.
- Input files are not paired DDA/DIA mzXML acquisitions (e.g. only DIA or only DDA
  data exist), since EIC matching requires the DIA raw chromatogram at the DDA feature's
  m/z and retention time.
- The goal is MS2 spectral annotation rather than intensity recovery — use the DaDIA.R
  annotation step with the MoNA spectral library instead.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DaDIA.R
  role: R program that flags DDA-detected features missing in DIA sample tables and
    re-determines their intensities via EIC matching in DIA raw data
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Provides the centWave function used by DaDIA.R for peak picking to extract
    metabolic features from DDA and DIA mzXML files prior to rescue
  repo: https://github.com/Huan Lab/DaDIA.git
- name: metaMS
  role: Required R dependency (Version 1.25.1) of the DaDIA.R rescue pipeline
  repo: https://github.com/Huan Lab/DaDIA.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:46f507fbb3f78d3408c8872e8d496706afdd6f40dd80fc91f7d5958c96c82158
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lcms-feature-rescue-from-dda-in-dia

## Summary

Rescue metabolic features that were detected in the pooled-QC DDA run but are missing from the individual-sample DIA feature tables by matching extracted-ion chromatograms (EICs) in the DIA raw data, as implemented in the DaDIA.R workflow.

## When to use

Use when DIA (SWATH) feature tables of individual biological samples lack metabolic features that were confidently detected in the pooled-QC DDA run, and their intensities must be recovered from the DIA raw data.

## When NOT to use

- The feature is already detected with an intensity in the DIA feature table of the individual sample — no rescue is needed.
- Input files are not paired DDA/DIA mzXML acquisitions (e.g. only DIA or only DDA data exist), since EIC matching requires the DIA raw chromatogram at the DDA feature's m/z and retention time.
- The goal is MS2 spectral annotation rather than intensity recovery — use the DaDIA.R annotation step with the MoNA spectral library instead.

## Inputs

- DDA mzXML files of pooled quality-control (QC) samples
- DIA (SWATH) mzXML files of individual biological samples
- XCMS centWave-detected feature tables from both DDA and DIA runs (feature m/z, retention time, intensity)

## Outputs

- Rescued-feature intensity table with feature m/z, retention time, and per-sample intensity (exported to CSV)
- NA entries for DDA-unique features that could not be rescued by EIC matching

## How to apply

Run DaDIA.R (R ≥ 4.0, XCMS ≥ 3.11.4, metaMS 1.25.1) with its centWave-based peak picking over both the DDA pooled-QC mzXML files and the DIA individual-sample mzXML files to extract features and MS2 spectra. Identify DDA-detected metabolic features whose intensities are missing (NA) in the DIA feature tables of individual samples — in the urine RP(+) demo these averaged 344 rescuable features, about 22% of the 1591 DDA features. For each such DDA-unique feature, re-determine its intensity in the DIA raw data by extracted-ion-chromatogram (EIC) matching at the feature's m/z and retention time; assign the matched intensity to the metabolite-intensity table. If EIC matching cannot rescue the feature, assign an NA value instead of a fabricated intensity. Export the rescued-feature table (m/z, retention time, rescued intensity per sample) and sanity-check by manually inspecting EICs — the paper reports ~71% true-positive and ~17% false-positive rescued EICs in urine RP(+) data.

## Related tools

- **DaDIA.R** (R program that flags DDA-detected features missing in DIA sample tables and re-determines their intensities via EIC matching in DIA raw data) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Provides the centWave function used by DaDIA.R for peak picking to extract metabolic features from DDA and DIA mzXML files prior to rescue) — https://github.com/Huan Lab/DaDIA.git
- **metaMS** (Required R dependency (Version 1.25.1) of the DaDIA.R rescue pipeline) — https://github.com/Huan Lab/DaDIA.git

## Examples

```
Rscript DaDIA.R
```

## Evaluation signals

- Rescued-feature output table exists and contains the m/z, retention time, and intensity fields for every DDA-unique feature
- Average number of rescued features from DDA is on the order of hundreds (paper reports 344, ≈22% of 1591 DDA features, in urine RP(+) data)
- Features that could not be rescued are assigned NA, not an arbitrary or zero intensity
- Manual inspection of a sample of rescued EICs should show mostly good chromatographic peak shapes (paper reports ~71% true-positive rate)
- DaDIA.R runs end-to-end without errors on the demo DDA/DIA mzXML files; the number of samples declared in line 38 must equal the real number of DIA mzXML files

## Limitations

- Rescued EICs are not all true positives: manual inspection showed a 17% false-positive rate and 12% undetermined peak shapes in urine RP(+) data, so rescued intensities should be spot-checked.
- DaDIA.R requires R 4.0+, XCMS ≥ 3.11.4, and metaMS 1.25.1; older or newer versions may break the rescue logic.
- The number of samples hardcoded in line 38 of DaDIA.R must match the real number of DIA mzXML files or the rescue will fail or produce wrong per-sample assignments.

## Evidence

- [results] In DaDIA.R's feature-rescue mechanism, a DDA-unique feature missing from a DIA run has its intensity determined from the DIA raw data and assigned to the metabolite-intensity table; if it cannot be rescued, an NA value is assigned.: [evidence span withheld — anchor status: not_found]
- [results] Manual EIC inspection of rescued features: 71% true-positive rate, 17% false-positive rate, 12% undetermined: [evidence span withheld — anchor status: not_found]
